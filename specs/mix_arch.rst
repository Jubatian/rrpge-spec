
RRPGE Audio Mixer DMA peripheral architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The Mixer DMA peripheral assists audio generation by providing hardware
accelerated sample mixing capabilities including amplitude modulation. It
operates on the Peripheral bus accessing the Peripheral RAM, and is driven by
the Mixer FIFO. By this it is capable to work in parallel with the RRPGE CPU.




Mixer architecture
------------------------------------------------------------------------------


The mixer operates on up to two 8 bit sources and one 8 bit destination within
the Peripheral RAM. One of the sources is optional, used for amplitude
modulation. The sources may have individual frequency settings.

The following block diagram depicts the overall architecture of the mixer: ::


    +----DRAM----+
    |            |
    | +--------+ |
    | | Sample |-------------Data-----------------------------------------+
    | | source | |  +------+   _  +----------------------+                |
    | |        |<---| Ptr. |<-|+|-|   Sample frequency   |                |
    | +--------+ |  +------+   ~  +----------------------+                |
    |            |                                                        |
    | +--------+ |                                                        |
    | |   AM   |-------------Data--------------------------+              |
    | | source | |  +------+   _  +----------------------+ |              |
    | |        |<---| Ptr. |<-|+|-|     AM frequency     | |              |
    | +--------+ |  +------+   ~  +----------------------+ |              |
    |            |                                 :       |              |
    |            |          +-------------+  +-----------+ V  +--------+  |
    |            |          | Add On/Off  |  | AM On/Off |-+->| A. mul |>|*|
    |            |          +-------------+  +-----------+    +--------+  |
    | +--------+ |                 |   _                                  |
    | | Sample |-------------Data->+<-|0|                                 |
    | | dest.  | |                 V   ~                                  |
    | |        |<------------Data-|+|-------------------------------------+
    | +--------+ |                 ~
    |            |
    +------------+


In one pass, 4 samples are processed to produce a 32 bit destination in one
operation. The AM source however is read only once each pass. To negate the
difference in processing rate, the AM frequency is multiplied by 4 (shifted
left by 2). Step by step, the following actions are performed during a pass:

- Fetch first sample.
- Increment sample pointer with Sample frequency.
- Fetch second sample.
- Increment sample pointer with Sample frequency.
- Fetch third sample.
- Increment sample pointer with Sample frequency.
- Fetch fourth sample.
- Increment sample pointer with Sample frequency.
- Merge the fetched sample values in a 32 bit holding register.
- Fetch AM source, update Amplitude multiplier with it (if AM is enabled).
- Increment AM source pointer with AM frequency * 4 (if AM is enabled).
- Packed multiply the 32 bit holding register with Amplitude multiplier.
- Read sample destination (32 bits).
- Packed add the read value to the 32 bit holding register (if it is enabled).
- Write the 32 bit holding register in the sample destination.
- Increment destination pointer.

The actual order of memory accesses within a pass may be different, and may
interleave with other passes in an implementation defined manner.

For both sources and the destination a partition size setting is provided, one
for each. This allows for selecting sample data sizes from 8 samples (2 cells)
to 256K samples (64K cells) in power of 2 increments. The pointers
automatically wrap to the beginning of the partition when passing it's end.

The Amplitude multiplier is applied to the 8 bit source data as follows:

src_a = (((src - 128) * amp) / 256) + 128

Without relying on signed arithmetic this may be expressed as:

src_a = (((src * amp) >> 7) + 256 - amp) >> 1

Note that this algorithm implies that the maximal valid amplitude of 0xFF can
not reproduce the original data. The Amplitude multiplier has to be turned off
to keep the data as-is.

If adding to the destination is enabled, the result forms as follows:

dest = satu(src_a + dest - 128)

The saturation trims the result to the 8 bit range (0x00 - 0xFF).




Mixer operation timing
------------------------------------------------------------------------------


The timing of the mixer operations is consistent for all configurations,
dominated by the necessary memory accesses (7 for each pass). A mixer
operation takes the following amount of main clock cycles:

16 + (14 * n)

'n' is the number of passes to process (so the operation takes 3.5 cycles /
sample). Note that the two cycles necessary for reading the Amplitude source
are present even when the AM source is turned off.




Mixer peripheral memory map
------------------------------------------------------------------------------


The following table describes the registers of the Mixer DMA. These
registers are only accessible through the Mixer FIFO (see "fifo.rst" for
details).

