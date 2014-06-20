
Graphics Display Generator architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the Graphics Display Generator
component of the RRPGE system. This component is responsible for generating
the signals producing the visual image on the display hardware, as required by
the targeted display standard.




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
- 32 bit Video bus width

The minimal number of main clock cycles required to be present under a
vertical blanking period or a display frame can be derived from these
requirements (at least 400 cycles per display line: at least 19600 cycles for
a vertical blanking period and at least 179600 cycles for a frame). Note that
the kernel is allowed to take a certain maximal amount of cycles for internal
tasks which reduces the amount of cycles available for user mode programs
within these periods. See "Kernel timing constraints" in "kernel.rst" for
further details. Note that with the proper use of the Graphics FIFO (see
"grapfifo.rst"), which is normally not affected by the kernel, these timings
should not be of too much concern.

To support the 4bit mode (640px width) the graphic display normally has to be
clocked by twice the frequency of the main clock.

Interlaced modes are implemented in a transparent way: they require the same
programming from the user like non-interlaced modes. The only exception is
that the user needs to be aware of that it might need two frames to produce a
complete image. See the "Interlaced rendering" chapter for more information.

The Video bus width is 32 bits. Several aspects of the graphics subsystems are
specified with this in mind, and the minimal memory access characteristics
normally also require the 32 bit bus. Other implementations might be possible
as long as they meet the minimal requirements of this specification.

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


The display generator provides a background pattern and up to two
independently scrollable display layers of which it composes the display in
real time.

