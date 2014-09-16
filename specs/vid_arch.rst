
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
- Accesses the 32 bit Peripheral bus

To support the 4bit mode (640px width) the graphic display normally has to be
clocked by twice the frequency of the main clock (except if it only supports
interlaced modes).

Interlaced modes are implemented in a transparent way: they require the same
programming from the user like non-interlaced modes. The only exception is
that the user needs to be aware of that it might need two frames to produce a
complete image. See the "Interlaced rendering" chapter for more information.

Note that faster rates than 70Hz may be provided by an implementation if it
meets the at least 449 line, and at least 400 main clock cycles per line
constraints. This implies that such an implementation uses a faster clock than
required by this specification.




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
display standard from the appropriate data in the Peripheral RAM, line by
line.

A line renderer generates the display line from a display list providing
parameters for loading chunks of source data into an internal line double
buffer's render side. The buffers are flipped on advancing a line, so next
line the previous render side becomes a display side, and is used to generate
the video signal producing pixel data. ::


    +-------------------------+
    | Peripheral RAM          |
    +-------------------------+
                 |
                 | Peripheral bus access every second cycle
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

To give time slots for other components accessing the Peripheral RAM (on the
Peripheral bus) the Display generator is capable to access the bus on every
second main clock cycle, so allowing 200 Peripheral bus accesses per line.


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
the Peripheral RAM onto the Render side of the Line double buffer.

The processing is adequately pipelined so no Peripheral bus access cycles are
spent idle as long as there is data to render for the line. From the user's
point of view the Line renderer may be seen as fetching a display list
command, then processing it. Up to 8 bus access cycles per line or line pair
is however lost for overhead, so up to 192 bus access cycles remain available
for processing by this scheme (392 in double scanned mode).

(Implementations are allowed to deviate from the strictly sequential scheme in
favor of meeting the bus access cycle requirement by pipelining, such as by
pre-fetching some cells of the display list)


Double buffering assistance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Graphics Display Generator provides some assistance for implementing
double (or triple) buffering.

This is primarily realized through the Display List Definition register.
Writing this register latches the previous value, and puts the Graphics FIFO
in suspend mode until the end of the current display frame (the rendering
passes the last display line of the frame).

The write also initiates a Display list clear described below, which can be
used to clean the work buffer for the next render.


Display list clear function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Within vertical blanking the Graphics Display Generator is capable to clear
(by writing zeros) Peripheral RAM cells in the previously rendered Display
List.

This clearing only takes place in a VBlank after a Display List Definition
change, using the previous Display List Definition. For the clear the Graphics
Display Generator only uses the Peripheral bus cycles allocated to it, which
would otherwise be left unused, so it is free from the point of other
peripherals on the bus.

The range to clear is defined by the previous Display List Definition to
either 1600, 3200, 6400 or 12800 cells (depending on the Display List entry
/ line size). The Clear controls register defines which cells may be cleared,
and which may be preserved in this range.

Up to 9600 cells may be written in the clearing process (that is using 48
lines, 200 cycles each line). If by the Clear controls register more cells
would be necessary to be cleared, the clearing process terminates when 9600
cells are cleared (this means on the largest display list in 400 line graphics
modes up to 24 cells may be cleared each line).

Using the Clear controls it is possible to preserve parts of a Display list,
such as a constant background pattern.




Graphics Display Generator memory map and command layouts
------------------------------------------------------------------------------


