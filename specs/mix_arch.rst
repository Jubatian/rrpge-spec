
RRPGE Audio mixer peripheral architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


The mixer peripheral assists audio generation by providing hardware
accelerated sample mixing capabilities including amplitude and frequency
modulation. It operates within the Data Memory space, on the first 256 banks
(1 MWords) which includes the buffers used by the audio DMA.

Mixer operations stall the CPU until their completion.




Mixer architecture
------------------------------------------------------------------------------


The mixer operates on up to three sources, a frequency table, and a
destination all found in it's accessible Data Memory space (the first 128
banks as seen from the CPU). The result of the operation depends on how these
sources and related algorithms are connected together.

The following block diagram depicts the overall architecture of the mixer: ::


    +----DRAM----+
    |            |
    | +--------+ |
    | | Sample |-------------Data-----------------------------------------+
    | | source | |  +------+   _  +--------+    +--------+    +--------+  |
    | |        |<---| Ptr. |<-|+|-| S. add |<-+-| S. frq |<-+-| FM sel |  |
    | +--------+ |  +------+   ~  +--------+  ^ +--------+  ^ +--------+  |
    |            |                            |             |     O:      |
    | +--------+ |                            |             |     f:      |
    | |   FM   |-------------Data------------)|(------------+     f:      |
    | | source | |  +------+   _  +--------+  | +--------+         :      |
    | |        |<---| Ptr. |<-|+|-| FM add |<-+-| FM frq |.........:      |
    | +--------+ |  +------+   ~  +--------+  ^ +--------+                |
    |            |                            |                           |
    | +--------+ |                            |                           |
    | |   AM   |-------------Data------------)|(-----------+              |
    | | source | |  +------+   _  +--------+  | +--------+ |              |
    | |        |<---| Ptr. |<-|+|-| AM add |<-+-| AM frq | |              |
    | +--------+ |  +------+   ~  +--------+  ^ +--------+ |              |
    |            |                            |      :     |              |
    | +--------+ |                            |     O:     |              |
    | | Freq.  | |                            |     f:     |              |
    | | table  |------------------------------+     f:     |              |
    | |        | |           +-----------+      +--------+ V  +--------+  |
    | +--------+ |           |Add enable |      | AM sel |-+->| AM mul |>|*|
    |            |           +-----------+      +--------+    +--------+  |
    | +--------+ |                 |   _                                  |
    | | Sample |-------------Data->+<-|0|                                 |
    | | dest.  | |                 V   ~                                  |
    | |        |<------------Data-|+|-------------------------------------+
    | +--------+ |                 ~
    |            |
    +------------+


For all reads the order of operations are first performing the access, then
performing the pointer addition (by the appropriate frequency). The order of
operations for processing a destination word are as follows:

- Fetch frequency index if FM mode is enabled.
- Increment FM source pointer with FM add if FM mode is enabled.
- Fetch frequency value from frequency table if FM mode is enabled.
- Fetch first sample.
- Temporarily increment sample pointer with half the frequency value.
- Fetch amplitude if AM mode is enabled.
- Increment amplitude source pointer with AM add if AM mode is enabled.
- Fetch second sample.
- Increment (original) sample pointer with the frequency value.
- Calculate the source data (multiply with amplitude).
- Fetch destination (two samples).
- Add source to destination (saturated, two samples).
- Write back into destination.
- Increment destination.

The AM and FM sources are read once every output word, that is, once for every
two samples. The frequencies ("AM frq" and "FM frq") correspond to this rate.
The sample source however is read twice per output. To keep it's frequency
working in an identical manner to that of the AM and FM sources, it implements
the following behavior:

- After fetching the first sample, a temporary pointer is created by adding
  half of the sample frequency (as in logical shifting right by one) to the
  sample pointer.

- The second sample is fetched using this temporary pointer.

- After fetching the second sample, the original pointer is incremented by the
  sample frequency.

The "AM sel" and "FM sel" settings control whether the AM or the FM sources
are active respectively. When these sources are inactive, the last written
value will persist in the "S. frq" and "AM mul" fields respectively, thus
disabling these features. Moreover when these are disabled, the loading of the
sources and the increment of the pointers are also disabled.

For all three sources and the destination a partition size setting is
provided, one for each. This allows for selecting sample data sizes from 4
samples (2 words) to 128K samples (64K words) in power of 2 increments. The
pointers automatically wrap to the beginning of the partition when passing
it's end.

Before starting a mixer operation either the frequency by the current sample
frequency (S. frq) is fetched from the frequency table for "S. add", or the
frequency by the FM mode (FM frq) for "FM add", depending on whether FM mode
is enabled or not. Likewise the frequency used for "AM add" is also fetched if
necessary.

Amplitude is applied to the source data as follows:

src_a = (((src - 128) * amp) / 256) + 128

Without relying on signed arithmetic this may be expressed as:

src_a = (((src * amp) >> 7) + 256 - amp) >> 1

Note that this algorithm implies that the maximal valid amplitude of 0xFF can
not reproduce the original data. AM modulation has to be turned off to keep
the data as-is.

If adding to the destination is enabled, the result forms as follows:

dest = satu(src_a + dest - 128)

The saturation trims the result to the 8 bit range (0x00 - 0xFF).




Mixer operation timing
------------------------------------------------------------------------------


The timing of the mixer operations depend only on whether FM mode is enabled
or not. Otherwise the timing is consistent for all configurations, dominated
by the necessary memory accesses.

FM disabled (cycles): 18 + (6 * n)

FM enabled (cycles): 20 + (10 * n)

'n' is the number of words to process.




Mixer peripheral memory map
------------------------------------------------------------------------------


The following table lists the memory addresses within the User peripheral page
which relate the mixer. Note that these repeat every 32 words in the 0xE00 -
0xFFF range within this page.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xEC0  |                                                                   |
| \-     | Other peripherals, not used by the Mixer DMA. See "mem_map.rst".  |
| 0xECD  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xECE  | Frequency table whole pointer in 256 word units, only low 12 bits |
|        | are used. The table contains 256 entries.                         |
+--------+-------------------------------------------------------------------+
| 0xECF  | Frequency table fractional pointer in 256 word units, only low 12 |
|        | bits are used. The table contains 256 entries.                    |
+--------+-------------------------------------------------------------------+
| 0xED0  | Frequency source start partition select bits. Used in FM mode for |
|        | reading the frequency source.                                     |
+--------+-------------------------------------------------------------------+
|        | Frequency source start pointer whole part (addresses 16 bit word  |
| 0xED1  | units). Used in FM mode for reading the frequency source. Updated |
|        | during a mixer operation in FM mode, otherwise ignored.           |
+--------+-------------------------------------------------------------------+
|        | Frequency source start pointer fractional part. Used in FM mode   |
| 0xED2  | for reading the frequency source. Updated during a mixer          |
|        | operation in FM mode, otherwise ignored.                          |
+--------+-------------------------------------------------------------------+
| 0xED3  | Amplitude source start partition select bits. Used in AM mode for |
|        | reading the frequency source.                                     |
+--------+-------------------------------------------------------------------+
|        | Amplitude source start pointer whole part (addresses 16 bit word  |
| 0xED4  | units). Used in AM mode for reading the frequency source. Updated |
|        | during a mixer operation in AM mode, otherwise ignored.           |
+--------+-------------------------------------------------------------------+
|        | Amplitude source start pointer fractional part. Used in AM mode   |
| 0xED5  | for reading the frequency source. Updated during a mixer          |
|        | operation in AM mode, otherwise ignored.                          |
+--------+-------------------------------------------------------------------+
|        | Frequency index for AM / FM source reads.                         |
| 0xED6  |                                                                   |
|        | - bit  8-15: Frequency index of Frequency source reads.           |
|        | - bit  0- 7: Frequency index of Amplitude source reads.           |
|        |                                                                   |
|        | These values index into the Frequency table, used as "AM frq" and |
|        | "FM frq" when the respective modes are enabled.                   |
+--------+-------------------------------------------------------------------+
|        | Partitioning settings.                                            |
| 0xED7  |                                                                   |
|        | - bit 12-15: Frequency source partitioning.                       |
|        | - bit  8-11: Amplitude source partitioning.                       |
|        | - bit  4- 7: Sample source partitioning.                          |
|        | - bit  0- 3: Destination partitioning.                            |
|        |                                                                   |
|        | Encoding of partition sizes:                                      |
|        |                                                                   |
|        | - 0x0: 2 Words (4 samples)                                        |
|        | - 0x1: 4 Words (8 samples)                                        |
|        | - 0x2: 8 Words (16 samples)                                       |
|        | - 0x3: 16 Words (32 samples)                                      |
|        | - 0x4: 32 Words (64 samples)                                      |
|        | - 0x5: 64 Words (128 samples)                                     |
|        | - 0x6: 128 Words (256 samples)                                    |
|        | - 0x7: 256 Words (512 samples)                                    |
|        | - 0x8: 512 Words (1K samples)                                     |
|        | - 0x9: 1 KWords (2K samples)                                      |
|        | - 0xA: 2 KWords (4K samples)                                      |
|        | - 0xB: 4 KWords (8K samples)                                      |
|        | - 0xC: 8 KWords (16K samples)                                     |
|        | - 0xD: 16 KWords (32K samples)                                    |
|        | - 0xE: 32 KWords (64K samples)                                    |
|        | - 0xF: 64 KWords (128K samples)                                   |
+--------+-------------------------------------------------------------------+
| 0xED8  | Destination start pointer & partition select (addresses 16 bit    |
|        | word units). Updated during a mixer operation.                    |
+--------+-------------------------------------------------------------------+
|        | 64KWord bank selection settings (start address high bits).        |
| 0xED9  |                                                                   |
|        | - bit 12-15: Frequency source bank select.                        |
|        | - bit  8-11: Amplitude source bank select.                        |
|        | - bit  4- 7: Sample source bank select.                           |
|        | - bit  0- 3: Destination bank select.                             |
+--------+-------------------------------------------------------------------+
|        | Amplitude multiplier.                                             |
| 0xEDA  |                                                                   |
|        | - bit  9-15: Unused, cleared during a mixer operation.            |
|        | - bit     8: If set, the multiplier is not effective.             |
|        | - bit  0- 7: Amplitude multiplier.                                |
|        |                                                                   |
|        | In AM mode the last read Amplitude source will be written here.   |
|        | Note that the layout of this cell allows writing 0x100 (one       |
|        | higher than the greatest valid multiplier) to turn this           |
|        | multiplication off.                                               |
+--------+-------------------------------------------------------------------+
| 0xEDB  | Sample source start partition select bits.                        |
+--------+-------------------------------------------------------------------+
| 0xEDC  | Sample source start pointer whole part (addresses 16 bit word     |
|        | units). Updated during a Mixer operation.                         |
+--------+-------------------------------------------------------------------+
| 0xEDD  | Sample source start pointer fractional part. Updated during a     |
|        | Mixer operation.                                                  |
+--------+-------------------------------------------------------------------+
|        | Frequency select.                                                 |
| 0xEDE  |                                                                   |
|        | - bit  8-15: Unused, cleared during a mixer operation.            |
|        | - bit  0- 7: Frequency of Sample source reads.                    |
|        |                                                                   |
|        | The value indexes into the frequency table, fetching the "S. add" |
|        | value. In FM mode the last read Frequency source will be written  |
|        | here.                                                             |
+--------+-------------------------------------------------------------------+
|        | Start on write & DMA mode.                                        |
| 0xEDF  |                                                                   |
|        | - bit    15: FM mode enabled if set, the FM source is used.       |
|        | - bit    14: AM mode enabled if set, the AM source is used.       |
|        | - bit    13: Destination overwrite if set (otherwise sat. add).   |
|        | - bit  9-12: Unused.                                              |
|        | - bit  0- 8: Number of words to process; 0: 512 (1024 samples).   |
+--------+-------------------------------------------------------------------+

The Frequency table contains 256 entries of 16.16 fixed point in two separate
arrays indexed by 0xEDA and 0xEDB. The table is indexed by the "S. frq", "FM
frq" or "AM frq" values producing the respective "S. add", "FM add" and "AM
add" values.

Note that bits marked unused keep any value written to them unless otherwise
specified in the memory map.
