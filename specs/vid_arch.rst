
Graphics Display Generator architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


This part of the specification defines the Graphics Display Generator
component of the RRPGE system. This component is responsible for generating
the signals producing the visual image on the display hardware, as required by
the targeted display standard.




Basic properties of the display
------------------------------------------------------------------------------


The basic properties of the RRPGE system's display generator are as follows:

- 640x400 visible pixels of 1:1 pixel aspect ratio
- 6 bits per pixel palettized, 64 palette entries of 4-4-4 RGB colors.
- 16:10 display aspect ratio
- 50Hz minimal / 60Hz maximal refresh rate
- At least 71 vertical blank lines (for a total of at least 471 lines)
- At least 400 main clock cycles per display line
- Interlaced display generation is supported for standards requiring it
- Accesses the 32 bit Peripheral bus

The graphic display normally has to be clocked by twice the frequency of the
main clock (except if it only supports interlaced modes).

Interlaced modes are implemented in a transparent way: they require the same
programming from the user like non-interlaced modes. The only exception is
that the user needs to be aware of that it might need two frames to produce a
complete image. See the "Interlaced rendering" chapter for more information.

Note that faster rates than 60Hz may be provided by an implementation if it
meets the at least 471 line, and at least 400 main clock cycles per line
constraints. This implies that such an implementation uses a faster clock than
required by this specification.




Display standards and RRPGE system compatibility
------------------------------------------------------------------------------


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
    |  +- 80 x 48 bit cells display data ------+- 48 x 48 bit cells void -+  |
    |  |                                       |                          |  |
    |  +------------- Display side ------------+--------------------------+  |
    |            |                                                           |
    +------------|-----------------------------------------------------------+
                 |
                 |     +--------------------------+
                 |<----| Palette data (64 colors) |
                 |     +--------------------------+
                 V
           Video signal


