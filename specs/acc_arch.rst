
Graphics Accelerator & 16 <=> 32 VRAM interface architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the Graphics Accelerator and the 16 <=>
32 VRAM interface components of the RRPGE system.

The Graphics Accelerator component provides for hardware accelerated graphics
operations such as sprite blitting, scaling, and filling.

The Accelerator component is capable to operate in parallel with the CPU and
any peripheral operating on the CPU bus as long as they don't attempt to
access the Video bus. If so, the CPU is stalled until the Video bus becomes
free. Using the Graphics FIFO complex rendering tasks may be executed in
parallel with the CPU's other tasks.

The 16 <=> 32 VRAM interface is also described here since it shares some
concepts with the operation of the Accelerator.




16 <=> 32 VRAM interface
------------------------------------------------------------------------------


The Video memory is connected to a 32 bit Video bus, so to provide access to
it by the CPU, data conversion has to be performed.

The lowest bit of the CPU's address is used to select the upper or lower half
of a 32 bit Video RAM cell, while the rest shifted right by one provide the
address for the cell. If the lowest bit of the address is zero, the high 16
bits of the cell are selected, otherwise the low 16 bits (this accords with
the Big Endian scheme the system uses).

When the CPU performs a Read access on the Video RAM, the targeted 32 bit
cell's value is latched, and the appropriate half of the cell is put on the
data bus.

Writes initiated by the CPU are assisted by the latch. Since the CPU can only
write 16 bits at a time, the other half of the data to be written into the 32
bit Video memory cell has to be provided from elsewhere. The source for this
data is the latch, which is enforced by the temporary automatic modification
of the VRAM Write mask.

The write logic when writing into the high half of a Video memory cell (the
lowest address bit from the CPU is zero) operates as follows: ::


    +----+----+
    | CPUdata | (16 bits from the CPU bus)
    +----+----+
         |
         V                   +----+----+----+----+
    +----+----+----+----+    |  VRAM Write mask  |    +----+----+----+----+
    |  data   | 00   00 |    +----+----+----+----+    |  Data from latch  |
    +----+----+----+----+         |                   +----+----+----+----+
              |                   V                             |
             _V_             +----+----+----+----+     ___     _V_
            |AND|<-----------|  mask   | 00   00 |--->|NEG|-->|AND|
             ~|~             +----+----+----+----+     ~~~     ~|~
              |                                                 |
              |                       ___                       |
              +--------------------->| OR|<---------------------+
                                      ~|~
                                       V
                          ---+----+----+----+----+---
                             | Target VRAM cell  |
                          ---+----+----+----+----+---


For the lower half (the lowest address bit from the CPU is one) the logic is
similar, just substituting zeros on the high half instead.

Note that the CPU when writing always performs an R-M-W operation as defined
in "Memory accessing" in "cpu_arch.rst". Moreover in the RRPGE system banking
in a Video memory page for writing is only allowed if the respective page is
also banked in for read on the same location.

These properties allow for simplifying the latching logic greatly in emulators
as from the CPU's point of view normal 16 bit accesses happen only affected by
the Video RAM Write mask.




Accelerator overview
------------------------------------------------------------------------------


The Accelerator component of the Graphics Display & Accelerator unit is
basically a very flexible blitter capable of operating in four distinct major
modes:

- (BB) Block blitter, combining a source area onto a destination.
- (FL) Filler, combining a source pattern onto a destination.
- (SC) Scaled blitter, combining a source area onto a destination.
- \(LI) Line, capable to draw lines by start point, direction and length.

On the source in "BB" and "SC" modes the following transformations may be
applied:

- Barrel rotation per pixel.
- AND & OR mask per pixel.
- Reindexing using a reindex table (color remapping).

All three modes are capable to combine the source and the destination together
in two ways before writing back to the destination. These are as follows:

- Colorkeying (a designated transparent color index).
- Reindexing by destination (color remapping).

Some more advanced examples of usages of the Accelerator are as follows:

- Using the AND mask per pixel and optionally the barrel rotation per pixel
  feature a given area of the Video memory may be utilized to hold two
  overlaid low bit depth images used separately. This allows for better
  utilization of memory.

- Using the AND and OR masks with an appropriate palette may be used to
  generate player-specific colored variants of objects without needing
  reindexing.

- The Filler may be used for polygon filling, especially by substituting the
  destination and count of pixels per line from the source X and Y pointers.

