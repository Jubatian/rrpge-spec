
RRPGE Graphics Display & Accelerator, Accelerator component architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the accelerator component from the
Graphics Display & Accelerator unit.

The accelerator component provides for hardware accelerated graphics
operations such as sprite blitting, scaling, and filling.

The Graphics Display and Accelerator unit also has some specifics regarding
accessing the Video RAM, these are also described here.

The Accelerator component is capable to operate in parallel with the CPU (and
the audio mixer). It however stalls the CPU until completion if it attempts to
access the Video RAM or the Video peripheral page while an operation is in
progress (see "Addressing stalls" in "cpu_arch.rst").




Video memory accessing
------------------------------------------------------------------------------


The Video memory internally operates over a 32 bit data bus: the Display and
the Accelerator components access it using this while for the CPU accesses are
transformed for it's 16 bit data bus.

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
    | CPUdata | (16 bits)
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
in a Video memory page or the Video peripheral area for writing is only
allowed if the respective page is also banked in for read on the same
location.

These properties allow for simplifying the latching logic greatly in emulators
as from the CPU's point of view normal 16 bit accesses happen only affected by
the Video RAM Write mask.




Accelerator overview
------------------------------------------------------------------------------


The Accelerator component of the Graphics Display & Accelerator unit is
basically a very flexible blitter capable of operating in three distinct major
modes:

- (BB) Block blitter, combining a source area onto a destination.
- (SC) Scaled blitter, combining a source area onto a destination.
- \(LI) Line filler, combining a source pattern onto a destination.

On the source in "BB" and "SC" modes the following transformations may be
applied:

- AND mask per pixel.
- Barrel rotation per pixel.
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

- The same way if the Display unit is also set up to display a lower bit depth
  layer from a Video memory area, this area may hold overlaid low bit depth
  images not affecting the display.

- The Scaled blitter combined with appropriate partitioning setting may be
  utilized for simple texture blitting for three dimensional scenes.

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

- (VMD) The mode selector which selects from the three main accelerator modes,
  the Block Blitter (BB), the Scaled Blitter (SC) and the Line Filler (LI).

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
mode (lowest bit ignored in 8 bit display mode).

Note that for short blits the begin and end of the blit may occur in the same
cell. This situation also has to be supported proper.

The mode selector (VMD) defines the path to take executing this stage. Only
VMR may have effect on the execution otherwise.