A display line is 640 x 6 bit pixels. One buffer in the Line double buffer
accordingly is capable to hold 80 x 48 bits of data (8 pixels per cell the
same manner like 4 bit pixels are represented in the PRAM's 32 bit cells),
while it's cells may have a 7 bit address. The extra cells addressable with
these address bits (cells 80 - 127) do not contribute to the Video signal, and
so they may not be implemented.

The Line double buffer cells are accessed in pairs, so it is effectively
addressed using a 6 bit address.

The filling of the render side starts in line -2 (2 lines before the first
display line), then a buffer flip happens on advancing to line 0. Subsequently
buffer flips happen either every second line or every line depending on the
double scanning setting, render accesses to the PRAM happening between line -2
and line 398 inclusive (401 lines).

The Render side also contains a reset circuity which can reset the state of
all cells to a given initial value (background pattern) in a single clock.

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
command, then processing it. Up to 16 bus access cycles per line or line pair
is however lost for overhead and PRAM refresh, so up to 184 bus access cycles
remain available for processing by this scheme (368 in double scanned mode).

(Implementations are allowed to deviate from the strictly sequential scheme in
favor of meeting the bus access cycle requirement by pipelining, such as by
pre-fetching some cells of the display list)


Double buffering assistance
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Graphics Display Generator provides some assistance for implementing
double (or triple) buffering.

This is primarily realized through the Display List Definition register.
When writing this register, it's previous value remains latched in an internal
register used for completing the current frame, and the Graphics FIFO is put
in suspend mode until the end of the current display frame (the rendering
passes the last display line of the frame).

The write also initiates a Display list clear described below, which can be
used to clean the work buffer for the next render.


Display list clear function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Within vertical blanking the Graphics Display Generator is capable to clear
(by writing zeros) Peripheral RAM cells in the previously rendered Display
List.

This clearing (roughly) only takes place in a VBlank after a Display List
Definition change, using the previous Display List Definition. For the clear
the Graphics Display Generator only uses the Peripheral bus cycles allocated
to it, which would otherwise be left unused, so it is free from the point of
other peripherals on the bus.

The range to clear is defined by the previous Display List Definition to
either 1600, 3200, 6400 or 12800 cells (depending on the Display List entry
/ line size). The Clear controls register defines which cells may be cleared,
and which may be preserved in this range.

By the above up to 12800 cells may be written in the clearing process,
requiring 70 lines if 184 cycles are given to the clearing process each line.
This produces the 71 line minimum requirement for the VBlank interval.

Using the Clear controls it is possible to preserve parts of a Display list,
such as a constant background pattern.

Note that the Display list clear function can not pass PRAM bank boundaries,
addressing will wrap around in such situations to the beginning of the bank.




Graphics Display Generator memory map and command layouts
------------------------------------------------------------------------------


The following table describes the Graphics Display Generator's registers. They
are accessible in the 0x0010 - 0x001F area in the User peripheral area.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | Colorkey values A                                                 |
| 0x0010 |                                                                   |
|        | - bit 12-15: Colorkey value for source A0                         |
|        | - bit  8-11: Colorkey value for source A1                         |
|        | - bit  4- 7: Colorkey value for source A2                         |
|        | - bit  0- 3: Colorkey value for source A3                         |
+--------+-------------------------------------------------------------------+
|        | Colorkey values B                                                 |
| 0x0011 |                                                                   |
|        | - bit 12-15: Colorkey value for source B0                         |
|        | - bit  8-11: Colorkey value for source B1                         |
|        | - bit  4- 7: Colorkey value for source B2                         |
|        | - bit  0- 3: Colorkey value for source B3                         |
+--------+-------------------------------------------------------------------+
|        | Double scan split                                                 |
| 0x0012 |                                                                   |
|        | - bit    15: Unused, reads zero                                   |
|        | - bit 12-14: High half-palette select for Background pattern      |
|        | - bit    11: Unused, reads zero                                   |
|        | - bit  8-10: Low half-palette select for Background pattern       |
|        | - bit  0- 7: Double scan split location                           |
|        |                                                                   |
|        | Defines how many double-scanned lines should appear on the top    |
|        | half of the display. Effective between 0 and 200, the first       |
|        | making the entire display single-scanned, the latter double-      |
|        | scanned. Note that the first single-scanned line always has two   |
|        | lines worth of cycles to render (392 cycles).                     |
+--------+-------------------------------------------------------------------+
|        | Display list clear controls                                       |
| 0x0013 |                                                                   |
|        | - bit 11-15: Initial cells to skip from clearing (0 - 31)         |
|        | - bit  6-10: Cells to skip after a streak (0 - 31)                |
|        | - bit  0- 5: Cells to clear in one streak (0 - 63)                |
|        |                                                                   |
|        | The display list clear begins at the Display list start offset    |
|        | found in the previous display list definition, then advances by   |
|        | the parameters provided in this register.                         |
+--------+-------------------------------------------------------------------+
|        | Shift mode region A                                               |
| 0x0014 |                                                                   |
|        | - bit    15: Clip positioned source A3 to region if set           |
|        | - bit    14: Clip positioned source A2 to region if set           |
|        | - bit  8-13: Output width in cell pairs (0: No output)            |
|        | - bit     7: Clip positioned source A1 to region if set           |
|        | - bit     6: Clip positioned source A0 to region if set           |
|        | - bit  0- 5: Begin position in cell pairs                         |
|        |                                                                   |
|        | Specifies the region of output for Shift mode sources in Source   |
|        | definitions A0 - A3. The bus access cycles required are one more  |
|        | than the output width. Positioned sources may also be clipped to  |
|        | this region (note: positioned sources always use display column   |
|        | 0 as base irrespective of this setting).                          |
+--------+-------------------------------------------------------------------+
|        | Shift mode region B                                               |
| 0x0015 |                                                                   |
|        | - bit    15: Clip positioned source B3 to region if set           |
|        | - bit    14: Clip positioned source B2 to region if set           |
|        | - bit  8-13: Output width in cell pairs (0: No output)            |
|        | - bit     7: Clip positioned source B1 to region if set           |
|        | - bit     6: Clip positioned source B0 to region if set           |
|        | - bit  0- 5: Begin position in cell pairs                         |
+--------+-------------------------------------------------------------------+
|        | Display list definition                                           |
| 0x0016 |                                                                   |
|        | - bit 12-15: Display list PRAM bank                               |
|        | - bit  2-11: Display list start offset high bits (6-15)           |
|        | - bit  0- 1: Display list line size                               |
|        |                                                                   |
|        | Display list line sizes:                                          |
|        |                                                                   |
|        | - 0: 4 entries (cells)                                            |
|        | - 1: 8 entries (cells)                                            |
|        | - 2: 16 entries (cells)                                           |
|        | - 3: 32 entries (cells)                                           |
|        |                                                                   |
|        | The effective portion of a display list depends on the location   |
|        | of the double scan split, requiring between 200 and 400 lines     |
|        | defined. Note that the display list clear function doesn't        |
|        | consider this split location, always aiming to clear 400 lines    |
|        | worth of display list.                                            |
|        |                                                                   |
|        | The newly written Display list definition does not affect the     |
|        | currently displayed frame (the previous value is latched          |
|        | internally), however the new value will show on reading this      |
|        | register.                                                         |
|        |                                                                   |
|        | Display lists can not cross PRAM bank boundaries. The address     |
|        | will wrap to the beginning of the bank.                           |
+--------+-------------------------------------------------------------------+
|        | Status flags                                                      |
| 0x0017 |                                                                   |
|        | - bit    15: Frame completion flag (read only)                    |
|        | - bit  0-14: Unused, reads zero                                   |
|        |                                                                   |
|        | The Frame completion flag becomes set when writing the Display    |
|        | list definition register, and clears when the Graphics Display    |
|        | Generator fetches the new Display List Definition for rendering   |
|        | the next frame and the Display list clear also completed.         |
+--------+-------------------------------------------------------------------+
|        | Source definition A0                                              |
| 0x0018 |                                                                   |
|        | - bit 12-15: PRAM bank select                                     |
|        | - bit    11: X expansion if set                                   |
|        | - bit  8-10: Low half-palette select                              |
|        | - bit     7: If set, shift source. If clear, positioned source.   |
|        | - bit     6: If set, enables tiled mode (disables bit 7)          |
|        | - bit  0- 5: Source line size in cell pairs (0: 64 cell pairs)    |
|        |                                                                   |
|        | Shift sources use the source line size field differently, only    |
|        | the low 3 bits:                                                   |
|        |                                                                   |
|        | - 0: 1 cell pair (16 pixels)                                      |
|        | - 1: 2 cell pairs                                                 |
|        | - 2: 4 cell pairs                                                 |
|        | - 3: 8 cell pairs                                                 |
|        | - 4: 16 cell pairs                                                |
|        | - 5: 32 cell pairs                                                |
|        | - 6: 64 cell pairs                                                |
|        | - 7: 128 cell pairs                                               |
|        |                                                                   |
|        | Shift sources wrap around on their end when rendering, always     |
|        | producing the output width defined in the appropriate Shift mode  |
|        | region register.                                                  |
|        |                                                                   |
|        | The half-palette bits produce bits 3-5 of the resulting pixel in  |
|        | the Line double buffer when rendering the source. The values in   |
|        | this register apply if the source pixel's bit 3 is clear (so      |
|        | selecting palette for color indices 0 - 7).                       |
|        |                                                                   |
|        | In X expanded mode every source pixel will expand to two          |
|        | destination pixels, doubling the width of the source (both for    |
|        | positioned and shift sources)                                     |
|        |                                                                   |
|        | A tiled mode is provided if bit 6 is set. Bit 7 is ignored this   |
|        | case, leaving the tiled mode acting like a positioned source.     |
|        | This mode is described further below ("Tiled mode").              |
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
entries (up to 3, 7, 15, 31 depending on line size) are render commands.