- The Scaled blitter may be utilized for simple texture blitting for three
  dimensional scenes.

- Reindexing by destination with an appropriate color remapping table may be
  utilized for alpha blending without needing to split up the display into
  separate bit planes.




Accelerator architecture
------------------------------------------------------------------------------


Since the Accelerator can be configured in a great variety of way, it is not
feasible to describe all types of operations individually. The Accelerator is
so broken up in stages, and each stage is described as an unit while defining
the ways how these stages may be coupled to perform an accelerator operation.

The various stages are enabled or disabled depending on the accelerator's
configuration. The essential configuration variables are outlined below which
affect how the accelerator stages are chained together:

- (VMD) The mode selector which selects from the four main accelerator modes,
  the Block Blitter (BB), the Filler (FL), the Scaled Blitter (SC) and the
  Line (LI) mode.

- (VMR) Adds a mirror stage to the Block Blitter (BB) which inverts the pixel
  order of the source data. Also available for the Scaled Blitter (SC).

- (VCK) Colorkey stage which can be applied to all modes. This provides a
  transparent pixel value where the background shows through.

- (VRE) Reindex stage which may be applied to all modes. This remaps pixel
  values according to a table without affecting the colorkey stage.

- (VDR) Reindex using destination. This extends the reindex stage by involving
  the destination data to select from a larger table.

The Accelerator has two major stages as follows:

- Source fetch. This stage is performed according to the the selected mode
  (VMD), giving three possible distinct paths.

- Destination combine. This stage varies according to whether reindexing is
  necessary (VRE), giving two possible distinct paths.




Source fetch major stage
------------------------------------------------------------------------------


The source fetch stage prepares the source data performing any transforms
possible on it without the knowledge of the destination. For each Video RAM
cell necessarily affected it prepares a Video RAM cell aligned data and a cell
begin / middle / end mask.

The latter is prepared according to the destination start pointer and the
count of units to process, bits from the latter used according to the display
mode (lowest bit ignored in 8 bit display mode). In Line (LI) mode this mask
always selects a single pixel on the destination cell.

Note that for short blits the begin and end of the blit may occur in the same
cell. This situation also has to be supported proper.

The mode selector (VMD) defines the path to take executing this stage. Only
VMR may have effect on the execution otherwise.


Source offset calculation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All modes except Filler (FL) (which has no source) share an identical source
offset mechanism. Note that in Line (LI) mode this mechanism is used to
generate destination offsets and begin / middle / end masks, but it still
executes the same way.

The source offset when needed, is combined from the components according to
the following chart: ::


                      |<- Source partition size ->|
                      |                           |
                      |           |<- X/Y split ->|
                      |           |               |
         +------------+-----------+---------------+--------------------------+
         | P.sel bits |  Y bits   |    X bits     |          X bits          |
    +----+------------+-----------+---------------+--------------------------+
    |Bank|          Whole part (16 bits)          | Fractional part (16 bits)|
    +----+----------------------------------------+--------------------------+


The source partition size has higher priority (only it affects the number of
partition select bits, even if X/Y split is larger).

Block Blitter (BB) mode performs a source increment after each cell fetched,
while Scaled Blitter (SC) and Line (LI) modes perform a source increment after
each pixel.


Block Blitter (BB)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Block Blitter normally produces a horizontal strip of sequentially read
data beginning at an arbitrary position. The source data may begin at an
arbitrary location, and the blit may have an arbitrary length in pixels. The
begin position is handled as a part of the incrementing logic, detailed
further below.

Preparing the source data requires a memory of the previous data to be able
to shift it according to the destination start pointer's fractional part. For
the first source read this data is undefined and irrelevant (it will be masked
out). The data from each source cell is prepared as follows: ::


    +----+----+----+----+
    |    Source data    | As read from the Video RAM
    +----+----+----+----+
              |
              V
    +-------------------+
    | Px. barrel rotate | Barrel rotates each pixel by the given count
    +-------------------+
              |
              V
    +-------------------+
    |  Read AND/OR mask | Applies the Read AND & OR masks on each pixel
    +-------------------+
              |
              V
    +-------------------+
    | Pixel order swap  | If VMR is enabled (Mirroring)
    +-------------------+
              |
              V
    +----+----+----+----+
    |  Transformed src. |
    +----+----+----+----+
              |
              +------------+ Shift to align with destination
                           V
    +----+----+----+----+----+----+----+----+
    | Prev. src. |   Current source  |      | Shift register
    +----+----+----+----+----+----+----+----+
              |
              V
    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+