Line Filler (LI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Line Filler normally produces a horizontal line of an arbitrary length (in
pixels) of an uniform source pattern.

The source data is prepared as follows, probably in advance: ::


    +----+----+
    | Pattern | 16 bit line pattern
    +----+----+
         |
         +---------+
         V         V
    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+


Block Blitter (BB)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Block Blitter normally produces a horizontal strip of sequentially read
data beginning at an arbitrary position. The source data can only begin at
Video RAM cell boundary, but can be of arbitrary length in pixels.

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
    |   Read AND mask   | Applies the Read AND mask on each pixel
    +-------------------+
              |
              V
    +-------------------+
    | Px. barrel rotate | Barrel rotates each pixel by the given count
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
    | Prev. src. |   Current source  |      |
    +----+----+----+----+----+----+----+----+
              |
              V
    +----+----+----+----+
    |    Data to blit   |
    +----+----+----+----+


The Block Blitter uses a simple source increment logic, only taking a
dedicated Source pointer and a Source increment register for it. The first
source cell is taken from the offset by the initial value of the Source
Pointer, then, and after all the source fetches, the Source pointer is
incremented with the Source increment. Note that the increment happens even
for the last fetched cell even if it is only used partially.


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


The Scaled Blitter uses a complex source incrementing scheme supporting two
dimensional texture blitting. This scheme supersedes that of the BB mode: in
SC mode the source increments of the BB mode are inactive.

The scheme uses the following variables:

- (SPX) Source X pointer. It has whole (Video RAM cell) and fractional parts.
- (SIX) Source X increment. It has whole and fractional parts.
- (SPY) Source Y pointer. It has whole (Video RAM cell) and fractional parts.
- (SIY) Source Y increment. It has whole and fractional parts.
- (SSP) Source Y/X split mask.

SPX and SPY increment by SIX and SIY respectively after each pixel fetched.

The SSP split mask specifies (with bits set) which bits of the whole part
should be taken from SPX when preparing the pixel address to fetch. The rest
of the bits are taken from SPY. The fractional part is fully taken from SPX.

The source partition setting applies to SPX and SPY separately. There is no
hardware connection between the source partition setting and SSP.




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


    +----+----+----+----+  If VCK   +---------+
    |    Data to blit   |---------->| Col.key |
    +----+----+----+----+           +---------+
              |                          |
              |    +----+----+----+----+ | +----+----+----+----+
              |    |  VRAM Write mask  | | |  Beg/Mid/End mask |
              |    +----+----+----+----+ | +----+----+----+----+
              |              |          _V_          |
              |              +-------->|AND|<--------+
             _V_                        ~|~
            |AND|<-----------------------+
             ~|~                         |
             _V_       ___              _V_
            | OR|<----|AND|<-----------|NEG|
             ~|~       ~A~              ~~~
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


    +----+----+----+----+  If VCK   +---------+
    |    Data to blit   |---------->| Col.key |
    +----+----+----+----+           +---------+
              |                          |
              |    +----+----+----+----+ | +----+----+----+----+
              |    |  VRAM Write mask  | | |  Beg/Mid/End mask |
              |    +----+----+----+----+ | +----+----+----+----+
              |              |          _V_          |
              |              +-------->|AND|<--------+
              V                         ~|~
    +-----------------------------+      |
    |   Reindex (enabled by VRE)  |      |
    +-----------------------------+      |
              |     A                    |
              |     | If VDR             |
             _V_    |                    |
            |AND|<-)|(-------------------+
             ~|~    |                    |
             _V_    |  ___              _V_
            | OR|<--+-|AND|<-----------|NEG|
             ~|~       ~A~              ~~~
              |         |
              |         |
              V         |
      ---+----+----+----+----+---
         | Target VRAM cell  |
      ---+----+----+----+----+---




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


Note that the other source transforms (Read AND mask and barrel rotate) also
behave in a similar manner, on pixel level.


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
sequentially a source read from a cell would happen after a destination write.
This case due to the implementation defined length of the pipeline the source
read may fetch not yet changed data.

- If VMR is used with Scaled Blit and the count of pixels to blit is not a
multiple of 4 (8 bit mode) or 8 (4 bit mode), for the last source cell pixels
not filled may have an implementation defined content (typically either zero
or data left over from a previous operation). Note that this data does not
become visible unless VMR is set.

- The exact location and order of accesses during the operation. Note that
this is not particularly important as the loose timing requirements and the
implementation defined aspects of the display generator also prohibit relying
on this property.

Note that the timing once it meets the minimal requirements is also
implementation defined.




Accelerator operation timing
------------------------------------------------------------------------------


The accelerator is designed to perform one 32 bit memory access on the Video
RAM every second cycle at it's peak rate. Most of the modes are pipelined to
perform by this rule except when delayed by reindexing.

Reindexing can be performed at one pixel per cycle irrespective of whether the
destination has to be accessed for it (VDR enabled) or not. It is however not
affected by Video RAM stalls (since the reindex table is a separate memory
within the accelerator), so the Display unit has no effect on this process.

Display accesses happen according to "Addressing stalls" in "vid_arch.rst".

When none, one or two layers are enabled, the Display unit accesses the Video
RAM at most once in every two cycles, thus not affecting the performance of
the Accelerator.

If three layers are enabled since a free memory access comes only once in four
cycles, the performance of the Accelerator (if it performs at peak rate)
halves, and if all four layers are enabled, it stalls. Note that the Display
unit only produces this access scheme for 320 cycles out of the 400 provided
for a display line.

Following the normal performance (in main clock cycles) for each of the six
major stage combinations are provided. 'n' is the Video RAM cell count which
has to be written during the operation, 'p' is the count of pixels to render.

+------+------+-----+------------------------+-------------------------------+
| Disp | Mode | VRE | Cycles                 | Notes                         |
+======+======+=====+========================+===============================+
| 4bit |  LI  | NO  | 20 + (n * 4)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 4bit |  BB  | NO  | 20 + (n * 6)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 4bit |  SC  | NO  | 20 + (n * 4) + (p * 2) |                               |
+------+------+-----+------------------------+-------------------------------+
| 4bit |  LI  | YES | 28 + (n * 8)           | 3 layers: (n * 8)             |
+------+------+-----+------------------------+-------------------------------+
| 4bit |  BB  | YES | 28 + (n * 8)           | 3 layers: (n * 12)            |
+------+------+-----+------------------------+-------------------------------+
| 4bit |  SC  | YES | 28 + (n * 4) + (p * 2) |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  LI  | NO  | 20 + (n * 4)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  BB  | NO  | 20 + (n * 6)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  SC  | NO  | 20 + (n * 4) + (p * 2) |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  LI  | YES | 28 + (n * 4)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  BB  | YES | 28 + (n * 6)           |                               |
+------+------+-----+------------------------+-------------------------------+
| 8bit |  SC  | YES | 28 + (n * 4) + (p * 2) |                               |
+------+------+-----+------------------------+-------------------------------+

Normally in 3 layer enabled lines while the Display unit fetches data, the
performance of the accelerator halves. In 4bit mode the LI and BB modes with
enabled reindex (VRE) are an exception from this rule since there the
reindexing stalls the accelerator not allowing it to perform one access every
two main clock cycles.

Note that in 4 bit mode 8 reindexing accesses are necessary for processing
each Video RAM cell while in 8 bit mode 4 such accesses are necessary. This is
the source of difference in the timings.

The "extra" 20 or 28 cycles are not affected by the stalls from the Display
unit, however their distribution is implementation defined (depending on the
realization of the accelerator pipeline).




Video peripheral, Accelerator component related memory map
------------------------------------------------------------------------------


The following table describes those elements of the Video peripheral area
which are related to the Accelerator component. Note that these are as seen
from the CPU: the cells behind these offsets have a 16 bit width. Also note
that these repeat every 32 words in the 0xE00 - 0xEFF range, so for example
offsets 0xE02, 0xE22, 0xE42 ... 0xEE2 all refer to the Video RAM partition
size register.

Note that all bits within this area are writable, and their values are
preserved unless an accelerator operation overwrites them.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xEE0  | Video RAM write mask (0xEE0: High, 0xEE1: Low). Clear bits in it  |
|   \-   | disable write to the respective position both for the CPU and the |
| 0xEE1  | accelerator functions.                                            |
+--------+-------------------------------------------------------------------+
| 0xEE2  | Video RAM partition size. Defines further partitioning within the |
|        | Video RAM banks. Only the low 3 bits are used (the rest may be    |
|        | written but are ignored). Note that this setting also applies to  |
|        | the Display unit.                                                 |
|        |                                                                   |
|        | - 0: 1K Words (512 * 32 bit cells)                                |
|        | - 1: 2K Words (1K * 32 bit cells)                                 |
|        | - 2: 4K Words (2K * 32 bit cells)                                 |
|        | - 3: 8K Words (4K * 32 bit cells)                                 |
|        | - 4: 16K Words (8K * 32 bit cells)                                |
|        | - 5: 32K Words (16K * 32 bit cells)                               |
|        | - 6: 64K Words (32K * 32 bit cells)                               |
|        | - 7: 128K Words (64K * 32 bit cells)                              |
+--------+-------------------------------------------------------------------+
| 0xEE3  |                                                                   |
|   \-   | Display list offsets. See "vid_arch.rst" for details.             |
| 0xEE7  |                                                                   |
+--------+-------------------------------------------------------------------+
|        | Source X pointer whole part (32 bit cells). Used only for the     |
| 0xEE8  | Scaled Blitter (SC). Updates after source pointer increments in   |
|        | SC mode.                                                          |
+--------+-------------------------------------------------------------------+
| 0xEE9  | Source X pointer fractional part. Used only for the Scaled        |
|        | Blitter (SC). Updates after source pointer increments in SC mode. |
+--------+-------------------------------------------------------------------+
|        | Source Y pointer whole part (32 bit cells). Used only for the     |
| 0xEEA  | Scaled Blitter (SC). Updates after source pointer increments in   |
|        | SC mode.                                                          |
+--------+-------------------------------------------------------------------+
| 0xEEB  | Source Y pointer fractional part. Used only for the Scaled        |
|        | Blitter (SC). Updates after source pointer increments in SC mode. |
+--------+-------------------------------------------------------------------+
| 0xEEC  | Source X increment whole. Used only for the Scaled Blitter (SC).  |
+--------+-------------------------------------------------------------------+
| 0xEED  | Source X incr. fraction. Used only for the Scaled Blitter (SC).   |
+--------+-------------------------------------------------------------------+
| 0xEEE  | Source Y increment whole. Used only for the Scaled Blitter (SC).  |
+--------+-------------------------------------------------------------------+
| 0xEEF  | Source Y incr. fraction. Used only for the Scaled Blitter (SC).   |
+--------+-------------------------------------------------------------------+
| 0xEF0  | Source start pointer (32 bit cells). Used only for the Block      |
|        | Blitter (BB). Updates after source pointer increments in BB mode. |
+--------+-------------------------------------------------------------------+
| 0xEF1  | Source increment. Used only for the Block Blitter (BB).           |
+--------+-------------------------------------------------------------------+
| 0xEF2  | Destination start pointer whole part (32 bit cells). Updates      |
|        | after destination pointer increments.                             |
+--------+-------------------------------------------------------------------+
|        | Destination start pointer fractional part. Updated at the end of  |
| 0xEF3  | the operation to respect the length of data in pixels. Note that  |
|        | only the highest 2 or 3 bits are used depending on display mode   |
|        | (8 bit or 4 bit), but all it's bits are writable.                 |
+--------+-------------------------------------------------------------------+
| 0xEF4  | Destination increment whole part (32 bit cells)                   |
+--------+-------------------------------------------------------------------+
|        | (SSP) Source split mask. Used for the Scaled Blitter (SC).        |
| 0xEF5  | Specifies the bits to take from Source X pointer whole when       |
|        | composing the next pixel address to fetch.                        |
+--------+-------------------------------------------------------------------+
| 0xEF6  | Reindex bank select. Only low 5 bits are used.                    |
+--------+-------------------------------------------------------------------+
|        | Source read transform & partitioning.                             |
| 0xEF7  |                                                                   |
|        | - bit    15: If set, source partition size is 64K * 32 bits       |
|        | - bit 11-14: Source partitioning                                  |
|        | - bit  8-10: Pixel barrel rotate right                            |
|        | - bit  0- 7: Read AND mask                                        |
|        |                                                                   |
|        | In 4 bit mode only bits 0-3 are used of the Read AND mask, and    |
|        | only bits 8-9 are used of the Pixel barrel rotate right.          |
|        |                                                                   |
|        | Source partitioning is overridden by the partition size from      |
|        | 0xEE2 if that specifies a smaller size. The setting in bits 11-14 |
|        | is only effective if bit 15 is clear, and specifies the following |
|        | partition sizes:                                                  |
|        |                                                                   |
|        | - 0:  2 Words (1 * 32 bit cell)                                   |
|        | - 1:  4 Words (2 * 32 bit cells)                                  |
|        | - 2:  8 Words (4 * 32 bit cells)                                  |
|        | - 3:  16 Words (8 * 32 bit cells)                                 |
|        | - 4:  32 Words (16 * 32 bit cells)                                |
|        | - 5:  64 Words (32 * 32 bit cells)                                |
|        | - 6:  128 Words (64 * 32 bit cells)                               |
|        | - 7:  256 Words (128 * 32 bit cells)                              |
|        | - 8:  512 Words (256 * 32 bit cells)                              |
|        | - 9:  1 KWords (512 * 32 bit cells)                               |
|        | - 10: 2 KWords (1K * 32 bit cells)                                |
|        | - 11: 4 KWords (2K * 32 bit cells)                                |
|        | - 12: 8 KWords (4K * 32 bit cells)                                |
|        | - 13: 16 KWords (8K * 32 bit cells)                               |
|        | - 14: 32 KWords (16K * 32 bit cells)                              |
|        | - 15: 64 KWords (32K * 32 bit cells)                              |
+--------+-------------------------------------------------------------------+
|        | Colorkey and Control flags.                                       |
| 0xEF8  |                                                                   |
|        | - bit 14-15: Unused                                               |
|        | - bit    13: (VDR) If bit 12 is set, Reindex using dest. if set   |
|        | - bit    12: (VRE) Reindexing enabled if set                      |
|        | - bit    11: (VMD) If bit 10 is clear, Scaled blit (SC) if set    |
|        | - bit    10: (VMD) Line filler (LI) mode if set                   |
|        | - bit     9: (VCK) Colorkey enabled if set                        |
|        | - bit     8: (VMR) Pixel order swap enabled if set (Mirroring)    |
|        | - bit  0- 7: Colorkey (only low 4 bits used in 4 bit mode)        |
+--------+-------------------------------------------------------------------+
|        | Count of 4 bit pixels to blit. Only bits 0 - 9 are used in 4 bit  |
| 0xEF9  | mode and only bits 1 - 9 are used in 8 bit mode. Setting all the  |
|        | used bits zero results in 1024 (4 bit) or 512 (8 bit) pixels.     |
+--------+-------------------------------------------------------------------+
| 0xEFA  | Source high (Video RAM bank) part (64K * 32 bit units). Only low  |
|        | 2 bits are used since the Video RAM's size is 256K * 32 bits.     |
+--------+-------------------------------------------------------------------+
| 0xEFB  | Destination high (Video RAM bank) part (64K * 32 bit units). Only |
|        | low 2 bits are used since the Video RAM's size is 256K * 32 bits. |
+--------+-------------------------------------------------------------------+
| 0xEFC  |                                                                   |
|   \-   | Unused. Writable, the values written here are preserved.          |
| 0xEFE  |                                                                   |
+--------+-------------------------------------------------------------------+
|        | Start on write, Pattern for Line Filler (LI). A write to this     |
| 0xEFF  | location starts the accelerator operation using the current       |
|        | values in the other registers.                                    |
+--------+-------------------------------------------------------------------+

The Destination increment (0xEF4) normally should be set to one (1). Otherwise
the Accelerator still performs the same way (also in calculating masks for
begin, middle, and end cells), just the result goes in a different layout.
Setting this to something else than one may be useful for example when
blitting small tiles to cell boundaries (such as emulating a character mode),
so a blit can be performed with less operations.

Note that the accelerator modifies the values written at 0xEE8 - 0xEEB, 0xEF0,
0xEF2 and 0xEF3 according to the descriptions of these fields.

The Accelerator also has a Reindex table in the 0xF00 - 0xFFF range. This
reindex table contains 8 bit values, two in each register. The layout of this
area is as follows:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | First reindex table entry, first reindex bank (bank 0).           |
| 0xF00  |                                                                   |
|        | bit  8-15: Reindex for source value 0x0.                          |
|        | bit  0- 7: Reindex for source value 0x1.                          |
+--------+-------------------------------------------------------------------+
| 0xF01  | Reindexes for source values 0x2 and 0x3, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF02  | Reindexes for source values 0x4 and 0x5, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF03  | Reindexes for source values 0x6 and 0x7, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF04  | Reindexes for source values 0x8 and 0x9, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF05  | Reindexes for source values 0xA and 0xB, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF06  | Reindexes for source values 0xC and 0xD, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF07  | Reindexes for source values 0xE and 0xF, bank 0.                  |
+--------+-------------------------------------------------------------------+
| 0xF08  | Further reindex banks (banks 1 - 31) to specify 512 reindex       |
|   \-   | values in total.                                                  |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+

Note that the value order accords with the Big Endian scheme the system uses.

In 4 bit mode the high 4 bits of each reidex value are left unused.
