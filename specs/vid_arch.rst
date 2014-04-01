
RRPGE Graphics Display & Accelerator, Display component architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the display component from the Graphics
Display & Accelerator unit. The display component is responsible for
generating the signals producing the visual image on the display hardware, as
required by the targeted display standard.




Basic properties of the display
------------------------------------------------------------------------------


The limits of the RRPGE system's display generator are set as follows:

- 640x400 visible pixels of 1:1 pixel aspect ratio in 4bit mode
- 320x400 visible pixels of 2:1 pixel aspect ratio in 8bit mode
- 16:10 display aspect ratio
- 50Hz minimal / 70Hz maximal refresh rate
- At least 49 vertical blank lines (for a total of at least 449 lines)
- At least 400 main clock cycles per display line
- Interlaced display generation is supported for standards requiring it
- 32 bit Video RAM memory bus width is assumed

The minimal number of main clock cycles required to be present under a
vertical blanking period or a display frame can be derived from these
requirements (at least 400 cycles per display line: at least 19600 cycles for
a vertical blanking period and at least 179600 cycles for a frame). Note that
the kernel is allowed to take a certain maximal amount of cycles for internal
tasks which reduces the amount of cycles available for user mode programs
within these periods. See "Kernel timing constraints" in "kernel.rst" for
further details. The ultimate low end limit for the available cycles in a
vertical blanking period is 12800, and within a display frame is 143680.

To support the 4bit mode the graphic display normally has to be clocked by
twice the frequency of the main clock.

Interlaced modes are implemented in a transparent way: they require the same
programming from the user like non-interlaced modes. The only exception is
that the user needs to be aware of that it might need two frames to produce a
complete image. See the "Interlaced rendering" chapter for more information.

The Video RAM memory bus width is assumed to be 32 bits. Several aspects of
the Graphics Display & Accelerator are specified with this in mind, and the
minimal memory access characteristics normally also require the 32 bit bus.
Other implementations might be possible as long as they meet the minimal
requirements of this specification.

Note that faster rates than 70Hz may be provided by an implementation if it
meets the minimal processing power requirements for a display frame outlined
above. To provide this the implementation has to be faster than the minimal
requirements in all aspects.




Display standards and RRPGE system compatibility
------------------------------------------------------------------------------


VGA 400 line / 70Hz
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the display mode with the fastest refresh rate the RRPGE system
supports: it's minimal display related timings are set out based on this mode.
To produce it a 25.175MHz base clock is necessary from which a slightly faster
than the minimal 12.5MHz main clock can be derived.

Normally this mode's aspect ratio is not suitable for the system, however if
used with a CRT monitor, it's aspect ratio can be corrected manually to fit,
offering a more eye friendly visual experience.


VGA 480 line / 60Hz
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To support this mode ("standard VGA") a 25.175MHz base clock is necessary from
which a slightly faster than the minimal 12.5MHz main clock can be derived.

The extra display lines this mode provides are not used by the display
generator, normally leaving 40 pixel tall blank areas on the top and the
bottom of a 4:3 display. In total there are 125 vertical blank lines.


NTSC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The timings are similar to that of the VGA 480 line / 60Hz mode except that
this video standard requires half the pixel clock, generating an interlaced
display.

See the "Interlaced rendering" chapter for more details.


PAL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To support this standard a 12.5MHz base clock is required (which is also the
main clock). This allows generating a PAL compatible signal with 640 visible
and 160 horizontal blanking pixels per line.

In this mode there are 225 vertical blank lines.

See the "Interlaced rendering" chapter for more details.




Display generator architecture
------------------------------------------------------------------------------


The display generator provides a background pattern and up to four
independently scrollable display layers of which it composes the display in
real time.

The display layers' properties and pixel sources are controlled through
display lists, a common one for most properties and a background pattern, and
four layer specific ones for defining each layer's output. These all contain
one 32 bit entry for each of the 400 displayed lines.

