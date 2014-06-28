
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


Overview
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The display generator produces the video signal as required by the targeted
display standard from the appropriate data in the Video RAM, line by line.

A line renderer generates the display line from a display list providing
parameters for loading chunks of source data into an internal line double
buffer's render side. The buffers are flipped on advancing a line, so next
line the previous render side becomes a display side, and is used to generate
the video signal producing pixel data. ::


    +-------------------------+
    | Video RAM               |
    +-------------------------+
                 |
                 | Video bus access every second cycle
                 V
    +-------------------------+
    | Line renderer           |
    +-------------------------+
                 |
                 |
    +------------|------------ Line double buffer ---------------------------+
    |            V                                                           |
    |  +------------- Render side -------------+--------------------------+  |
    |  |                                       |                          |  |
    |  +- 80 x 32 bit cells display data ------+- 48 x 32 bit cells void -+  |
    |  |                                       |                          |  |
    |  +------------- Display side ------------+--------------------------+  |
    |            |                                                           |
    +------------|-----------------------------------------------------------+
                 |
                 |     +--------------+
                 |<----| Palette data |
                 |     +--------------+
                 V
           Video signal


A display line is 640 x 4 bit pixels or 320 x 8 bit pixels depending on the
display mode, taking 80 x 32 bit Video RAM cells. One buffer in the Line
double buffer accordingly is capable to hold 80 x 32 bits of data, while it's
cells may have a 7 bit address. The cells addressable with these address bits
(cells 80 - 127) do not contribute to the Video signal, and so they may not
be implemented.

The buffers are typically flipped when advancing a line, that is every 400
main clock cycles. The display generator supports a doubly scanned mode when
the buffers are only flipped after every second line.

The Render side also contains a reset circuity which can reset the state of
all cells to a given initial value in a single clock.

Due to this architecture the Line renderer is free to build up the following
display line in any order as long as it fits in the line's cycle budget.

To give time slots for other components accessing the Video RAM (on the Video
bus) the Display generator is capable to access the bus on every second main
clock cycle, so allowing 200 Video bus accesses per line.


Display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Line renderer operates based on a display list concept, which list
provides a sequence of rendering commands to be performed on the line. The
Line renderer fetches and performs these commands as far as the line's (or
the pair of lines' in doubly scanned mode) cycle budget permits or the list is
drained.

If the cycle budget is exhausted, the rendering of the line is simply
terminated, and the following line is started normally.

The first command of a line or line pair's command set is a 32 bit background
pattern which is used to reset the Render side of the Line double buffer.
Subsequent commands are rendering commands which combine a line of data from
the Video RAM onto the Render side of the Line double buffer.

The processing is adequately pipelined so no Video bus access cycles are spent
idle as long as there is data to render for the line. From the user's point of
view the Line renderer may be seen as fetching a display list command, then
processing it. Up to 8 bus access cycles per line or line pair is however lost
for overhead, so up to 192 bus access cycles remain available for processing
by this scheme (392 in double scanned mode).

(Implementations are allowed to deviate from the strictly sequential scheme in
favor of meeting the bus access cycle requirement by pipelining, such as by
pre-fetching some cells of the display list)