The Block Blitter upon entry calculates the destination alignment shift. This
is taken from the destination fraction after subtracting the source fraction
from it. If the subtraction wraps around, the first source fetch stage
terminates after the write into the shift register (the destination combine
stage is not started). Note that the calculation of the begin mask is not
affected, and the mask applies on the first executed destination combine.


Filler (FL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Filler normally produces a horizontal line of an arbitrary length (in
pixels) of an uniform source pattern.

The source data is prepared as follows: ::


    +----+----+
    | Pattern | 16 bit line pattern
    +----+----+
         |
         +---------+
         V         V
    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+


Scaled blitter (SC)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Scaled Blitter normally produces a horizontal strip of data beginning at
an arbitrary position from evenly spaced out source pixels of arbitrary length
in pixels.

This mode taps in the Block Blitter (BB) producing source cell data for it
pixel by pixel using the whole and fractional source position and increment
parameters.

The data is prepared as follows: ::


    +----+ +----+ +----+ +----+
    | Px | | Px | | Px | | Px | Up to 8 4bit pixels or 4 8bit pixels
    +----+ +----+ +----+ +----+
      |      |      |      |
      |    +-+      |      |
      |    |    +---+      |
      |    |    |    +-----+
      |    |    |    |
    +----+----+----+----+
    |    Source data    | Aligned with the destination cells
    +----+----+----+----+
              |
              V
    +-------------------+
    |   Block Blitter   |
    +-------------------+


Line (LI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Line mode has no source, however it uses the source offset mechanism to
produce destination pixels. Note that even the partitioning settings are
reversed (so the source partition setting applies to the destination). The
begin / middle / end mask is used for every pixel to select the destination
pixel within the cell for the Destination combine major stage.

The line pattern is used to produce the line's color. The pattern is rotated
one pixel (4 or 8 bits) after every two pixels output, always using the lowest
pixel (4 or 8 bits) for the output.




Destination combine major stage
------------------------------------------------------------------------------


The destination combine stage uses the prepared source ("Data to blit") and
the begin / middle / end mask for blitting it onto the destination. The VCK,
VRE and VDR configuration variables affect how this stage is performed.

VRE (Reindex) selects from the two possible paths in this stage.


No reindex blit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This path is used if VRE is disabled (no reindexing). This case VDR is
ignored. The data is blit as follows: ::


    +----+----+----+----+  If VCK   +----+----+----+----+
    |    Data to blit   |---------->|   Colorkey mask   |
    +----+----+----+----+           +----+----+----+----+
              |                               |
              |         +----+----+----+----+ | +----+----+----+----+
              |         |  VRAM Write mask  | | |  Beg/Mid/End mask |
              |         +----+----+----+----+ | +----+----+----+----+
              |                   |          _V_          |
              |                   +-------->|AND|<--------+
             _V_                             ~|~
            |AND|<----------------------------+
             ~|~                              |
             _V_       ___                   _V_
            | OR|<----|AND|<----------------|NEG|
             ~|~       ~A~                   ~~~
              V         |
      ---+----+----+----+----+---
         | Target VRAM cell  |
      ---+----+----+----+----+---


Reindexing blit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This path is used if VRE is enabled (reindex mode). This case if VDR is also
enabled, the path feeding in the target VRAM cell's data is also effective and
is used for providing the high bits (up to 5) for selecting a new pixel value
from the reindex table. ::


    +----+----+----+----+  If VCK   +----+----+----+----+
    |    Data to blit   |---------->|   Colorkey mask   |
    +----+----+----+----+           +----+----+----+----+
              |                               |
              |         +----+----+----+----+ | +----+----+----+----+
              |         |  VRAM Write mask  | | |  Beg/Mid/End mask |
              |         +----+----+----+----+ | +----+----+----+----+
              |                   |          _V_          |
              |                   +-------->|AND|<--------+
              V                              ~|~
    +-----------------------------+           |
    |   Reindex (enabled by VRE)  |           |
    +-----------------------------+           |
              |     A                         |
              |     | If VDR                  |
             _V_    |                         |
            |AND|<-)|(------------------------+
             ~|~    |                         |
             _V_    |  ___                   _V_
            | OR|<--+-|AND|<----------------|NEG|
             ~|~       ~A~                   ~~~
              V         |
      ---+----+----+----+----+---
         | Target VRAM cell  |
      ---+----+----+----+----+---


Accelerated combine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For every destination combine, the combined mask is checked. If the mask
is all set (all bits are to be taken from the source), and VDR (reindex by
destination) is unset, the destination data read is omitted, saving 2
cycles if possible (reindexing might stall the pipeline negating this).




Finalizing the row
------------------------------------------------------------------------------


When the row is complete, the original values of the source and destination
pointers are incremented by the contents of the appropriate post-add
registers, and are used to start the next row. These increments happen in all
modes.

Note that intermediate increments (described in "Source offset calculation")
performed during the blit are all discarded.




Minor stages explained
------------------------------------------------------------------------------


This chapter explains some of the minor stages of the accelerator.


Pixel order swap (Mirror: VMR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This stage swaps the pixel order. It behaves differently depending on the
display mode, as shown on the following charts: ::


    4 bit mode                        8bit mode
    +--+--+--+--+--+--+--+--+         +-----+-----+-----+-----+
    |P0|P1|P2|P3|P4|P5|P6|P7|         | P0  | P1  | P2  | P3  |
    +--+--+--+--+--+--+--+--+         +-----+-----+-----+-----+
                |                                 |
                |    Pixel order swap (Mirror)    |
                V                                 V
    +--+--+--+--+--+--+--+--+         +-----+-----+-----+-----+
    |P7|P6|P5|P4|P3|P2|P1|P0|         | P3  | P2  | P1  | P0  |
    +--+--+--+--+--+--+--+--+         +-----+-----+-----+-----+


Note that the other source transforms (Read AND & OR mask and barrel rotate)
also behave in a similar manner, on pixel level.


Reindex (VRE and VDR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Re-indexes each pixel using a table within the Accelerator component. It
operates as follows on pixel level (differently for 4 bit and 8 bit modes): ::


    +------+                  +---------------------+
    | S.Px | Old pixel value  | Reindex bank select |
    +------+                  +---------------------+
       |                         |
       +------------------------)|(----+
                                 |     |
                                 V     V
                              +-----+----+
                              | Tb. Addr | 9 bit reindex table address
                              +-----+----+
                                    |
                                    V
                            ----+--------+----
                                | New px |     Reindex table (512 x 8bit)
                            ----+--------+----
                                    |
       +----------------------------+
       V
    +------+
    |  Px  | New pixel value stored
    +------+


The operation is performed at pixel level. In 4 bit mode the Source pixel
(S.Px) is used as-is, in 8 bit mode however it's high bits are discarded (so
in either mode only 4 bits from the pixel may be used to index the table).

The reindex table contains 8 bit entries. In 4 bit mode the high 4 bits of
these entries are discarded before writing back.

If VDR is also enabled, instead of the "Reindex bank select" peripheral
register the low 5 bits of the destination's appropriate pixel is used after
applying write masks. In 4 bit mode the highest bit of the table address is
always zero if VDR is enabled.


Colorkey (VCK)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Colorkeying selects a color index for which the source should be masked out.
This stage works by testing each pixel's value for equivalence with the
colorkey, building a colorkey mask as follows:

- If the pixel's value equals the colorkey, corresponding bits are cleared.
- Otherwise corresponding bits in the mask are set.

This mask is then combined with the other write masks as defined in the paths
of the Destination combine major stage.




Implementation defined
------------------------------------------------------------------------------


The following notable aspects of the operation of the accelerator are
implementation defined:

- The result of operations where the source overlaps the destination if
  sequentially a source read from a cell would happen after a destination
  write. This case due to the implementation defined length of the pipeline
  the source read may fetch not yet changed data.

- If VMR is used with Scaled Blit and the count of pixels to blit is not a
  multiple of 4 (8 bit mode) or 8 (4 bit mode), for the last source cell
  pixels not filled may have an implementation defined content (typically
  either zero or data left over from a previous operation). Note that this
  data does not become visible unless VMR is set.

- The exact location and order of accesses during the operation. Emulators are
  allowed to perform the entire accelerator operation in one pass, without
  considering the Graphics Display Generator's operation.

Note that the timing once it meets the minimal requirements is also
implementation defined.




Accelerator operation timing
------------------------------------------------------------------------------


The accelerator is designed to perform one 32 bit memory access on the Video
RAM every second cycle (interleaved with the Graphics Display Generator's
accesses) at it's peak rate. Most of the modes are pipelined to perform by
this rule except when delayed by reindexing.

Reindexing can be performed at one pixel per cycle irrespective of whether the
destination has to be accessed for it (VDR enabled) or not.

Following the performance (in main clock cycles) for each of the eight major
stage combinations are provided. 'n' is the Video RAM cell count which has to
be written during the operation, 'p' is the count of pixels to render, 'r' is
the number of rows to render. In the Accel. combine column only the 'n' member
is shown where appropriate.

+------+------+-----+------------------------------------+-------------------+
| Disp | Mode | VRE | Cycles                             | Accel. combine    |
+======+======+=====+====================================+===================+
| 4bit |  BB  | NO  | 20 + (r * 8) + (n * 6)             | n * 4             |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  FL  | NO  | 20 + (r * 8) + (n * 4)             | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  SC  | NO  | 20 + (r * 8) + (n * 4) + (p * 2)   | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  LI  | NO  | 20 + (r * 8)           + (p * 4)   | \-                |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  BB  | YES | 28 + (r * 8) + (n * 8) (*)         | n * 8 (*)         |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  FL  | YES | 28 + (r * 8) + (n * 8) (*)         | n * 8 (*)         |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  SC  | YES | 28 + (r * 8) + (n * 4) + (p * 2)   | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 4bit |  LI  | YES | 28 + (r * 8)           + (p * 4)   | \-                |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  BB  | NO  | 20 + (r * 8) + (n * 6)             | n * 4             |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  FL  | NO  | 20 + (r * 8) + (n * 4)             | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  SC  | NO  | 20 + (r * 8) + (n * 4) + (p * 2)   | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  LI  | NO  | 20 + (r * 8)           + (p * 4)   | \-                |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  BB  | YES | 28 + (r * 8) + (n * 6)             | n * 4             |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  FL  | YES | 28 + (r * 8) + (n * 4)             | n * 4 (*)         |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  SC  | YES | 28 + (r * 8) + (n * 4) + (p * 2)   | n * 2             |
+------+------+-----+------------------------------------+-------------------+
| 8bit |  LI  | YES | 28 + (r * 8)           + (p * 4)   | \-                |
+------+------+-----+------------------------------------+-------------------+

Note that in 4 bit mode 8 reindexing accesses are necessary for processing
each Video RAM cell while in 8 bit mode 4 such accesses are necessary. Modes
where this determines the performance are marked with a '*'.

Note that the Accelerated combine may be in effect for any processed cell if
it's conditions are met. In Line mode the conditions of it can never be met.

In some cases the Block Blitter may need one more source fetch in a row than
destination combines. The cycles (two) taken for this fetch should be
interpreted as part of the row transition cycles.




Accelerator memory map
------------------------------------------------------------------------------


The following table describes those elements of the graphics registers which
are related to the Accelerator component. Note that these registers are only
accessible through the Graphics FIFO (see "grapfifo.rst" for details).

The graphics registers in the 0x000 - 0x0FF range repeat every 32 words, so
for example the address 0x020 also refers to the register at 0x000.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Unused.                                                           |
| 0x003  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x004  | Video RAM write mask (0x000: High, 0x001: Low). Clear bits in it  |
| \-     | mask writes to the respective positions both for the 16 <=> 32    |
| 0x005  | VRAM interface and the Accelerator.                               |
+--------+-------------------------------------------------------------------+
|        | Source bank & partition select.                                   |
| 0x006  |                                                                   |
|        | - bit  2-15: Partition select bits                                |
|        | - bit  0- 1: Bank select (selects a 64K cell bank of the VRAM)    |
|        |                                                                   |
|        | The partition select bits are OR combined with the whole part of  |
|        | the source offset after it is masked with the partition size.     |
|        |                                                                   |
|        | Note that bit 1 is also OR combined if the partition size is 2    |
|        | VRAM cells.                                                       |
+--------+-------------------------------------------------------------------+
|        | Destination bank & partition select.                              |
| 0x007  |                                                                   |
|        | - bit  2-15: Partition select bits                                |
|        | - bit  0- 1: Bank select (selects a 64K cell bank of the VRAM)    |
|        |                                                                   |
|        | The partition select bits are OR combined with the whole part of  |
|        | the destination offset after it is masked with the partition      |
|        | size.                                                             |
|        |                                                                   |
|        | Note that bit 1 is also OR combined if the partition size is 2    |
|        | VRAM cells.                                                       |
+--------+-------------------------------------------------------------------+
|        | Partitioning settings.                                            |
| 0x008  |                                                                   |
|        | - bit 12-15: Source partition size                                |
|        | - bit  8-11: X/Y split location (X size)                          |
|        | - bit  4- 7: Destination partition size                           |
|        | - bit  0- 3: Unused                                               |
|        |                                                                   |
|        | The Source & Destination partition sizes and the X/Y split        |
|        | location may specify the following sizes:                         |
|        |                                                                   |
|        | - 0:  4 Words (2 * 32 bit cells)                                  |
|        | - 1:  8 Words (4 * 32 bit cells)                                  |
|        | - 2:  16 Words (8 * 32 bit cells)                                 |
|        | - 3:  32 Words (16 * 32 bit cells)                                |
|        | - 4:  64 Words (32 * 32 bit cells)                                |
|        | - 5:  128 Words (64 * 32 bit cells)                               |
|        | - 6:  256 Words (128 * 32 bit cells)                              |
|        | - 7:  512 Words (256 * 32 bit cells)                              |
|        | - 8:  1 KWords (512 * 32 bit cells)                               |
|        | - 9:  2 KWords (1K * 32 bit cells)                                |
|        | - 10: 4 KWords (2K * 32 bit cells)                                |
|        | - 11: 8 KWords (4K * 32 bit cells)                                |
|        | - 12: 16 KWords (8K * 32 bit cells)                               |
|        | - 13: 32 KWords (16K * 32 bit cells)                              |
|        | - 14: 64 KWords (32K * 32 bit cells)                              |
|        | - 15: 128 KWords (64K * 32 bit cells)                             |
+--------+-------------------------------------------------------------------+
|        | Reindex bank select.                                              |
| 0x009  |                                                                   |
|        | - bit  5-15: Unused                                               |
|        | - bit  0- 4: Reindex bank select                                  |
+--------+-------------------------------------------------------------------+
|        | Substitution flags & Source barrel rotate.                        |
| 0x00A  |                                                                   |
|        | - bit    15: Load destination from Source X every row if set      |
|        | - bit    14: Load destination from Source Y every row if set      |
|        | - bit    13: Load count from Source Y every row if set            |
|        | - bit  3-12: Unused                                               |
|        | - bit  0- 2: Pixel barrel rotate right                            |
|        |                                                                   |
|        | In 4 bit mode only bits 0-1 are used of the Pixel barrel rotate   |
|        | right.                                                            |
|        |                                                                   |
|        | The Load count (bit 13) flag does not change the limitations of   |
|        | count. Bits 13 - 15 of Y fraction will be loaded into the low 3   |
|        | bits of count, and bits 0 - 6 of Y whole will be loaded into bits |
|        | 3 - 9 of count.                                                   |
|        |                                                                   |
|        | If both bit 15 and 14 is set, the destination is loaded from      |
|        | Source Y.                                                         |
|        |                                                                   |
|        | The partition is not affected by bits 15 or 14, it is always      |
|        | selected by the destination partition select register.            |
+--------+-------------------------------------------------------------------+
|        | Source masks.                                                     |
| 0x00B  |                                                                   |
|        | - bit  8-15: Pixel OR mask                                        |
|        | - bit  0- 7: Pixel AND mask                                       |
|        |                                                                   |
|        | The OR mask is stronger than the AND mask. In 4 bit mode, only    |
|        | the low 4 bits of each are used.                                  |
+--------+-------------------------------------------------------------------+
|        | Colorkey and Control flags.                                       |
| 0x00C  |                                                                   |
|        | - bit 14-15: Unused                                               |
|        | - bit    13: (VDR) If bit 12 is set, Reindex using dest. if set   |
|        | - bit    12: (VRE) Reindexing enabled if set                      |
|        | - bit 10-11: (VMD) Selects blit mode                              |
|        | - bit     9: (VCK) Colorkey enabled if set                        |
|        | - bit     8: (VMR) Pixel order swap enabled if set (Mirroring)    |
|        | - bit  0- 7: Colorkey (only low 4 bits used in 4 bit mode)        |
|        |                                                                   |
|        | The blit modes:                                                   |
|        |                                                                   |
|        | - 0: Block Blitter (BB)                                           |
|        | - 1: Filler (FL)                                                  |
|        | - 2: Scaled Blitter (SC)                                          |
|        | - 3: Line (LI)                                                    |
+--------+-------------------------------------------------------------------+
| 0x00D  | Count of rows to blit. Only bits 0 - 8 are used. If all these     |
|        | bits are set zero, 512 rows are produced.                         |
+--------+-------------------------------------------------------------------+
|        | Count of 4 bit pixels to blit. Only bits 0 - 9 are used in 4 bit  |
| 0x00E  | mode and only bits 1 - 9 are used in 8 bit mode. Setting all the  |
|        | used bits zero results in 1024 (4 bit) or 512 (8 bit) pixels.     |
+--------+-------------------------------------------------------------------+
|        | Start on write, Pattern for Filler (FL) & Line (LI). A write to   |
| 0x00F  | this location starts the accelerator operation.                   |
|        |                                                                   |
|        | The pattern is rotated by 1 pixel to the right after every row,   |
|        | useful for producing dithered fills.                              |
+--------+-------------------------------------------------------------------+
| 0x010  | Source Y whole part                                               |
+--------+-------------------------------------------------------------------+
| 0x011  | Source Y fractional part                                          |
+--------+-------------------------------------------------------------------+
| 0x012  | Source Y increment whole part                                     |
+--------+-------------------------------------------------------------------+
| 0x013  | Source Y increment fractional part                                |
+--------+-------------------------------------------------------------------+
| 0x014  | Source Y post-add whole part                                      |
+--------+-------------------------------------------------------------------+
| 0x015  | Source Y post-add fractional part                                 |
+--------+-------------------------------------------------------------------+
| 0x016  | Source X whole part                                               |
+--------+-------------------------------------------------------------------+
| 0x017  | Source X fractional part                                          |
+--------+-------------------------------------------------------------------+
| 0x018  | Source X increment whole part                                     |
+--------+-------------------------------------------------------------------+
| 0x019  | Source X increment fractional part                                |
+--------+-------------------------------------------------------------------+
| 0x01A  | Source X post-add whole part                                      |
+--------+-------------------------------------------------------------------+
| 0x01B  | Source X post-add fractional part                                 |
+--------+-------------------------------------------------------------------+
| 0x01C  | Destination whole part                                            |
+--------+-------------------------------------------------------------------+
| 0x01D  | Destination fractional part (only high 3 bits are used)           |
+--------+-------------------------------------------------------------------+
| 0x01E  | Destination increment whole part                                  |
+--------+-------------------------------------------------------------------+
| 0x01F  | Destination post-add whole part                                   |
+--------+-------------------------------------------------------------------+

The Destination increment normally should be set to one (1). Otherwise the
Accelerator still performs the same way (also in calculating masks for begin,
middle, and end cells), just the result goes in a different layout. Setting
this to something else than one may be useful for example when blitting small
tiles to cell boundaries (such as emulating a character mode), so a blit can
be performed with less operations.

Note that no interface register changes it's value during the course of an
accelerator operation, so retriggering the accelerator performs the exact same
blit.

The Accelerator also has a Reindex table in the 0x100 - 0x1FF range. This
reindex table contains 8 bit values, two in each register. The layout of this
area is as follows:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | First reindex table entry, first reindex bank (bank 0).           |
| 0x100  |                                                                   |
|        | - bit  8-15: Reindex for source value 0x0.                        |
|        | - bit  0- 7: Reindex for source value 0x1.                        |
+--------+-------------------------------------------------------------------+
| 0x101  | Reindexes for source values 0x2 and 0x3, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x102  | Reindexes for source values 0x4 and 0x5, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x103  | Reindexes for source values 0x6 and 0x7, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x104  | Reindexes for source values 0x8 and 0x9, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x105  | Reindexes for source values 0xA and 0xB, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x106  | Reindexes for source values 0xC and 0xD, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x107  | Reindexes for source values 0xE and 0xF, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x108  | Further reindex banks (banks 1 - 31) to specify 512 reindex       |
| \-     | values in total.                                                  |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+

Note that the value order accords with the Big Endian scheme the system uses.

In 4 bit mode the high 4 bits of each reidex value are left unused.