The layout of a render command is as follows:

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
|        | Start offset within PRAM bank. PRAM bank boundaries can not be    |
| 16-31  | crossed. Shift sources ignore the lower bits depending on the     |
|        | specified line size (for example 128 cells width ignores the low  |
|        | 7 bits).                                                          |
+--------+-------------------------------------------------------------------+
| 13-15  | Source definition select                                          |
+--------+-------------------------------------------------------------------+
| 4-12   | Mode specific bits                                                |
+--------+-------------------------------------------------------------------+
| 0-3    | Cell pair right shift amount (0 - 15 pixels)                      |
+--------+-------------------------------------------------------------------+

The mode specific bits in Shift mode:

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
|        | High half-palette select. If this field is zero, the render       |
| 10-12  | command is disabled. Otherwise it specifies bits 3 - 5 of the     |
|        | resulting pixel value in the Line buffer if source pixel bit 3    |
|        | was set (so effectively selects palette for indices 8 - 15).      |
+--------+-------------------------------------------------------------------+
|        | Negated source start offset in cell pairs. Offset is generated as |
| 4-9    | (this_value ^ 0x3F), shifted left by one if X expansion is off.   |
|        | Note that the source fetches for the first cell pair don't        |
|        | directly generate output, only populate the output shift          |
|        | register. This way, combined with bits 0 - 3 of the render        |
|        | command, this forms a source start offset in pixels on the        |
|        | display.                                                          |
+--------+-------------------------------------------------------------------+

The mode specific bits in Positioned mode:

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
|        | High half-palette select. If this field is zero, the render       |
| 10-12  | command is disabled. Otherwise it specifies bits 3 - 5 of the     |
|        | resulting pixel value in the Line buffer if source pixel bit 3    |
|        | was set (so effectively selects palette for indices 8 - 15).      |
+--------+-------------------------------------------------------------------+
| 4-9    | Start offset on display (in cell pairs). Combined with bits 0 - 3 |
|        | of the render command, this is essentially a pixel position.      |
+--------+-------------------------------------------------------------------+