Pixel data is rendered from left to right, and within the Video RAM it is laid
out in Big Endian order (leftmost pixels on the higher bit positions of a
Video RAM cell). This is consistent with the method the CPU implements
sub-word addressing modes.

The following block diagram depicts the workflow of the display generator as
seen by the user, simplified: ::


    +Peripheral-+       +----VRAM---+                        +---+
    |           |       |           |                        |   |
    | +-------+ |       | +-------+ |  +----Bg.pattern------>|   |
    | |Backgrn| |       | | Backg | |  :                     |   |
    | | list  |-----+---->| disp. |----+  +--------------+   |   |
    | |pointer| |   ^   | | list  | |  :  | Layer, bank, |   |   |
    | +-------+ |   :   | +-------+ |  +->| mask and     |   |   |
    |           |   :   |           |     | colorkey     |   |   |
    |           |   :   |           |     | selector     |   | S |
    | +-------+ |   :   | +-------+ |     +--------------+   | h |
    | |Display| |   :   | | Disp. | |        V    :   :      | i |
    | |list 0 |-----+---->| list  |----->+<--+    :   :      | f |
    | |pointer| |   ^   | | 0     | |    :   :    :   :      | t |
    | +-------+ |   :   | +-------+ |    :  |E|   :   :      |   |
    |           |   :   |           |    :  |n|   :   :      | & |
    |           |   :   | +-------+ |    :  |a|   :   :      |   |
    |           |   :   | | Line  |<-----+  |b|   :   :      | C |
    |           |   :   | | pixel | |       |l|   V   V      | o |
    |           |   :   | | data  |--------)|e|(--+===+=====>| m |
    |           |   :   | +-------+ |       |d|   :   :      | b |  +---+
    |           |   :   |           |        :   |M| |C|     | i |  | P |
    | +-------+ |   :   | +-------+ |        :   |a| |o|     | n |  | a |
    | |Display| |   :   | | Disp. | |        V   |s| |l|     | e |  | l |
    | |list 1 |-----+---->| list  |----->+<--+   |k| |o|     |   |  | e |
    | |pointer| |   ^   | | 1     | |    :   :    :  |r|     |   |  | t |
    | +-------+ |  |L|  | +-------+ |    :   :    :  |k|     |   |  | t |
    |           |  |i|  |           |    :   :    :  |e|     |   |  | e |
    |           |  |n|  | +-------+ |    :   :    :  |y|     |   |  +---+
    |           |  |e|  | | Line  |<-----+   :    :   :      |   |    :
    |           |  | |  | | pixel | |        :    V   V      |   |    V
    |           |  |p|  | | data  |---------):(---+===+=====>|   |----+--->
    |           |  |t|  | +-------+ |        :    :   :      |   |
    |           |  |r|  |           |        :    :   :      |   |
    | +-------+ |   :   | +-------+ |        :    :   :      |   |
    | |Display| |   V   | | Disp. | |        V    :   :      |   |
    | |list 2 |-----+---->| list  |----->+<--+    :   :      |   |
    | |pointer| |   :   | | 2     | |    :   :    :   :      |   |
    | +-------+ |   :   | +-------+ |    :   :    :   :      |   |
    |           |   :   |           |    :   :    :   :      |   |
    |           |   :   | +-------+ |    :   :    :   :      |   |
    |           |   :   | | Line  |<-----+   :    :   :      |   |
    |           |   :   | | pixel | |        :    V   V      |   |
    |           |   :   | | data  |---------):(---+===+=====>|   |
    |           |   :   | +-------+ |        :    :   :      |   |
    |           |   :   |           |        :    :   :      |   |
    | +-------+ |   :   | +-------+ |        :    :   :      |   |
    | |Display| |   V   | | Disp. | |        V    :   :      |   |
    | |list 3 |-----+---->| list  |----->+<--+    :   :      |   |
    | |pointer| |       | | 3     | |    :        :   :      |   |
    | +-------+ |       | +-------+ |    :        :   :      |   |
    |           |       |           |    :        :   :      |   |
    |           |       | +-------+ |    :        :   :      |   |
    |           |       | | Line  |<-----+        :   :      |   |
    |           |       | | pixel | |             V   V      |   |
    |           |       | | data  |---------------+===+=====>|   |
    |           |       | +-------+ |                        |   |
    |           |       |           |                        |   |
    +-----------+       +-----------+                        +---+


Within every displayed line (that is lines 0-399), in the horizontal blanking
period before the line, the data at the offset specified by the line pointer
is read from all five display lists (background and four layer display lists).

For the display cycles of the line (this is 320 main clock cycles) each of
the enabled layer's display data is read, combined, and output to the display.

The display list reads and the graphics output in the display cycles require
several memory accesses which cause stalls. Read the "Addressing stalls"
section for further information on the layout and effects of these. The
Display component has the highest priority in accessing the Video RAM.

The background pattern and the four display layers have fixed priority order.
From lowest to highest this is as follows:

- Background pattern
- Display layer 0
- Display layer 1
- Display layer 2
- Display layer 3

A higher priority layer may hide parts of the image composed from the lower
priority layers in two ways:

- Mask. The higher priority layer defines bit planes within the pixel data
  which bit planes will be taken from it's data, hiding the respective bits
  from the composition underneath. With a suitable palette this mode may be
  utilized for transparent blending.

- Colorkey. The higher priority layer defines a color index. Every pixel not
  having this index from it's data will hide the respective pixels of the
  composition underneath. If the pixel in it's data matches this index, then
  the pixel from the composition underneath will be shown. For the colorkey
  matching only those bit planes are taken in account which will not be taken
  away by higher priority layers in Mask mode or a global mask.


Background display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The background display list defines the background pattern and the common
properties for the individual display layers. It has 400 entries, one for each
line, one entry is 32 bits. According to the Big Endian scheme, the CPU sees
it's high 16 bits as it's first word, and the low 16 bits as it's second word.

+-------+--------------------------------------------------------------------+
| Bits  | Description                                                        |
+=======+====================================================================+
| 30-31 | Layer 0 Video RAM bank selection (0-3)                             |
+-------+--------------------------------------------------------------------+
| 28-29 | Layer 1 Video RAM bank selection (0-3)                             |
+-------+--------------------------------------------------------------------+
| 26-27 | Layer 2 Video RAM bank selection (0-3)                             |
+-------+--------------------------------------------------------------------+
| 24-25 | Layer 3 Video RAM bank selection (0-3)                             |
+-------+--------------------------------------------------------------------+
| 21-23 | Global mask (number of bits masked starting from the high end)     |
|       |                                                                    |
|       | - 0: 0xFF (8bit) / 0xF (4bit); 256 colors (all enabled)            |
|       | - 1: 0x7F (8bit) / 0xF (4bit); 128 colors                          |
|       | - 2: 0x3F (8bit) / 0xF (4bit); 64 colors                           |
|       | - 3: 0x1F (8bit) / 0xF (4bit); 32 colors                           |
|       | - 4: 0x0F (8bit) / 0xF (4bit); 16 colors                           |
|       | - 5: 0x07 (8bit) / 0x7 (4bit); 8 colors                            |
|       | - 6: 0x03 (8bit) / 0x3 (4bit); 4 colors                            |
|       | - 7: 0x01 (8bit) / 0x1 (4bit); 2 colors                            |
+-------+--------------------------------------------------------------------+
| 19-20 | Layer mask & colorkey usage. Defines the usage of the mask and     |
|       | colorkey field in the layer display lists.                         |
|       |                                                                    |
|       | - 0: Layer0: CKey; Layer1: Mask; Layer2: Mask; Layer3: Mask        |
|       | - 1: Layer0: CKey; Layer1: Ckey; Layer2: Mask; Layer3: Mask        |
|       | - 2: Layer0: CKey; Layer1: Ckey; Layer2: Ckey; Layer3: Mask        |
|       | - 3: Layer0: CKey; Layer1: Ckey; Layer2: Ckey; Layer3: Ckey        |
+-------+--------------------------------------------------------------------+
| 16-18 | Layer configuration (enabled layers). See table further below.     |
+-------+--------------------------------------------------------------------+
|  0-15 | Background pattern. In 4bit mode this is 4 colors in the usual     |
|       | high (leftmost) to low (rightmost) order, in 8 bit mode 2 colors.  |
|       | The global mask does not affect this field.                        |
+-------+--------------------------------------------------------------------+

The Video RAM is partitioned into 64K * 32 bit unit banks across which
addresses may not increment (this increment is further limited by the
partition size setting at 0xEE2, see later). The Video RAM bank selection
fields for each layer select one from the first four such partitions. Note
that even if the actual display memory would be larger, displaying from
beyond the first four banks is not supported (in the RRPGE system however the
display memory is defined to be exactly four such banks).

The Global mask limits the effective color bits of all display layers except
the background. It may be used to force such limitation if the application
uses less colors: this can be useful for freeing up bit planes of the higher
bits for further data storage (such as sprites or tiles; which may be
exploited using the Accelerator component).

Note that the background is always fully covered by pixels not matching the
colorkey of a colorkeyed layer even if it uses bit planes disabled for the
layers by a global mask.

Note that all colorkey layers share the same mask, that is the same bit planes
will be effective of each determined by the Global mask and the higher
priority layer masks as specified by the Layer mask & colorkey usage field.

The Layer configuration field provides the following configurations:

+-------+---------+---------+---------+---------+
| Value | Layer 0 | Layer 1 | Layer 2 | Layer 3 |
+=======+=========+=========+=========+=========+
|     0 |         |         | Enabled |         |
+-------+---------+---------+---------+---------+
|     1 | Enabled |         | Enabled |         |
+-------+---------+---------+---------+---------+
|     2 | Enabled |         | Enabled | Enabled |
+-------+---------+---------+---------+---------+
|     3 | Enabled | Enabled | Enabled | Enabled |
+-------+---------+---------+---------+---------+
|     4 | Enabled |         |         |         |
+-------+---------+---------+---------+---------+
|     5 | Enabled |         | Enabled |         |
+-------+---------+---------+---------+---------+
|     6 | Enabled | Enabled | Enabled |         |
+-------+---------+---------+---------+---------+
|     7 | Enabled | Enabled | Enabled | Enabled |
+-------+---------+---------+---------+---------+

Note that the mask of the layer is effective even if the layer is not enabled
for graphics output.


Layer display lists
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The layer display lists provide the properties for each of the layers
individually. Like the background display list, these have 400 entries with
each entry being 32 bits in size.

+-------+--------------------------------------------------------------------+
| Bits  | Description                                                        |
+=======+====================================================================+
| 16-31 | Source pixel data pointer, whole (32bit VRAM cell unit) part       |
+-------+--------------------------------------------------------------------+
|  9-15 | Source pixel data pointer, fractional part                         |
+-------+--------------------------------------------------------------------+
|     8 | Absolute pointer if set, relative (additive) pointer if clear      |
+-------+--------------------------------------------------------------------+
|  0- 7 | Mask or colorkey data. Only the bits not masked be the Global mask |
|       | or higher priority layer masks are effective. In mask mode set     |
|       | bits indicate that the appropriate bit plane from this layer's     |
|       | pixel data is effective.                                           |
+-------+--------------------------------------------------------------------+

For the first line (Line 0) the source pixel data pointer is assumed to be
zero if the first entry of the display list contains a relative pointer.

The relative pointer is applied before starting processing the line.

Adding the relative pointer and increments during the output of the line are
constrained by the 64K * 32 bit Video RAM banks, and further by the
partition size setting at 0xEE2. The partition to work within is selected by
the absolute address (or the previous address in relative mode). The address
will wrap around to the beginning of the partition when passing it's boundary.

Using the relative pointer mode eases implementing the scrolling of the layer
since using it only a singe absolute address have to be written to affect
multiple lines.

The fractional part of the address determines the pixel precise start offset
of the layer line, which is effectively a left shift of the 32 bit source
data. In 4 bit mode the high 3 bits of the fraction give distinct visible
shifts (as 8 pixels fill a 32 bit Video RAM cell), in 8 bit mode, the high 2
bits. Note that the rest of the fraction bits are also implemented and may
have visible effect through relative pointer additions in several consequent
lines.

Note that for each display list line in total 81 Video RAM cells of pixel
data are read in. The first cell is read in separately within the Horizontal
Blanking period before the line, subsequent cells (80) are read in within the
Display period of the line.


The line counter & pointer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The line counter & pointer have two roles. First as pointer it addresses the
display lists in visible lines (lines 0-399), second as counter it triggers
raster interrupts and provides for the "Query current display line" kernel
call (see the appropriate section in "kcall.rst", and the "Video raster
passed" section in "kernel.rst").

Through these features it conveys information to the user application which
may synchronize to it for various purposes implementing graphics engines.

The line counter & pointer increments and triggers it's interrupt when
entering the Horizontal Blanking period of the line it refers to. That is the
Line counter & pointer will be zero within the first displayed line's
Horizontal Blank, and it's 320 (main clock) Display cycles.

Note that the Vertical blanking period is handled specially. Raster interrupt
can only be set to trigger at the start of the Vertical blank (that is, when
completing Line 399 and incrementing from it). The "Query current display
line" kernel call returns the lines remaining to the next frame's Line 0
calculated from this value and the knowledge of the total number of VBlank
lines.


Palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette can only be written through kernel calls. This component only
affects the generated data, assigning the actual visible colors to each pixel
of the output stream. In real hardware it might be a rather simple Digital
Analog Converter (DAC).

Colors are expressed as 16 bit RGB values in the following layout:

+-------+--------------------------------------------------------------------+
| Bits  | Description                                                        |
+=======+====================================================================+
| 12-15 | Unused                                                             |
+-------+--------------------------------------------------------------------+
|  8-11 | Red component (0 - 15)                                             |
+-------+--------------------------------------------------------------------+
|  4- 7 | Green component (0 - 15)                                           |
+-------+--------------------------------------------------------------------+
|  0- 3 | Blue component (0 - 15)                                            |
+-------+--------------------------------------------------------------------+


Implementation defined
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Some aspects of the Display generator which may be accessible to the
application programmer are declared "Implementation defined" to allow for
simpler emulation or to restrict probable hardware implementations less. These
are as follows:

- The timing of any display related Video RAM access within the rendered line.
  Note that this does not refer to the layout of stalls which is defined, only
  to the timing of data fetches. No Video RAM accesses for a line must happen
  before incrementing the Line counter & pointer to the given line, and no
  Video RAM accesses must happen for a line after the Line counter & pointer
  is incremented beyond it.

- After setting the palette data through the kernel call, it's effect may
  delay for up to "a few" frames, not even necessarily taking effect in
  Vertical Blank period. It must not affect any data rendered before the call.
  Note that the limit is loosely set to allow for software emulators using
  actual palettized displays, not necessarily being capable of synchronizing
  to the display hardware. These can't guarantee fast response if they also
  have to skip frames.




Addressing stalls
------------------------------------------------------------------------------


The Video Display component only accesses the Video RAM in display lines (and
not at all in VBlank), thus only generating stalls during these.

The effect of the stalls from the point of minimal limits to support is
described in the "Memory access stalls" section of the CPU instruction set
("cpu_inst.rst") and in the "Accelerator operation timing" section of the
Display accelerator's specification ("acc_arch.rst").

Below the hardware level assumptions are described leading to defining those
minimal timing requirements.

The Graphics Display & Accelerator unit is capable to perform one memory
access each main clock cycle. Since it's bus width is 32 bits, this is a 32
bit access. Both in 4 bit and 8 bit mode one layer of pixel data takes 320 + 4
bytes, which is 80 + 1 Video RAM cells to read in. The extra (first of the
line) cell is read in separately in the Horizontal Blanking period. If all
four layers are enabled, the Display component necessarily has to access the
Video RAM in every clock cycle of a 320 cycle Display period within the
display line.

To simplify hardware accesses are performed in a fixed pattern, as follows: ::

    0  1  2  3  4  5  6  7  8  9  ... (main clock cycles)
    L0 L1 L2 L3 L0 L1 L2 L3 L0 L1 ... (accessed layer)

When some layers are disabled, their access is omitted, freeing that slot for
the accelerator or the CPU if either waits for accessing the Video RAM. Note
that two layers can only enabled as L0 + L2, so producing an alternating
access pattern between display output and other functions. This is relied upon
in the design of accelerators.

In the Horizontal Blanking of displayed lines the same access pattern is
assumed like if one layer was enabled. Note that up to 9 accesses need to be
performed in total in this period: one access for each Display List, and four
accesses for reading in the first cells of each line.




Video peripheral, Display component related memory map
------------------------------------------------------------------------------


The following table describes those elements of the Video peripheral area
which are related to the Display component. Note that these are as seen from
the CPU: the cells behind these offsets have a 16 bit width. Also note that
these repeat every 32 words in the 0xE00 - 0xEFF range, so for example offsets
0xE02, 0xE22, 0xE42 ... 0xEE2 all refer to the Video RAM partition size
register.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xEE0  | Video RAM write mask (0xEE0: High, 0xEE1: Low). Does not belong   |
|   \-   | to the display unit, see "acc_arch.rst" for details.              |
| 0xEE1  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xEE2  | Video RAM partition size. Defines further partitioning within the |
|        | Video RAM banks. Only the low 3 bits are used (the rest may be    |
|        | written but are ignored). Note that this setting also applies to  |
|        | the accelerator.                                                  |
|        |                                                                   |
|        | - 0: 512 * 32 bit cells                                           |
|        | - 1: 1K * 32 bit cells                                            |
|        | - 2: 2K * 32 bit cells                                            |
|        | - 3: 4K * 32 bit cells                                            |
|        | - 4: 8K * 32 bit cells                                            |
|        | - 5: 16K * 32 bit cells                                           |
|        | - 6: 32K * 32 bit cells                                           |
|        | - 7: 64K * 32 bit cells                                           |
+--------+-------------------------------------------------------------------+
| 0xEE3  | Background display list offset in 512 * 32 bit VRAM cell units.   |
|        | The display list occupies the first 400 cells of the area. High   |
|        | bits which would address outside the Video RAM are ignored.       |
+--------+-------------------------------------------------------------------+
| 0xEE4  | Layer 0 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0xEE5  | Layer 1 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0xEE6  | Layer 2 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0xEE7  | Layer 3 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0xEE8  | Scaled blitter pointers and increments. Does not belong to the    |
|   \-   | display unit, see "acc_arch.rst" for details.                     |
| 0xEEF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xEF0  | Video accelerator area. Does not belong to the display unit, see  |
|   \-   | "acc_arch.rst" for details.                                       |
| 0xEFF  |                                                                   |
+--------+-------------------------------------------------------------------+




Interlaced rendering
------------------------------------------------------------------------------


For interlaced standards an interlaced rendering mechanism has to be
supported. The key concepts behind it is that it should be as transparent for
the user as reasonably possible.

To achieve this the graphic display unit's line fetching stage works in an
identical way to the "normal" (non interlaced) targets, if meeting the minimal
requirements, fetching a complete line in 400 cycles. The display however
completes a line in 800 cycles, but after each line, it advances it's line
pointer by two.

The actual display stage is necessarily detached from the fetching stage,
communicating through a buffer capable to hold at least one complete line.

The contents of this buffer are implementation defined, to meet the
requirements of this specification, it is sufficient to provide the combined
binary data as seen before applying the palette data.

This style of implementation is necessary to fully conform with this
specification as it requires that reading the pixel data for a line begins
after advancing the line pointer to the line in question, and completes
before advancing it further, on which behavior applications may rely.

Note that the requirements for applying palette data is less strict, so it is
not necessary to provide line exact behavior for this aspect.

