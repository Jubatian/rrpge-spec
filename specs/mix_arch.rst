
RRPGE Audio Mixer DMA peripheral architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The Mixer DMA peripheral assists audio generation by providing hardware
accelerated sample mixing capabilities. It operates on the Peripheral bus
accessing the Peripheral RAM, and is driven by the Mixer FIFO. By this it is
capable to work in parallel with the RRPGE CPU.




Mixer architecture
------------------------------------------------------------------------------


The mixer generates 16 bit (unsigned) digital audio suitable for output from
an arbitrary bit depth source (1 to 16 bits unsigned) within the Peripheral
RAM. It is capable to reduce the rate of the source using linear
interpolation.

Rate reduction works by a 16 bit register (Sample pointer fraction) which can
be incremented with an arbitrary value between 1 and 0x10000, triggering a
sample fetch when it wraps around.

Arbitrary source bit depth is realized using a 64 bit wide source input
register latching the source (32 bit) cell at the current bit offset and the
next cell, allowing to cross cell boundaries with a sample.

The logic of sample fetching could be realized as follows: ::


    Sample bit offset
       low 5 bits
            |
            V
    +----+----+----+----+----+----+----+----+
    | 32 bit curr. src. | 32 bit next src.  |  64 bit source input register
    +----+----+----+----+----+----+----+----+
            |     |
            V     V
            +-----+
            |     |  Source sample (1 - 8 bits or 16 bits)
            +-----+
               |
               |  Sample expansion to 16 bits
               V
          +----+----+
          | 16 bits |  Expanded sample
          +----+----+
               |
               |    +----+----+     +----+----+
               |    |  Next   |---->| Current |  Samples latched for
               |    +----+----+     +----+----+  interpolation
               |         A
               |         | (After loading Current)
               +---------+


The Sample bit offset afterwards is incremented by the Sample bit width, then,
if the low 5 bits wrapped, the Source input register is shifted left by 32.

Sample expansion is performed by copying the sample repeatedely into the lower
bits until all bits are filled. For example a 6 bit input is expanded as
follows: ::


    +---+---+---+---+---+---+
    | 0 | 1 | 2 | 3 | 4 | 5 | Source sample (6 bits)
    +---+---+---+---+---+---+
                |
                +-----------------------+-----------------------+
                |                       |                       |
                V           |           V           |           V
    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | 0 | 1 | 2 | 3 | 4 | 5 | 0 | 1 | 2 | 3 | 4 | 5 | 0 | 1 | 2 | 3 |
    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+


Linear interpolation works from the latched current and next samples, using
the high 3 bits of the Sample pointer fraction register. It so realizes 7
intermediate steps between the two points (the current and next sample
values).

In every processing cycle, two destination samples are produced to fill a PRAM
cell, accordingly the necessary source logic is also performed twice.

Before starting a mixer operation, the following initialization steps are
performed:

- Fetch current source cell.
- Fetch next source cell, completing the 64 bit source input register.
- Fetch current sample (using the sample fetch logic).
- Fetch next sample (using the sample fetch logic).

Note that sample fetching this way is always one sample ahead. The PRAM cell
offset at source fetches is 32 bits (one cell) ahead compared to the sample
pointer (it may be generated from the sample pointer in this manner).

Then the main processing is started, performing as many processing cycles
(sample pairs) as required. The logic of a processing cycle is as follows:

- Fetch next source (32 bit PRAM cell), done even if it is not necessary.
- Update next source bits in the 64 bit source input register.
- Perform interpolation to generate first result sample.
- Increment Sample pointer fraction.
- If wrapped, do a Sample fetch.
- Perform interpolation to generate second result sample.
- Increment Sample pointer fraction.
- If wrapped, do a Sample fetch.
- Combine the two result samples (2 x 16 bits).
- Apply amplitude on the result samples.
- Add (saturated) Amplitude multiplier add value to amplitude.
- Fetch next destination (32 bit PRAM cell).
- Add (saturated) result samples to destination (if enabled).
- Write back destination.

The actual order of memory accesses within a processing cycle may be
different, and may interleave with other passes in an implementation defined
manner.

The Amplitude multiplier is applied to the 16 bit source data as follows:

src_a = (((src - 32768) * amp) / 65535) + 32768

Without relying on signed arithmetic this may be expressed as:

src_a = (((src * amp) >> 15) + 65536 - amp) >> 1

If adding to the destination is enabled, the result forms as follows:

dest = satu(src_a + dest - 32768)

The saturation trims the result to the 16 bit range (0x0000 - 0xFFFF).




Mixer operation timing
------------------------------------------------------------------------------


The mixer should be designed so the necessary memory accesses dominate its
timing using appropriate pipelining and implementation. Note that the layout
of memory accesses is implementation defined.

To perform a processing cycle (2 samples), 3 memory accesses (one source read,
one destination read, and one destination write) are necessary, which makes
6 main clock cycles. In overall the following formula should give the cycles
necessary for a mixer operation:

30 + (6 * n)

Where 'n' is the count of processing cycles to perform (so taking 3 cycles /
sample).




Mixer peripheral memory map
------------------------------------------------------------------------------