The mode specific bits in Tiled mode:

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
| 12     | If set, enables pseudo 6 bit mode.                                |
+--------+-------------------------------------------------------------------+
|        | PRAM bank select for tiles in pseudo 6 bit mode. This field is    |
| 8-11   | unused otherwise, but this case either bit 10 or 11 should be set |
|        | to enable the render command (so bits 10 - 12 remain nonzero).    |
+--------+-------------------------------------------------------------------+
|        | Tile row select. In X expanded mode this is XOR combined with     |
| 4-7    | bits 0 - 3 of the tile address, otherwise it is XOR combined with |
|        | bits 1 - 4.                                                       |
+--------+-------------------------------------------------------------------+

A render command may be disabled by leaving its bits 10 - 12 zero. Such a
render command does not contribute to the line's contents, and only takes one
bus access cycle (the cycle in which it was fetched).

PRAM boundaries can not be crossed by source fetches: the addressing wraps
around to the beginning of the given PRAM bank on such event.

Note that if the source line size field is set to 128 cell pairs for Shift
mode, infinite scrolling using bits 0 - 9 of the render command becomes
impossible. This setup however remains useful for panning over a 1664 pixels
wide surface (2048 pixels effective width, but 384 pixels of this can not be
made visible).




Rendering process
------------------------------------------------------------------------------


The rendering process for cells are identical for Shift and Position modes,
and is carried out according to the following simplified guide: ::


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
              |                                           Colorkey value
              V                                                  |
    +----+----+----+----+               +----+----+----+----+    |
    | Data to blit (32) |-------------->|   Colorkey mask   |<---+
    +----+----+----+----+               +----+----+----+----+
              |                                   |
              |                                   |      +----+----+----+----+
              |   +---- Half-palette selects      | +----|  Beg/Mid/End mask |
              |   |                               | |    +----+----+----+----+
              V   V                               | |
    +--+--+--+--+--+--+--+--+                     | | +--- Clip mask (0 / 1)
    |  48 bit output data   |                     | | |
    +--+--+--+--+--+--+--+--+                    _V_V_V_
                |                               |  AND  |
                |                                ~~~|~~~
                |                                   | Pixel-level mask
               _V_                                  | (8 x 6 bit pixels)
              |AND|<--------------------------------+
               ~|~                                  |
               _V_     ___                         _V_
              | OR|<--|AND|<----------------------|NEG|
               ~|~     ~A~                         ~~~
                |       |
                V       |
     ---+--+--+--+--+--+--+--+--+---
        | Target line buf. cell |
     ---+--+--+--+--+--+--+--+--+---


The Beg/Mid/End mask is used in Position mode to mask the partially filled
cells on the beginning and the end of the rendered streak of data.

The Clip mask is sourced from the Shift mode region registers in Position mode
as needed, generated to indicate whether the target line buffer cell can be
filled or not.

In normal Positioned or Shift modes (no X expansion), 2 such source fetches
are performed to populate the two target Line buffer cells.

If X expansion is set, only one source fetch is performed, which is expanded
(from 32 bits to 64 bits by duplicating each 4 bit pixel) before writing into
the Shift register in two passes of the above algorithm. This way half as many
source fetches are performed than in no X expansion modes, thus halving the
total width of the source data.

If X expansion is clear, two source fetches are performed. The address for the
second is generated OR combining one onto the source offset. This produces no
differences in Shift mode, however in Positioned mode it affects how odd start
offsets are handled.

The simplification in the above chart means that it describes a single cell
operation. To operate with cell pairs as required, the shift register has to
be twice as wide (128 bits).

The half-palette selection is performed according to the following scheme
(both for the background pattern and normal renders): ::


    +----+----+----+----+
    |   Source pixel    | One 4 bit pixel from a 32 bit cell
    +----+----+----+----+
      |         |
      |         +-----------------------------------------------------+
      |                                                               |
      +----+----+                                                     |
      V    V    V                                                     V
    +----+----+----+             ___                          +----+----+----+
    | b3 | b3 | b3 |------+---->|NEG|                         | b2 | b1 | b0 |
    +----+----+----+      |      ~|~                          +----+----+----+
                          |       |                                   |
    +----+----+----+     _V_     _V_     +----+----+----+             |
    | High h. pal. |--->|AND|   |AND|<---|  Low h. pal. |             |
    +----+----+----+     ~|~     ~|~     +----+----+----+             |
                          |       |                                   |
                          |      _V_                                  |
                          +---->| OR|             +-------------------+
                                 ~|~              |
                                  |               |
                                  V               V
                           +----+----+----+----+----+----+
                           |     6 bit output pixel      |
                           +----+----+----+----+----+----+




