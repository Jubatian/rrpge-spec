
Graphics Accelerator architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the Graphics Accelerator component of
the RRPGE system.

The Graphics Accelerator provides for hardware accelerated graphics operations
such as sprite blitting, scaling, and filling.

It operates on the Peripheral bus accessing the Peripheral RAM (PRAM), having
the lowest priority for accessing it (though higher than it's FIFO). By this
it is capable to work in parallel with the RRPGE CPU.


Design notes:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Accelerator originally was capable to perform 8 bit blits (256 color mode
surfaces), however current RRPGE specifications don't require this capability.
The structure is kept so it is possible to expand it to 8 bit mode if
necessary (the last version containing the 8 bit capability was 00.017.000).




Accelerator overview
------------------------------------------------------------------------------


The Accelerator component is basically a very flexible blitter capable of
operating in four distinct major modes:

- (BB) Block blitter, combining a source area onto a destination.
- (FL) Filler, combining a source pattern onto a destination.
- (SC) Scaled blitter, combining a source area onto a destination.
- \(LI) Line, capable to draw lines by start point, direction and length.

In all modes the following transformations may be applied (in "FL" and "LI"
modes the source being the pattern to output):

- Source barrel rotation per pixel.
- Source AND mask per pixel.
- Colorkeying (a designated transparent color index).
- Source OR mask per pixel.
- Reindexing using a reindex table (color remapping).
- Reindexing by destination (color remapping).
- Destination write mask (selecting bit positions to be written).

Some more advanced examples of usages of the Accelerator are as follows:

- Using the AND mask per pixel and optionally the barrel rotation per pixel
  feature a given area of the Peripheral RAM may be utilized to hold two
  overlaid low bit depth images used separately. This allows for better
  utilization of memory.

- Using the AND and OR masks with an appropriate palette may be used to
  generate player-specific colored variants of objects without needing slower
  reindexing.

- The Filler may be used for polygon filling: the destination and count
  post-adds can be used to shape a half-triangle, the pattern rotation can
  produce appropriate dithered output.

- The Scaled blitter may be utilized for sprite scaling, rotation, and simple
  texture blitting for three dimensional scenes.

- Reindexing by destination with an appropriate color remapping table may be
  utilized for alpha blending without needing to split up the display into
  separate bit planes.




Accelerator stages
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
  order of the source data.

- (VCK) Colorkey stage which can be applied to all modes. This provides a
  transparent pixel value where the background shows through.

- (VRE) Reindex stage which may be applied to all modes. This remaps pixel
  values according to a table without affecting the colorkey stage.

- (VDR) Reindex using destination. This extends the reindex stage by involving
  the destination data to select from a larger table.

- (VBT) Display mode to work for: either 4 bit mode or 8 bit mode.

The Accelerator has two major stages as follows:

- Source fetch. This stage is performed according to the the selected mode
  (VMD), giving four possible distinct paths.

- Destination combine. This stage varies according to whether reindexing is
  necessary (VRE), giving two possible distinct paths.




Source fetch major stage
------------------------------------------------------------------------------


The source fetch stage prepares the source data performing any transforms
possible on it without the knowledge of the destination. For each Peripheral
RAM cell necessarily affected it prepares a PRAM cell aligned data and a cell
begin / middle / end mask.

The mode selector (VMD) defines the path to take executing this stage. VMR and
VBT may have effect on the execution otherwise.


The begin / middle / end mask
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The begin / middle / end mask is prepared according to the destination start
pointer fraction and the count of units (cells, or pixels in the case of Line
mode) to process, bits from the latter used according to the display mode. In
Line (LI) mode this mask always selects a single pixel on the destination
cell.

Note that for short blits (in FL or SC modes) the begin and end of the blit
may occur in the same cell. This situation also has to be supported proper,
producing an appropriate merged mask.


Offset calculation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Source offset calculation if necessary is done by Pointer X and Pointer Y, as
described for each mode.

Destination offset calculation is done using the Destination pointer in
Block Blitter (BB), Filler (FL) and Scaled Blitter (SC) modes. In Line (LI)
mode, Pointer X and Pointer Y is used to calculate destination.

The destination pointer has no increment register: It always increments one
cell after processing a cell of data.

Neither source or destination may cross bank (64K cells) boundary during these
calculations, they wrap around (as there are only 16 bits whole part for
either of these pointers). Moreover, the partition settings affect how much of
the higher bits of the pointers are discarded for the generation of offset,
using the contents of the appropriate partition register instead (note that
the partition register is OR combined, so may affect lower bits).


Block Blitter (BB)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Block Blitter produces a horizontal strip of sequentially read data
beginning at an arbitrary position for each row. Source data is fetched by
Pointer X. The source data may only begin at cell boundary (the fractional
part of Pointer X is ignored), and the blit's row length is specified in cell
units (the fractional part of Count is ignored).

Pointer X increment is not used. The increment is one cell if VMR is clear,
0xFFFF cells (or one cell decrement) if VMR is set.

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
    |  Pixel order swap | If VMR is enabled (Mirroring)
    +-------------------+
              |
              +------------+ Shift to align with destination
                           V
    +----+----+----+----+----+----+----+----+
    | Prev. src. |   Current source  |      | Shift register
    +----+----+----+----+----+----+----+----+
              |
              V
    +----+----+----+----+
    |    Data to blit   | Aligned with the destination cells
    +----+----+----+----+


Filler (FL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Filler produces a horizontal line of an arbitrary length of an uniform
source pattern for each row. The destination post-add, and count post-add
registers are used (both the whole and fractional parts), making half-triangle
blitting possible (for polygon blits).

The source data is fetched from the pattern (provided in the Start trigger
register). The pattern is rotated left 4 bits (1 pixel) on row transitions,
providing support for dithering fills.

Note that the pattern is not rotated in any manner to align it with the
destination fraction: it always aligns with half-cell boundary.

The Pointer X and Pointer Y registers are not used.

The source data is prepared as follows: ::


    +----+----+
    | Pattern | 16 bit line pattern, expanded to 32 bit cells
    +----+----+
         |
         +---------+
         V         V
    +----+----+----+----+
    |    Data to blit   | Aligned with the destination cells
    +----+----+----+----+


Scaled blitter (SC)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Scaled Blitter produces a horizontal strip of data beginning at an
arbitrary position from evenly spaced out source pixels of arbitrary length in
pixels, for each row.

Source offset generation for each pixel operates as follows: ::


    |<--- Source ---->|<- Source partition size ->|
    |                 |                           |
    |                 |           |<- X/Y split ->|
    |                 |           |               |
    |    +------------+-----------+---------------+--------------------------+
    |    | P.sel bits |  Y bits   |    X bits     |          X bits          |
    +----+------------+-----------+---------------+--------------------------+
    |Bank|          Whole part (16 bits)          | Fractional part (16 bits)|
    +----+----------------------------------------+--------------------------+


The source partition size has higher priority (only it affects the number of
partition select bits, even if X/Y split is larger).

Note that the Partition select bits are OR combined on the whole part, so the
contents of the Source partition select register may have effect within the
Source partition size.

The destination post-add, and count post-add registers are also used (both the
whole and fractional parts), making half-triangle blitting possible (for
polygon blits, or producing segments of an arbitrarily rotated sprite).

The source data is prepared as follows: ::


    +----+ +----+     +----+ +----+
    | Px | | Px | ... | Px | | Px | Up to 8 pixels
    +----+ +----+     +----+ +----+
      |      |          |      |
      | +----+          |      |
      | |           +---+      |
      | |           | +--------+
      | |           | |
    +----+----+----+----+
    |    Data to blit   | Aligned with the destination cells
    +----+----+----+----+


Line (LI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Line mode outputs a line specified by Pointer X and Pointer Y, and the
Count register as pixel count for the line. The Row count is not used, neither
any of the post-add registers (which are only valid for row transitions).

Pixels to output are selected by Pointer X and Pointer Y, addressing in the
destination. The destination bank and partition settings are used to produce
the high part of this offset, otherwise it is generated in an identical manner
to the Scaled Blitter's offset generation mechanism: ::


    |<- Destination ->|<- Dest. partition size -->|
    |                 |                           |
    |                 |           |<- X/Y split ->|
    |                 |           |               |
    |    +------------+-----------+---------------+--------------------------+
    |    | P.sel bits |  Y bits   |    X bits     |          X bits          |
    +----+------------+-----------+---------------+--------------------------+
    |Bank|          Whole part (16 bits)          | Fractional part (16 bits)|
    +----+----------------------------------------+--------------------------+


Note the OR combining of the Partition select on the whole part.

For each selected pixel, a Begin / Middle / End mask is generated selecting
that single pixel, and the Destination Combine stage is started with this
input.

The line pattern is used to produce the line's color. The pattern is rotated
right one pixel (4 bits) after every two pixels output, always using the
lowest pixel (4 bits) for the output.




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


    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+
              |
              V
    +-------------------+
    | Px. barrel rotate | Barrel rotates each pixel by the given count
    +-------------------+
              |
              V
    +-------------------+
    |   Pixel AND mask  | Applies the Pixel AND mask on each pixel
    +-------------------+
              |
              V
    +----+----+----+----+  If VCK   +----+----+----+----+
    |  Transformed data |---------->|   Colorkey mask   |
    +----+----+----+----+           +----+----+----+----+
              |                               |
              |         +----+----+----+----+ | +----+----+----+----+
              |         |  PRAM Write mask  | | |  Beg/Mid/End mask |
              |         +----+----+----+----+ | +----+----+----+----+
              V                   |          _V_          |
    +-------------------+         +-------->|AND|<--------+
    |   Pixel OR mask   |                    ~|~
    +-------------------+                     |
              |                               |
             _V_                              |
            |AND|<----------------------------+
             ~|~                              |
             _V_       ___                   _V_
            | OR|<----|AND|<----------------|NEG|
             ~|~       ~A~                   ~~~
              V         |
      ---+----+----+----+----+---
         | Target PRAM cell  |
      ---+----+----+----+----+---


Reindexing blit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This path is used if VRE is enabled (reindex mode). This case if VDR is also
enabled, the path feeding in the target PRAM cell's data is also effective and
is used for providing the high bits (up to 5) for selecting a new pixel value
from the reindex table. ::


    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+
              |
              V
    +-------------------+
    | Px. barrel rotate | Barrel rotates each pixel by the given count
    +-------------------+
              |
              V
    +-------------------+
    |   Pixel AND mask  | Applies the Pixel AND mask on each pixel
    +-------------------+
              |
              V
    +----+----+----+----+  If VCK   +----+----+----+----+
    |  Transformed data |---------->|   Colorkey mask   |
    +----+----+----+----+           +----+----+----+----+
              |                               |
              |         +----+----+----+----+ | +----+----+----+----+
              |         |  PRAM Write mask  | | |  Beg/Mid/End mask |
              |         +----+----+----+----+ | +----+----+----+----+
              V                   |          _V_          |
    +-------------------+         +-------->|AND|<--------+
    |   Pixel OR mask   |                    ~|~
    +-------------------+                     |
              |                               |
              V                               |
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
         | Target PRAM cell  |
      ---+----+----+----+----+---


Accelerated combine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For every destination combine, the combined mask is checked. If the mask
is all set (all bits are to be taken from the source), and VDR (reindex by
destination) is unset, the destination data read is omitted, saving 2
cycles if possible (reindexing might stall the pipeline negating this).




Finalizing the row
------------------------------------------------------------------------------


When the row is complete, if the selected blit mode uses those, the original
values (as they were before starting the row) of Pointer X and Pointer Y are
incremented by the contents of the appropriate post-add registers, and are
used to start the next row.

Note that intermediate increments performed during the output of the row are
discarded.

The Destination pointer and the Count are also incremented by the respective
post-add in modes where these are appropriate.




Minor stages explained
------------------------------------------------------------------------------


This chapter explains some of the minor stages of the accelerator.


Pixel order swap (Mirror: VMR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This stage swaps the pixel order: ::


    +--+--+--+--+--+--+--+--+
    |P0|P1|P2|P3|P4|P5|P6|P7|
    +--+--+--+--+--+--+--+--+
                |
                |    Pixel order swap (Mirror)
                V
    +--+--+--+--+--+--+--+--+
    |P7|P6|P5|P4|P3|P2|P1|P0|
    +--+--+--+--+--+--+--+--+


Note that the other source transforms (Read AND & OR mask and barrel rotate)
also behave in a similar manner, on pixel level.


Reindex (VRE and VDR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Re-indexes each pixel using a table within the Accelerator component. It
operates as follows on pixel level: ::


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
                                | New px |     Reindex table (512 x 4bit)
                            ----+--------+----
                                    |
       +----------------------------+
       V
    +------+
    |  Px  | New pixel value stored
    +------+


If VDR is also enabled, instead of the "Reindex bank select" peripheral
register the destination's appropriate pixel is used after applying write
masks. The highest bit of the table address is always zero if VDR is enabled.


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

- The exact location and order of accesses during the operation. Emulators are
  allowed to perform the entire accelerator operation in one pass, without
  considering other peripherals' operation (such as the Graphics Display
  Generator) on the peripheral bus.

Note that the timing once it meets the minimal requirements is also
implementation defined.




Accelerator operation timing
------------------------------------------------------------------------------


The accelerator is designed to perform one 32 bit memory access on the
Peripheral RAM every second cycle (interleaved with the Graphics Display
Generator's accesses) at it's peak rate. Most of the modes are pipelined to
perform by this rule except when delayed by reindexing.

Reindexing can be performed at one pixel per cycle irrespective of whether the
destination has to be accessed for it (VDR enabled) or not.

Following the performance (in main clock cycles) for each of the eight major
stage combinations are provided. 'n' is the PRAM cell count which has to be
written during the operation, 'p' is the count of pixels to render, 'r' is the
number of rows to render. In the Accel. combine column only the 'n' member is
shown where appropriate. Note that in Line mode the row count is unused, so
there are no 'r' cycles.

+------+-----+-------------------------------------------+-------------------+
| Mode | VRE | Cycles                                    | Accel. combine    |
+======+=====+===========================================+===================+
|  BB  | NO  | 20 + (r * 2) + (n * 6)                    | n * 4             |
+------+-----+-------------------------------------------+-------------------+
|  FL  | NO  | 20 + (r * 4) + (n * 4)                    | n * 2             |
+------+-----+-------------------------------------------+-------------------+
|  SC  | NO  | 20 + (r * 8) + (n * 4) + (p * 2)          | n * 2             |
+------+-----+-------------------------------------------+-------------------+
|  LI  | NO  | 20                     + (p * 4)          | \-                |
+------+-----+-------------------------------------------+-------------------+
|  BB  | YES | 28 + (r * 2) + (n * 8) (*)                | n * 8 (*)         |
+------+-----+-------------------------------------------+-------------------+
|  FL  | YES | 28 + (r * 4) + (n * 8) (*)                | n * 8 (*)         |
+------+-----+-------------------------------------------+-------------------+
|  SC  | YES | 28 + (r * 8) + (n * 4) + (p * 2)          | n * 2             |
+------+-----+-------------------------------------------+-------------------+
|  LI  | YES | 28                     + (p * 4)          | \-                |
+------+-----+-------------------------------------------+-------------------+

Note that 8 reindexing accesses are necessary for processing each PRAM cell.
Modes where this determines the performance are marked with a '*'.

Note that the Accelerated combine may be in effect for any processed cell if
it's conditions are met. In Line mode the conditions of it can never be met.




Accelerator memory map
------------------------------------------------------------------------------


The following table describes the registers of the Accelerator. These
registers are only accessible through the Graphics FIFO (see "fifo.rst" for
details).

The Accelerator components are accessed by a 9 bit address of which the first
half represents the Accelerator registers repeating every 32 words in this
range, and the second half represents the Reindex table.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 | Peripheral RAM write mask (0x0000: High, 0x0001: Low). Clear bits |
| \-     | in it mask writes to the respective positions in the Destination  |
| 0x0001 | combine stage of the Accelerator.                                 |
+--------+-------------------------------------------------------------------+
|        | Destination bank select & Partition size.                         |
| 0x0002 |                                                                   |
|        | - bit 12-15: Destination partition size                           |
|        | - bit  4-11: Unused                                               |
|        | - bit  0- 3: Bank select (selects a 64K cell bank of the PRAM)    |
|        |                                                                   |
|        | For the interpretation of Destination partition size, see 0x0014. |
+--------+-------------------------------------------------------------------+
|        | Destination partition select. OR combined with the whole part of  |
| 0x0003 | the destination offset after that offset is masked with the       |
|        | partition size.                                                   |
+--------+-------------------------------------------------------------------+
| 0x0004 | Destination post-add whole part. Not used for LI.                 |
+--------+-------------------------------------------------------------------+
| 0x0005 | Destination post-add fractional part. Not used for BB and LI.     |
+--------+-------------------------------------------------------------------+
| 0x0006 | Count post-add whole part. Not used for BB and LI.                |
+--------+-------------------------------------------------------------------+
| 0x0007 | Count post-add fractional part. Not used for BB and LI.           |
+--------+-------------------------------------------------------------------+
| 0x0008 | Pointer Y post-add whole part. Only used for SC.                  |
+--------+-------------------------------------------------------------------+
| 0x0009 | Pointer Y post-add fractional part. Only used for SC.             |
+--------+-------------------------------------------------------------------+
| 0x000A | Pointer X post-add whole part. Only used for BB and SC.           |
+--------+-------------------------------------------------------------------+
| 0x000B | Pointer X post-add fractional part. Only used for SC.             |
+--------+-------------------------------------------------------------------+
| 0x000C | Pointer Y increment whole part. Only used for SC and LI.          |
+--------+-------------------------------------------------------------------+
| 0x000D | Pointer Y increment fractional part. Only used for SC and LI.     |
+--------+-------------------------------------------------------------------+
| 0x000E | Pointer X increment whole part. Only used for SC and LI.          |
+--------+-------------------------------------------------------------------+
| 0x000F | Pointer X increment fractional part. Only used for SC and LI.     |
+--------+-------------------------------------------------------------------+
| 0x0010 | Pointer Y whole part. Only used for SC and LI.                    |
+--------+-------------------------------------------------------------------+
| 0x0011 | Pointer Y fractional part. Only used for SC and LI.               |
+--------+-------------------------------------------------------------------+
|        | Source bank select.                                               |
| 0x0012 |                                                                   |
|        | - bit  4-15: Unused                                               |
|        | - bit  0- 3: Bank select (selects a 64K cell bank of the PRAM)    |
|        |                                                                   |
|        | Not used for FL and LI.                                           |
+--------+-------------------------------------------------------------------+
|        | Source partition select. OR combined with the whole part of the   |
| 0x0013 | the source offset after that offset is masked with the partition  |
|        | size.                                                             |
|        |                                                                   |
|        | Not used for FL and LI.                                           |
+--------+-------------------------------------------------------------------+
|        | Source partitioning settings.                                     |
| 0x0014 |                                                                   |
|        | - bit 12-15: Source partition size. Only for BB and SC.           |
|        | - bit  8-11: X/Y split location (X size). Only for SC and LI.     |
|        | - bit  0- 7: Unused                                               |
|        |                                                                   |
|        | The Source & Destination partition sizes (the latter in 0x0002)   |
|        | and the X/Y split location may specify the following sizes:       |
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
|        | Blit control flags & Source barrel rotate.                        |
| 0x0015 |                                                                   |
|        | - bit  7-15: Unused                                               |
|        | - bit  5- 6: (VMD) Selects blit mode                              |
|        | - bit     4: Unused                                               |
|        | - bit     3: (VCK) Colorkey enabled if set                        |
|        | - bit     2: Unused                                               |
|        | - bit  0- 1: Pixel barrel rotate right                            |
|        |                                                                   |
|        | The blit modes:                                                   |
|        |                                                                   |
|        | - 0: Block Blitter (BB)                                           |
|        | - 1: Filler (FL)                                                  |
|        | - 2: Scaled Blitter (SC)                                          |
|        | - 3: Line (LI)                                                    |
+--------+-------------------------------------------------------------------+
|        | Pixel AND mask & Colorkey.                                        |
| 0x0016 |                                                                   |
|        | - bit 12-15: Unused                                               |
|        | - bit  8-11: Pixel AND mask                                       |
|        | - bit  4- 7: Unused                                               |
|        | - bit  0- 3: Colorkey                                             |
+--------+-------------------------------------------------------------------+
| 0x0017 | Count of rows to blit. Only bits 0 - 8 are used. If all these     |
|        | bits are set zero, 512 rows are produced. Not used for LI.        |
+--------+-------------------------------------------------------------------+
|        | Count of cells / pixels to blit, whole part.                      |
| 0x0018 |                                                                   |
|        | Only bits 0 - 7 are used for producing 0 - 255 cells of output in |
|        | BB, FL and SC modes. In LI mode all bits are used, defining the   |
|        | count of pixels to produce.                                       |
+--------+-------------------------------------------------------------------+
|        | Count of cells / pixels to blit, fractional part.                 |
| 0x0019 |                                                                   |
|        | Used in FL and SC modes for a pixel precise row length. Only the  |
|        | high 3 bits are used for generating the row. Not used for BB and  |
|        | LI.                                                               |
+--------+-------------------------------------------------------------------+
| 0x001A | Source X whole part. Not used for FL.                             |
+--------+-------------------------------------------------------------------+
| 0x001B | Source X fractional part. Not used for BB and FL.                 |
+--------+-------------------------------------------------------------------+
| 0x001C | Destination whole part. Not used for LI.                          |
+--------+-------------------------------------------------------------------+
| 0x001D | Destination fractional part. Not used for LI.                     |
+--------+-------------------------------------------------------------------+
|        | Reindexing & Pixel OR mask.                                       |
| 0x001E |                                                                   |
|        | - bit    15: (VMR) Pixel order swap enabled if set (Mirroring)    |
|        | - bit    14: (VDR) If bit 13 is set, Reindex using dest. if set   |
|        | - bit    13: (VRE) Reindexing enabled if set                      |
|        | - bit  8-12: Reindex bank select                                  |
|        | - bit  4- 7: Unused                                               |
|        | - bit  0- 3: Pixel OR mask                                        |
|        |                                                                   |
|        | The VMR flag only has effect in BB mode.                          |
+--------+-------------------------------------------------------------------+
| 0x001F | Start on write & Pattern for Filler (FL) & Line (LI). A write to  |
|        | this location starts the accelerator operation.                   |
+--------+-------------------------------------------------------------------+

Note that no interface register changes it's value during the course of an
accelerator operation, so retriggering the accelerator performs the exact same
blit.

Register usage table, summarizing which of the registers each blit mode uses:

+--------+-----------------------------------------------+----+----+----+----+
| Range  | Description                                   | BB | FL | SC | LI |
+========+===============================================+====+====+====+====+
| 0x0000 | Peripheral RAM write mask, high               |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0001 | Peripheral RAM write mask, low                |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0002 | Destination bank select & Partition size      |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0003 | Destination partition select                  |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0004 | Destination post-add whole part               |  X |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0005 | Destination post-add fractional part          |    |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0006 | Count post-add whole part                     |    |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0007 | Count post-add fractional part                |    |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0008 | Pointer Y post-add whole part                 |    |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0009 | Pointer Y post-add fractional part            |    |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000A | Pointer X post-add whole part                 |  X |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000B | Pointer X post-add fractional part            |    |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000C | Pointer Y increment whole part                |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000D | Pointer Y increment fractional part           |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000E | Pointer X increment whole part                |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x000F | Pointer X increment fractional part           |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0010 | Pointer Y whole part                          |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0011 | Pointer Y fractional part                     |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0012 | Source bank select                            |  X |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0013 | Source partition select                       |  X |    |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0014 | Source partitioning settings                  |  X |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0015 | Blit control flags & Source barrel rotate     |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0016 | Source AND mask & Colorkey                    |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0017 | Count of rows to blit                         |  X |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0018 | Count of cells / pixels to blit, whole part   |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x0019 | Count of cells / pixels to blit, fract. part  |    |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001A | Source X whole part                           |  X |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001B | Source X fractional part                      |    |    |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001C | Destination whole part                        |  X |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001D | Destination fractional part                   |  X |  X |  X |    |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001E | Reindexing & Pixel OR mask                    |  X |  X |  X |  X |
+--------+-----------------------------------------------+----+----+----+----+
| 0x001F | Start on write & Pattern                      |    |  X |    |  X |
+--------+-----------------------------------------------+----+----+----+----+

The Start on write (0x001F) register is necessarily written for all blit modes
to start the operation, however the Pattern written into it is only used for
Filler and Line modes.

The Reindex table:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | First reindex table entry, first reindex bank (bank 0).           |
| 0x0100 |                                                                   |
|        | - bit 12-15: Unused.                                              |
|        | - bit  8-11: Reindex for source value 0x0.                        |
|        | - bit  4- 7: Unused.                                              |
|        | - bit  0- 3: Reindex for source value 0x1.                        |
+--------+-------------------------------------------------------------------+
| 0x0101 | Reindexes for source values 0x2 and 0x3, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0102 | Reindexes for source values 0x4 and 0x5, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0103 | Reindexes for source values 0x6 and 0x7, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0104 | Reindexes for source values 0x8 and 0x9, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0105 | Reindexes for source values 0xA and 0xB, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0106 | Reindexes for source values 0xC and 0xD, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0107 | Reindexes for source values 0xE and 0xF, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0x0108 | Further reindex banks (banks 1 - 31) to specify 512 reindex       |
| \-     | values in total.                                                  |
| 0x01FF |                                                                   |
+--------+-------------------------------------------------------------------+

Note that the value order accords with the Big Endian scheme the system uses.