The display layers' properties and pixel sources are controlled through
display lists, a common one for most properties and a background pattern, and
two layer specific ones for defining each layer's output. These all contain
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
    | |pointer| |   ^   | | list  | |  :  | Layer, mask, |   |   |
    | +-------+ |   :   | +-------+ |  +->| and colorkey |   |   |
    |           |   :   |           |     | selector     |   |   |
    |           |   :   |           |     +--------------+   | S |
    | +-------+ |   :   | +-------+ |        :    :   :      | h |
    | |Display| |   :   | | Disp. | |        V    :   :      | i |
    | |list 0 |-----+---->| list  |----->+<--+    :   :      | f |
    | |pointer| |   ^   | | 0     | |    :   :    :   :      | t |
    | +-------+ |  |L|  | +-------+ |    :  |E|   :   :      |   |
    |           |  |i|  |           |    :  |n|   :   :      | & |
    |           |  |n|  | +-------+ |    :  |a|   :   :      |   |
    |           |  |e|  | | Line  |<-----+  |b|   :   :      | C |
    |           |  | |  | | pixel | |       |l|   V   V      | o |
    |           |  |p|  | | data  |--------)|e|(--+===+=====>| m |
    |           |  |t|  | +-------+ |       |d|   :   :      | b |  +---+
    |           |  |r|  |           |        :   |M| |C|     | i |  | P |
    | +-------+ |   :   | +-------+ |        :   |a| |o|     | n |  | a |
    | |Display| |   V   | | Disp. | |        V   |s| |l|     | e |  | l |
    | |list 1 |-----+---->| list  |----->+<--+   |k| |o|     |   |  | e |
    | |pointer| |       | | 1     | |    :        :  |r|     |   |  | t |
    | +-------+ |       | +-------+ |    :        :  |k|     |   |  | t |
    |           |       |           |    :        :  |e|     |   |  | e |
    |           |       | +-------+ |    :        :  |y|     |   |  +---+
    |           |       | | Line  |<-----+        :   :      |   |    :
    |           |       | | pixel | |             V   V      |   |    V
    |           |       | | data  |---------------+===+=====>|   |----+--->
    |           |       | +-------+ |                        |   |
    |           |       |           |                        |   |
    +-----------+       +-----------+                        +---+


Within every displayed line (that is lines 0-399), in the horizontal blanking
period before the line, the data at the offset specified by the line pointer
is read from all five display lists (background and two layer display lists).

For the display cycles of the line (this is 320 main clock cycles) each of
the enabled layer's display data is read, combined, and output to the display.

The display list reads and the graphics output in the display cycles require
several memory accesses which cause stalls. Read the "Addressing stalls"
section for further information on the layout and effects of these. The
Display component has the highest priority in accessing the Video RAM.

The background pattern and the two display layers have fixed priority order.
From lowest to highest this is as follows:

- Background pattern
- Display layer 0
- Display layer 1

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
|    31 | Layer 0 Disabled (0) / Enabled (1)                                 |
+-------+--------------------------------------------------------------------+
|    30 | Layer 1 Mask (0) / Colorkey (1) mode                               |
+-------+--------------------------------------------------------------------+
| 24-29 | Unused                                                             |
+-------+--------------------------------------------------------------------+
| 16-23 | Global mask. Only low 4 bits used in 4 bit mode.                   |
+-------+--------------------------------------------------------------------+
|  0-15 | Background pattern. In 4bit mode this is 4 colors in the usual     |
|       | high (leftmost) to low (rightmost) order, in 8 bit mode 2 colors.  |
|       | The global mask does not affect this field.                        |
+-------+--------------------------------------------------------------------+

Layer 0 is always in Colorkey mode. Layer 1 is always enabled.

The Global mask limits the effective color bits of all display layers except
the background. It may be used to force such limitation if the application
uses less colors: this can be useful for freeing up bit planes for further
data storage (such as sprites or tiles; which may be exploited using the
Accelerator).

Note that the background is always fully covered by pixels not matching the
colorkey of a colorkeyed layer even if it uses bit planes disabled for the
layers by the global mask.

The effective bits of Layer 0 are determined by the Global mask, and
optionally Layer 1's mask if it is in mask mode.

If Layer 1 is in Colorkey mode, it's effective bits are determined by the
Global mask.


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
| 13-15 | Source pixel data pointer, fractional part                         |
+-------+--------------------------------------------------------------------+
|    12 | Unused                                                             |
+-------+--------------------------------------------------------------------+
|  8-11 | Partition size                                                     |
|       |                                                                    |
|       | - 0:  4 Words (2 * 32 bit cells)                                   |
|       | - 1:  8 Words (4 * 32 bit cells)                                   |
|       | - 2:  16 Words (8 * 32 bit cells)                                  |
|       | - 3:  32 Words (16 * 32 bit cells)                                 |
|       | - 4:  64 Words (32 * 32 bit cells)                                 |
|       | - 5:  128 Words (64 * 32 bit cells)                                |
|       | - 6:  256 Words (128 * 32 bit cells)                               |
|       | - 7:  512 Words (256 * 32 bit cells)                               |
|       | - 8:  1 KWords (512 * 32 bit cells)                                |
|       | - 9:  2 KWords (1K * 32 bit cells)                                 |
|       | - 10: 4 KWords (2K * 32 bit cells)                                 |
|       | - 11: 8 KWords (4K * 32 bit cells)                                 |
|       | - 12: 16 KWords (8K * 32 bit cells)                                |
|       | - 13: 32 KWords (16K * 32 bit cells)                               |
|       | - 14: 64 KWords (32K * 32 bit cells)                               |
|       | - 15: 128 KWords (64K * 32 bit cells)                              |
+-------+--------------------------------------------------------------------+
|  0- 7 | Mask or colorkey data. Only the bits not masked by the Global mask |
|       | or a higher priority layer mask are effective. In mask mode set    |
|       | bits indicate that the appropriate bit plane from this layer's     |
|       | pixel data is effective.                                           |
+-------+--------------------------------------------------------------------+

The Video RAM bank selection part of the source pixel data pointer is provided
for each layer as peripheral registers accessible by the Graphics FIFO, easing
the implementation of double buffering.

The fractional part of the address determines the pixel precise start offset
of the layer line, which is effectively a left shift of the 32 bit source
data. In 4 bit mode the high 3 bits of the fraction give distinct visible
shifts (as 8 pixels fill a 32 bit Video RAM cell), in 8 bit mode, the high 2
bits.

The partition size is used to wrap the offset during drawing the line to the
beginning of the partition. This is useful for implementing horizontal
scrolling.

Note that for each display list line in total 81 Video RAM cells of pixel
data are read in.


The line counter & pointer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The line counter & pointer has three roles. First as pointer it addresses the
display lists in visible lines (lines 0-399), second as counter it provides
beam wait conditions for the Graphics FIFO (see "Beam wait condition" in
"grapfifo.rst") and results for the "Query current display line" kernel call
(see the appropriate section in "kcall.rst").

Through these features it conveys information to the user application which
may synchronize to it for various purposes implementing graphics engines.

The line counter & pointer increments when entering the Horizontal Blanking
period of the line it refers to. That is the Line counter & pointer will be
zero within the first displayed line's Horizontal Blank, and it's 320 (main
clock) Display cycles.

When entering the Horizontal Blanking, the Graphics Display Generator also
latches all it's registers (the three display list offsets and the two video
RAM bank selections), so any Graphics FIFO operation on these is guaranteed to
only have effect in the next line.


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
  No Video RAM accesses for a line must happen before incrementing the Line
  counter & pointer to the given line, and no Video RAM accesses must happen
  for a line after the Line counter & pointer is incremented beyond it.