Graphics Display Generator memory map and command layouts
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
|        | Shift mode region                                                 |
| 0x002  |                                                                   |
|        | - bit    15: Double scanned mode if set                           |
|        | - bit  8-14: Output width in cells (0: No output)                 |
|        | - bit     7: Unused                                               |
|        | - bit  0- 6: Begin position in cells                              |
|        |                                                                   |
|        | Specifies the region of output for Shift mode sources. The bus    |
|        | access cycles required are one more than the output width.        |
+--------+-------------------------------------------------------------------+
|        | Display list definition                                           |
| 0x003  |                                                                   |
|        | - bit  9-15: Unused                                               |
|        | - bit  2- 8: Display list start offset in 2048 VRAM cell units    |
|        | - bit  0- 1: Display list entry / line size                       |
|        |                                                                   |
|        | Display list entry / line sizes:                                  |
|        |                                                                   |
|        | - 0: 4 / 8 (double scan) entries                                  |
|        | - 1: 8 / 16 (double scan) entries, bit 3 is unused                |
|        | - 2: 16 / 32 (double scan) entries, bits 3-4 are unused           |
|        | - 3: 32 / 64 (double scan) entries, bits 3-5 are unused           |
|        |                                                                   |
|        | Note that in double scanned mode there are only 200 lines, so the |
|        | total size of the display list is identical to that of the single |
|        | scanned mode.                                                     |
+--------+-------------------------------------------------------------------+
|        | Source definition 0                                               |
| 0x004  |                                                                   |
|        | - bit  8-15: Base offset bits 8-15                                |
|        | - bit  6- 7: VRAM bank select                                     |
|        | - bit     5: If set, shift source. If clear, positioned source.   |
|        | - bit  3- 4: Width multiplier for positioned source               |
|        | - bit  0- 2: Width in cells                                       |
|        |                                                                   |
|        | Width multipliers:                                                |
|        |                                                                   |
|        | - 0: x1                                                           |
|        | - 1: x3                                                           |
|        | - 2: x5                                                           |
|        | - 3: x7                                                           |
|        |                                                                   |
|        | Width in cells:                                                   |
|        |                                                                   |
|        | - 0: 1 cell (8 pixels in 4 bit mode, 4 pixels in 8 bit)           |
|        | - 1: 2 cells                                                      |
|        | - 2: 4 cells                                                      |
|        | - 3: 8 cells                                                      |
|        | - 4: 16 cells                                                     |
|        | - 5: 32 cells                                                     |
|        | - 6: 64 cells                                                     |
|        | - 7: 128 cells                                                    |
|        |                                                                   |
|        | The width multiplier only has effect for positioned sources. It   |
|        | multiplies the width for the render, however only the width is    |
|        | used to calculate offsets in the source.                          |
|        |                                                                   |
|        | Shift sources wrap around on their end when rendering, always     |
|        | producing the output width defined in the Shift mode region       |
|        | register.                                                         |
+--------+-------------------------------------------------------------------+
| 0x005  | Source definition 1                                               |
+--------+-------------------------------------------------------------------+
| 0x006  | Source definition 2                                               |
+--------+-------------------------------------------------------------------+
| 0x007  | Source definition 3                                               |
+--------+-------------------------------------------------------------------+
| 0x008  |                                                                   |
| \-     | Accelerator registers. See "acc_arch.rst".                        |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+

Display lists hold commands, each command defining one chunk of data to be
rendered on the Render side of the Line double buffer. The first entry of a
line of a display list is a background pattern which is used to reset the
Render side of the Line double buffer before starting the render. Subsequent
entries (up to 3, 7, 15, 31 or 63 depending on entry size and double scanning)
are render commands.

The layout of a render command is as follows:

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
|        | If set, bit 3 (4 bit mode) or bit 5 (8 bit mode) of destination   |
| 31     | pixel values become a priority selector. If bit 3 / 5 of the      |
|        | destination pixel is set, it won't be overriden by the source     |
|        | pixel.                                                            |
+--------+-------------------------------------------------------------------+
| 30     | Combine with colorkey if clear (!)                                |
+--------+-------------------------------------------------------------------+
| 28-29  | Source definition select                                          |
+--------+-------------------------------------------------------------------+
|        | Source line select. This is multiplied with the width of the      |
| 16-27  | source (not including the multiplier) to produce a VRAM offset,   |
|        | and is OR combined with the source's base offset. Video RAM bank  |
|        | boundaries can not be crossed.                                    |
+--------+-------------------------------------------------------------------+
| 15     | Combine with mask if clear (!)                                    |
+--------+-------------------------------------------------------------------+
|        | Mask / Colorkey extend mode for pixel bits 4-7                    |
| 14     |                                                                   |
|        | - 0: bits 4-7 are zero                                            |
|        | - 1: bits 0-3 are zero, and bits 4-7 get the Mask / Colorkey      |
|        |   value with some exceptions.                                     |
|        |                                                                   |
|        | Exceptions are based on the Mask / Colorkey value as follows:     |
|        |                                                                   |
|        | - 0101 (binary): Mask / Colorkey is 00011111 (binary).            |
|        | - 1010 (binary): Mask / Colorkey is 00111111 (binary).            |
|        | - 1011 (binary): Mask / Colorkey is 01111111 (binary).            |
|        | - 1101 (binary): Mask / Colorkey is 11111111 (binary).            |
+--------+-------------------------------------------------------------------+
| 10-13  | Mask / Colorkey value for pixel bits 0-3 (unless in extend mode)  |
+--------+-------------------------------------------------------------------+
|        | Shift / Position amount in 4 bit pixel units. If the source is in |
| 0-9    | shift mode, this value shifts it to the left by the given number  |
|        | of pixel units. If the source is in position mode, this value     |
|        | determines it's start position on the Render side of the Line     |
|        | double buffer. In 8 bit mode the lowest bit is ignored.           |
+--------+-------------------------------------------------------------------+