The following table describes the registers of the Mixer DMA. These
registers are only accessible through the Mixer FIFO (see "fifo.rst" for
details).

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 |                                                                   |
| \-     | Unused.                                                           |
| 0x0004 |                                                                   |
+--------+-------------------------------------------------------------------+
|        | Destination bank select.                                          |
| 0x0005 |                                                                   |
|        | - bit  4-15: Unused                                               |
|        | - bit  0- 3: Destination bank select.                             |
+--------+-------------------------------------------------------------------+
| 0x0006 | Destination start pointer (addresses 32 bit cell units). Note     |
|        | that destination wraps around on PRAM bank boundary.              |
+--------+-------------------------------------------------------------------+
|        | Destination cell count.                                           |
| 0x0007 |                                                                   |
|        | - bit    15: Destination overwrite if clear (otherwise sat. add). |
|        | - bit 12-14: Unused                                               |
|        | - bit  0-11: Number of cells to process; 0: 4096 (8192 samples).  |
|        |                                                                   |
|        | Bit 15 becomes set after a Mixer operation. This simplifies       |
|        | usual mixing processes, only necessiting a single write to this   |
|        | register.                                                         |
+--------+-------------------------------------------------------------------+
|        | Source configuration.                                             |
| 0x0008 |                                                                   |
|        | - bit    15: If set, sample width is 16 bits.                     |
|        | - bit 12-14: Sample width in bits (0: 1 bit; 7: 8 bits).          |
|        | - bit  5-11: Unused                                               |
|        | - bit     4: If set, no partitioning is used (full PRAM).         |
|        | - bit  0- 3: Source partition size.                               |
|        |                                                                   |
|        | Narrower than 16 bits samples are expanded to 16 bits by copying  |
|        | them repeatedely on the lower bits (for example a 6 bit sample of |
|        | 0x20: 0b100000 would give 0x8208: 0b1000001000001000 in 16 bits). |
|        |                                                                   |
|        | Source partition sizes are as follows:                            |
|        |                                                                   |
|        | - 0x0: 1 Cell (32 bits)                                           |
|        | - 0x1: 1 Cell (32 bits)                                           |
|        | - 0x2: 1 Cell (32 bits)                                           |
|        | - 0x3: 1 Cell (32 bits)                                           |
|        | - 0x4: 1 Cell (32 bits)                                           |
|        | - 0x5: 2 Cells (64 bits)                                          |
|        | - 0x6: 4 Cells (128 bits)                                         |
|        | - 0x7: 8 Cells (256 bits)                                         |
|        | - 0x8: 16 Cells (512 bits)                                        |
|        | - 0x9: 32 Cells (1K bits)                                         |
|        | - 0xA: 64 Cells (2K bits)                                         |
|        | - 0xB: 128 Cells (4K bits)                                        |
|        | - 0xC: 256 Cells (8K bits)                                        |
|        | - 0xD: 512 Cells (16K bits)                                       |
|        | - 0xE: 1024 Cells (32K bits)                                      |
|        | - 0xF: 2048 Cells (64K bits)                                      |
|        |                                                                   |
|        | If bit 4 is set (partitioning is turned off), the whole Sample    |
|        | bit pointer increments, covering the full Peripheral RAM. If the  |
|        | bit is clear, partitioning is used, disabling carry-over into bit |
|        | 16, and using as many high bits from Sample partition select as   |
|        | required to produce the desired partition size.                   |
+--------+-------------------------------------------------------------------+
|        | Sample partition select bits. Aligns with Sample bit pointer,     |
| 0x0009 | low, providing the higher fixed bits of it in partitioned modes.  |
|        | If partitioning is enabled, only the low 16 bits of the Sample    |
|        | bit pointer increment (there is no carry-over to Sample bit       |
|        | pointer, high).                                                   |
+--------+-------------------------------------------------------------------+
|        | Sample pointer fraction add value, 0: 65536. The sample bit       |
| 0x000A | pointer is incremented with sample width when the sample pointer  |
|        | fraction wraps.                                                   |
+--------+-------------------------------------------------------------------+
| 0x000B | Sample pointer fraction.                                          |
+--------+-------------------------------------------------------------------+
|        | Amplitude multiplier add value.                                   |
| 0x000C |                                                                   |
|        | Signed 2's complement value which is added to the amplitude       |
|        | multiplier after each destination write (so after every two       |
|        | samples). This operation is performed with saturation, limiting   |
|        | amplitude between 1 and 0x10000 inclusive.                        |
+--------+-------------------------------------------------------------------+
|        | Initial amplitude multiplier.                                     |
| 0x000D |                                                                   |
|        | If it is zero, the multiplier is not effective (source goes into  |
|        | destination unchanged). Otherwise the 16 bit source is multiplied |
|        | with this value into 32 bits, then the high 16 bits of that is    |
|        | propagated towards the destination.                               |
+--------+-------------------------------------------------------------------+
| 0x000E | Sample bit pointer, high (Low 9 bits effective).                  |
+--------+-------------------------------------------------------------------+
| 0x000F | Sample bit pointer, low & Start trigger.                          |
+--------+-------------------------------------------------------------------+

Note that no interface register changes it's value during the course of a
Mixer DMA operation, so retriggering the mixer performs the exact same
operation.