- After setting the palette data through the kernel call, it's effect may
  delay for up to "a few" frames, not even necessarily taking effect in
  Vertical Blank period. It must not affect any data rendered before the call.
  Note that the limit is loosely set to allow for software emulators using
  actual palettized displays, not necessarily being capable of synchronizing
  to the display hardware. These can't guarantee fast response if they also
  have to skip frames.




Graphics Display Generator timing
------------------------------------------------------------------------------


The Graphics Display Generator uses a fixed scheme for accessing the Video
bus, generating an access (read) every second cycle irrespective of it's
tasks.

The effect of these accesses from the point of minimal limits to support is
described in the "Memory access stalls" section of the CPU instruction set
("cpu_inst.rst").

Below the hardware level assumptions are described leading to constraining the
Graphics Display Generator to this bus access scheme.

Both in 4 bit and 8 bit mode one layer of pixel data takes 80 + 1 32 bit Video
RAM cells to read in. To read both layers so 162 memory accesses are required,
which take 324 cycles. In addition to these the display lists have to be read,
adding another 3 memory accesses, incrementing the cycle requirements to 330
cycles. This fits in the 400 cycle budget allowed for a line.




Graphics Display Generator memory map
------------------------------------------------------------------------------


The following table describes those elements of the graphics registers which
are related to the Display Generator component. Note that these registers are
only accessible through the Graphics FIFO (see "grapfifo.rst" for details).

The graphics registers in the 0x000 - 0x0FF range repeat every 32 words, so
for example the address 0x020 also refers to the register at 0x000.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Accelerator registers. See "acc_arch.rst".                        |
| 0x001  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x002  | Unused.                                                           |
+--------+-------------------------------------------------------------------+
| 0x003  | Background display list offset in 512 * 32 bit VRAM cell units.   |
|        | The display list occupies the first 400 cells of the area. High   |
|        | bits which would address outside the Video RAM are ignored.       |
+--------+-------------------------------------------------------------------+
| 0x004  | Layer 0 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0x005  | Layer 1 display list offset in 512 * 32 bit VRAM cell units.      |
|        | Works the same way like the Background display list offset.       |
+--------+-------------------------------------------------------------------+
| 0x006  | Layer 0 bank select. Only the low 2 bits are used.                |
+--------+-------------------------------------------------------------------+
| 0x007  | Layer 1 bank select. Only the low 2 bits are used.                |
+--------+-------------------------------------------------------------------+
| 0x008  |                                                                   |
| \-     | Accelerator registers. See "acc_arch.rst".                        |
| 0x1FF  |                                                                   |
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