A render command is inactive if it's bits 15 and 10-13 are set zero. Such a
render command does not contribute to the line's contents, and only takes one
bus access cycle (the cycle in which it was fetched).




Rendering process
------------------------------------------------------------------------------


The rendering process for cells are identical for Shift and Position modes,
and is carried out according to the following guide: ::


    +----+----+----+----+
    |    Source data    | As read from the Video RAM
    +----+----+----+----+
              |
              +------------+ Shift to align with destination
                           V
    +----+----+----+----+----+----+----+----+
    | Prev. src. |   Current source  |      | Shift register
    +----+----+----+----+----+----+----+----+
              |
              |                                        Mask / C.key value
              V                                              | |
    +----+----+----+----+ If c.key  +----+----+----+----+    | |
    |    Data to blit   |---------->|   Colorkey mask   |<---+ |
    +----+----+----+----+           +----+----+----+----+      | If mask
              |                               |                V
              |    +----+----+----+----+      |      +----+----+----+----+
              |    |   Priority mask   |----+ | +----|      Bit mask     |
              |    +----+----+----+----+    | | |    +----+----+----+----+
              |              A              | | |
              |              | If priority  | | |    +----+----+----+----+
              |              |              | | | +--|  Beg/Mid/End mask |
              |              |              | | | |  +----+----+----+----+
              |              |             _V_V_V_V_
              |              |            |   AND   |
             _V_             |             ~~~~|~~~~
            |AND|<----------)|(----------------+
             ~|~             |                 |
             _V_     ___     |                _V_
            | OR|<--|AND|<--)|(--------------|NEG|
             ~|~     ~A~     |                ~~~
              |       |      |
              |       +------+
              |       |
              V       |
     ---+----+----+----+----+---
        | Target r.buf cell |
     ---+----+----+----+----+---


The Beg/Mid/End mask is used in Position mode to mask the partially filled
cells on the beginning and the end of the rendered streak of data.

In Shift mode the fractional part (low 3 bits) of the Shift / Position amount
is 2's complement negated to produce the alignment shift. In Shift mode
typically a source cell has to be fetched in advance (without producing
destination for it), so the shift register may be properly filled for the
first output data.




Renderer cycle budget
------------------------------------------------------------------------------


As defined in the "Display List" chapter, in single scanned mode from the
user's point of view there are at least 192 useful Video bus access cycles,
and in doubly scanned mode, there are 392.

The rendering from the user's point of view may be interpreted as being
sequential: the renderer fetches a display list command, then processes it,
then goes on to the next command as long as there are commands for the line
and there are bus access cycles remaining for the render.

Bus access cycles are taken by the following rules:

- 1 cycle for reading a display list command.
- The Shift mode region's Output width count of cycles plus one for sources in
  Shift mode.
- The multiplied width count of cycles for sources in Position mode.

Note that the renderer is not capable to optimize out access cycles which
would be used to render into off-screen area, neither it has a limit on how
many cycles may it consume for a command in Position mode (that is defining a
source of width 128 x 7 would cause 896 bus access cycles to be taken for it's
render unless the line's end terminated it).

(Pipelining notes: if a source is in Position mode, to render it on the Line
double buffer, one more cycle is necessary than it's width. This extra cycle
should be performed in parallel with a display list command fetch)




Other components of the Display Generator
------------------------------------------------------------------------------


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

- The exact number of bus access cycles available for render beyond the
  required 192 / 392 cycles, and how the pipeline behaves regarding the
  termination of a render because of exhausting the cycle budget. Note that if
  by the specification the effective render finishes within the cycle budget
  leaving only disabled display list commands which may be terminated, the
  behavior must be defined (the terminated display list must not affect the
  contents of the line). The next line or line pair's render must always start
  proper regardless of the termination of the line or line pair before.

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