The following table describes the Graphics Display Generator's registers. They
are accessible in the 0x0010 - 0x001F area in the User peripheral area.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | Mask / Colorkey definition 0                                      |
| 0x0010 |                                                                   |
|        | - bit  8-15: Mask / Colorkey for 0x8                              |
|        | - bit  0- 7: Mask / Colorkey for 0x9                              |
|        |                                                                   |
|        | Provides user-defineable Mask or Colorkey values for the given    |
|        | values of the Mask / Colorkey selector in the render command.     |
+--------+-------------------------------------------------------------------+
|        | Mask / Colorkey definition 1                                      |
| 0x0011 |                                                                   |
|        | - bit  8-15: Mask / Colorkey for 0xA                              |
|        | - bit  0- 7: Mask / Colorkey for 0xB                              |
|        |                                                                   |
|        | Provides user-defineable Mask or Colorkey values for the given    |
|        | values of the Mask / Colorkey selector in the render command.     |
+--------+-------------------------------------------------------------------+
|        | Mask / Colorkey definition 2                                      |
| 0x0012 |                                                                   |
|        | - bit  8-15: Mask / Colorkey for 0xC                              |
|        | - bit  0- 7: Mask / Colorkey for 0xD                              |
|        |                                                                   |
|        | Provides user-defineable Mask or Colorkey values for the given    |
|        | values of the Mask / Colorkey selector in the render command.     |
+--------+-------------------------------------------------------------------+
|        | Mask / Colorkey definition 3                                      |
| 0x0013 |                                                                   |
|        | - bit  8-15: Mask / Colorkey for 0xE                              |
|        | - bit  0- 7: Mask / Colorkey for 0xF                              |
|        |                                                                   |
|        | Provides user-defineable Mask or Colorkey values for the given    |
|        | values of the Mask / Colorkey selector in the render command.     |
+--------+-------------------------------------------------------------------+
|        | Shift mode region A                                               |
| 0x0014 |                                                                   |
|        | - bit    15: Unused, reads zero                                   |
|        | - bit  8-14: Output width in cells (0: No output)                 |
|        | - bit     7: Unused, reads zero                                   |
|        | - bit  0- 6: Begin position in cells                              |
|        |                                                                   |
|        | Specifies the region of output for Shift mode sources in Source   |
|        | definitions A0 - A3. The bus access cycles required are one more  |
|        | than the output width.                                            |
+--------+-------------------------------------------------------------------+
|        | Shift mode region B                                               |
| 0x0015 |                                                                   |
|        | - bit    15: Unused, reads zero                                   |
|        | - bit  8-14: Output width in cells (0: No output)                 |
|        | - bit     7: Unused, reads zero                                   |
|        | - bit  0- 6: Begin position in cells                              |
|        |                                                                   |
|        | Specifies the region of output for Shift mode sources in Source   |
|        | definitions B0 - B3. The bus access cycles required are one more  |
|        | than the output width.                                            |
+--------+-------------------------------------------------------------------+
|        | Display list clear controls                                       |
| 0x0016 |                                                                   |
|        | - bit 11-15: Initial cells to skip from clearing (0 - 31)         |
|        | - bit  6-10: Cells to skip after a streak (0 - 31)                |
|        | - bit  0- 5: Cells to clear in one streak (0 - 63)                |
|        |                                                                   |
|        | The display list clear begins at the Display list start offset    |
|        | found in the previous display list definition, then advances by   |
|        | the parameters provided in this register.                         |
+--------+-------------------------------------------------------------------+
|        | Display list definition & process flags                           |
| 0x0017 |                                                                   |
|        | - bit    15: Frame rate limiter flag                              |
|        | - bit    14: Display list clear is waiting or processing if set   |
|        | - bit 11-13: Unused, reads zero                                   |
|        | - bit  2-10: Display list start offset in 2048 PRAM cell units    |
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
|        |                                                                   |
|        | The Frame rate limiter flag becomes set when writing this         |
|        | register, and clears when the Graphics Display Generator fetches  |
|        | the new Display List Definition for rendering the next frame. It  |
|        | always clears after the Display list clear is waiting or          |
|        | processing flag.                                                  |
|        |                                                                   |
|        | The Display list clear is waiting or processing flag becomes set  |
|        | when writing this register, and clears as soon as the clearing    |
|        | process is completed or terminated.                               |
+--------+-------------------------------------------------------------------+
|        | Source definition A0                                              |
| 0x0018 |                                                                   |
|        | - bit 12-15: Base offset bits 12-15                               |
|        | - bit  8-11: PRAM bank select                                     |
|        | - bit  5- 7: Source line size (line select shift)                 |
|        | - bit     4: If set, shift source. If clear, positioned source.   |
|        | - bit  0- 3: Positioned source width multiplier                   |
|        |                                                                   |
|        | Source line sizes:                                                |
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
|        | The positioned source width multiplier specifies odd values from  |
|        | 1 to 31 (0 => 1; 15 => 31). It multiplies the Source line size,   |
|        | but has no effect on the Source line select in the render         |
|        | command.                                                          |
|        |                                                                   |
|        | Shift sources wrap around on their end when rendering, always     |
|        | producing the output width defined in the appropriate Shift mode  |
|        | region register.                                                  |
+--------+-------------------------------------------------------------------+
| 0x0019 | Source definition A1                                              |
+--------+-------------------------------------------------------------------+
| 0x001A | Source definition A2                                              |
+--------+-------------------------------------------------------------------+
| 0x001B | Source definition A3                                              |
+--------+-------------------------------------------------------------------+
| 0x001C | Source definition B0                                              |
+--------+-------------------------------------------------------------------+
| 0x001D | Source definition B1                                              |
+--------+-------------------------------------------------------------------+
| 0x001E | Source definition B2                                              |
+--------+-------------------------------------------------------------------+
| 0x001F | Source definition B3                                              |
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
| 28-30  | Source definition select                                          |
+--------+-------------------------------------------------------------------+
|        | Source line select. This is multiplied with the width of the      |
| 16-27  | source (not including the multiplier) to produce a PRAM offset,   |
|        | and is OR combined with the source's base offset. PRAM bank       |
|        | boundaries can not be crossed.                                    |
+--------+-------------------------------------------------------------------+
| 15     | Combine with mask if clear (!)                                    |
+--------+-------------------------------------------------------------------+
| 14     | Combine with colorkey if clear (!)                                |
+--------+-------------------------------------------------------------------+
|        | Mask / Colorkey value                                             |
| 10-13  |                                                                   |
|        | - 0x0: Mask / Colorkey is 00000000 (binary).                      |
|        | - 0x1: Mask / Colorkey is 11111111 (binary).                      |
|        | - 0x2: Mask / Colorkey is 00001111 (binary).                      |
|        | - 0x3: Mask / Colorkey is 00111111 (binary).                      |
|        | - 0x4: Mask / Colorkey is 00000011 (binary).                      |
|        | - 0x5: Mask / Colorkey is 00001100 (binary).                      |
|        | - 0x6: Mask / Colorkey is 00110000 (binary).                      |
|        | - 0x7: Mask / Colorkey is 11000000 (binary).                      |
|        | - 0x8: Mask / Colorkey is taken from bits 8 - 15 of 0x0010.       |
|        | - 0x9: Mask / Colorkey is taken from bits 0 - 7 of 0x0010.        |
|        | - 0xA: Mask / Colorkey is taken from bits 8 - 15 of 0x0011.       |
|        | - 0xB: Mask / Colorkey is taken from bits 0 - 7 of 0x0011.        |
|        | - 0xC: Mask / Colorkey is taken from bits 8 - 15 of 0x0012.       |
|        | - 0xD: Mask / Colorkey is taken from bits 0 - 7 of 0x0012.        |
|        | - 0xE: Mask / Colorkey is taken from bits 8 - 15 of 0x0013.       |
|        | - 0xF: Mask / Colorkey is taken from bits 0 - 7 of 0x0013.        |
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