Tiled mode
------------------------------------------------------------------------------


Sources can be configured to generate a tiled mode. In this mode a tile map
(tile descriptor source) is used to retrieve source locations. The
half-palettes can be set for each individual tile, thus supporting using more
than 16 colors on a tiled mode surface. Alternatively, tile mode can be used
to produce a pseudo 6 bit mode, which combined with X expansion and a tile
height of one, is capable to assign (almost) any of the 64 palette colors to
every pixel.

It basically uses the Positioned source logic for display, with the addition
of tile descriptor source fetches for each cell pair (so a tile is always two
cells wide). Actual source fetches are performed after producing the source
offset from the tile descriptor source. The tile descriptor source is read in
the same manner as a normal X expanded source would.

The tile descriptor source defines the address and palettes for the tiles. Its
low 16 bits provide the tile address, which is XOR combined with the Tile row
select to generate the actual offset. If X expansion is off, the source is 2
cells wide. The second cell this case is fetched by setting the lowest bit of
the address to 1 (so if the address was odd, the same cell will be fetched).

The high 16 bits are as follows in normal mode (pseudo 6 bit off):

+--------+-------------------------------------------------------------------+
| Bits   | Description                                                       |
+========+===================================================================+
|    31  | Unused                                                            |
+--------+-------------------------------------------------------------------+
| 28-30  | High half-palette select                                          |
+--------+-------------------------------------------------------------------+
|    27  | Unused                                                            |
+--------+-------------------------------------------------------------------+
| 24-26  | Low half-palette select                                           |
+--------+-------------------------------------------------------------------+
| 20-23  | Colorkey value                                                    |
+--------+-------------------------------------------------------------------+
| 16-19  | PRAM bank select for the tile                                     |
+--------+-------------------------------------------------------------------+

The high 16 bits in Pseudo 6 bit mode provide the two high palette index bits
for 8 pixel pairs, bits 30 and 31 for the leftmost pixel's bit 4 and 5
respectively. If X expansion is set, this layout corresponds to that of the
pixels, thus allowing supplying a 6 bit color for every "wide" pixel.

Note that in Pseudo 6 bit mode the Colorkey matching is still done on the low
4 bits only, so essentially four 6 bit indices will produce a transparent
pixel, thus only allowing selecting 60 colors (plus one if laid over a plain
background).

A probable way of building a pseudo 6 bit 320 x 200 display is using double
scanning (setting Double Scan Split to 200), then using a tile height of one
row (so every row has its own tile source, allowing to specify the high two
bits for each individual pixel). If the same time the tile offsets are laid
out horizontally incrementing, the resulting 4 bit portion of the display
surface will allow using the Accelerator normally on it.

Using Tiled mode normally adds one cycle for processing each cell pair
compared to the cycle budget of Positioned sources (due to the additional
tile source fetch from the PRAM). In X expanded mode however the extra PRAM
accesses replace the access which would have to be done to fetch two source
cells, so this case Tiled mode is "free" on term of cycles consumed.




Renderer cycle budget
------------------------------------------------------------------------------


As defined in the "Display List" chapter, in single scanned mode from the
user's point of view there are at least 184 useful Video bus access cycles,
and in doubly scanned mode, there are 368.

The rendering from the user's point of view may be interpreted as being
sequential: the renderer fetches a display list command, then processes it,
then goes on to the next command as long as there are commands for the line
and there are bus access cycles remaining for the render.

Bus access cycles are taken by the following rules:

- 1 cycle for reading a render command (including the background pattern).
- In Shift mode, twice the Output width of cycles, plus two for the initial
  source fetch.
- In Positioned mode, twice the Source line size of cycles, plus one for an
  extra Line buffer write (actually two, but one of these cycles should be
  pipelined with the reading of a subsequent render command).
- In Tiled mode with X expansion off, three times the Source line size of
  cycles, plus one for an extra Line buffer write (see above).
- In Tiled mode with X expansion, the same as in Positioned mode.

Note that the renderer is not capable to optimize out access cycles which
would be used to render into non-displayed area (off-screen or clipped).




Other components of the Display Generator
------------------------------------------------------------------------------


Palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette can only be written through the "0x08: Set palette entry" kernel
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


PRAM refresh
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Graphics Display Generator should also provide Peripheral RAM refresh
logic to handle Dynamic RAM chips. These are taken from the 16 overhead cycles
allowed in each line (including within VBLANK).


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
  frame's Display List Definition, and clearing the Frame completion flag).

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