The Mixer is accessed by a 4 bit address. The addresses however are provided
with bit 15 set as this is how they should be supplied to the FIFO.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x8000 | Amplitude source partition select bits. Used in AM mode for       |
|        | reading the amplitude source.                                     |
+--------+-------------------------------------------------------------------+
| 0x8001 | Amplitude source start pointer whole part (addresses 32 bit cell  |
|        | units). Used in AM mode for reading the amplitude source.         |
+--------+-------------------------------------------------------------------+
| 0x8002 | Amplitude source start pointer fractional part. Used in AM mode   |
|        | for reading the amplitude source.                                 |
+--------+-------------------------------------------------------------------+
| 0x8003 | Frequency for AM source read, whole part. Provides the increment  |
|        | for the AM source pointer.                                        |
+--------+-------------------------------------------------------------------+
| 0x8004 | Frequency for AM source read, fractional part. Provides the       |
|        | increment for the AM source pointer.                              |
+--------+-------------------------------------------------------------------+
|        | Partitioning settings.                                            |
| 0x8005 |                                                                   |
|        | - bit 12-15: Unused                                               |
|        | - bit  8-11: Amplitude source partitioning.                       |
|        | - bit  4- 7: Sample source partitioning.                          |
|        | - bit  0- 3: Destination partitioning.                            |
|        |                                                                   |
|        | Encoding of partition sizes:                                      |
|        |                                                                   |
|        | - 0x0: 2 Cells (8 samples)                                        |
|        | - 0x1: 4 Cells (16 samples)                                       |
|        | - 0x2: 8 Cells (32 samples)                                       |
|        | - 0x3: 16 Cells (64 samples)                                      |
|        | - 0x4: 32 Cells (128 samples)                                     |
|        | - 0x5: 64 Cells (256 samples)                                     |
|        | - 0x6: 128 Cells (512 samples)                                    |
|        | - 0x7: 256 Cells (1K samples)                                     |
|        | - 0x8: 512 Cells (2K samples)                                     |
|        | - 0x9: 1 KCells (4K samples)                                      |
|        | - 0xA: 2 KCells (8K samples)                                      |
|        | - 0xB: 4 KCells (16K samples)                                     |
|        | - 0xC: 8 KCells (32K samples)                                     |
|        | - 0xD: 16 KCells (64K samples)                                    |
|        | - 0xE: 32 KCells (128K samples)                                   |
|        | - 0xF: 64 KCells (256K samples)                                   |
+--------+-------------------------------------------------------------------+
|        | 64 KCell bank selection settings (start address high bits).       |
| 0x8006 |                                                                   |
|        | - bit 12-15: Unused                                               |
|        | - bit  8-11: Amplitude source bank select.                        |
|        | - bit  4- 7: Sample source bank select.                           |
|        | - bit  0- 3: Destination bank select.                             |
+--------+-------------------------------------------------------------------+
| 0x8007 | Destination partition select bits.                                |
+--------+-------------------------------------------------------------------+
| 0x8008 | Destination start pointer (addresses 32 bit cell units).          |
+--------+-------------------------------------------------------------------+
|        | Amplitude multiplier.                                             |
| 0x8009 |                                                                   |
|        | - bit  9-15: Unused                                               |
|        | - bit     8: If set, the multiplier is not effective.             |
|        | - bit  0- 7: Amplitude multiplier.                                |
|        |                                                                   |
|        | Used only if AM mode is disabled.                                 |
|        |                                                                   |
|        | Note that the layout of this register allows writing 0x100 (one   |
|        | higher than the greatest valid multiplier) to turn this           |
|        | multiplication off.                                               |
+--------+-------------------------------------------------------------------+
| 0x800A | Sample source partition select bits.                              |
+--------+-------------------------------------------------------------------+
| 0x800B | Sample source start pointer whole part (addresses 32 bit cell     |
|        | units).                                                           |
+--------+-------------------------------------------------------------------+
| 0x800C | Sample source start pointer fractional part.                      |
+--------+-------------------------------------------------------------------+
| 0x800D | Frequency, whole part. Provides the increment for the Sample      |
|        | source pointer.                                                   |
+--------+-------------------------------------------------------------------+
| 0x800E | Frequency, fractional part. Provides the increment for the Sample |
|        | source pointer.                                                   |
+--------+-------------------------------------------------------------------+
|        | Mode & Start trigger.                                             |
| 0x800F |                                                                   |
|        | - bit    15: Destination overwrite if set (otherwise sat. add).   |
|        | - bit    14: AM mode enabled if set, the AM source is used.       |
|        | - bit 10-13: Unused                                               |
|        | - bit  0-11: Number of cells to process; 0: 4096 (16384 samples). |
+--------+-------------------------------------------------------------------+

If partitioning settings are set to anything other than 64 KCells for a
pointer, the appropriate (high) bits of the matching whole part register are
ignored, and the partition select's matching bits are used instead for
generating the address.

Note that no interface register changes it's value during the course of a
Mixer DMA operation, so retriggering the mixer performs the exact same
operation.