Note that it is possible to combine with both Mask and Colorkey.

Note that Peripheral RAM bank boundaries can not even be crossed in position
mode with an appropriate source line select and a larger than one multiplier.
The reading of the source wraps around fetching the remaining cells from the
beginning of the same PRAM bank.




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
- The positioned source width count of cycles for sources in Position mode (0
  to 127 cycles).

Note that the renderer is not capable to optimize out access cycles which
would be used to render into off-screen area, neither it has a limit on how
many cycles may it consume for a command in Position mode.

(Pipelining notes: if a source is in Position mode, to render it on the Line
double buffer, one more cycle is necessary than it's width. This extra cycle
should be performed in parallel with a display list command fetch)




Other components of the Display Generator
------------------------------------------------------------------------------


Mode and double scanning
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The graphics mode (4 bit / 8 bit) and double scanning can be set using a
kernel call. See "0x0330: Change video mode" in "kcall.rst" for details.


Palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette can only be written through the "0x0300: Set palette entry" kernel
call. This component only affects the generated data, assigning the actual
visible colors to each pixel of the output stream. In real hardware it might
be a rather simple Digital Analog Converter (DAC).

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

The scale must be according to a gamma of 2.2, such as an interlacing pattern
of colors 0xFFF (white) and 0x000 (black) should produce approximately the
same luminance as color 0xBBB (grey).


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

- Fetching of the Graphics Display Generator registers relative to the render
  of lines or the frame.

- The timing of any display related Peripheral RAM access within the rendered
  line.

- The time the Display List clear function takes, provided it finishes within
  VBlank before starting the next display frame (including fetching the next
  frame's Display List Definition, and clearing the Frame rate limiter flag).

- After setting the palette data through the kernel call, it's effect may
  delay for up to "a few" frames, not even necessarily taking effect in
  Vertical Blank period. It must not affect any data rendered before the call.
  Note that the limit is loosely set to allow for software emulators using
  actual palettized displays, not necessarily being capable of synchronizing
  to the display hardware. These can't guarantee fast response if they also
  have to skip frames.




Graphics Display Generator timing
------------------------------------------------------------------------------


The Graphics Display Generator uses a fixed scheme for accessing the
Peripheral bus, generating an access every second cycle irrespective of it's
tasks.

The effect of these accesses from the point of minimal limits to support is
described in the "Memory access stalls" section of the CPU instruction set
("cpu_inst.rst").




Interlaced rendering
------------------------------------------------------------------------------


For interlaced standards an interlaced rendering mechanism has to be
supported. The key concepts behind it is that it should be as transparent for
the user as reasonably possible.

Since no state is remembered across lines, it is sufficient to simply slow the
line rendering process down to take 800 main clock cycles / line instead of
400, and increment the line counter by 2 after each line. Note that the access
cycles available for each line should still be constrained to the specified
limits, however it is not critical to realize an acceptable implementation.
