
RRPGE User Library - Fast scrolling tile map
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The fast scrolling tile mapper provides an automatically operated single
surface scrolling tile map using a tile map object (see tile map manager), a
destination surface, and an appropriately set-up Graphics Display Generator
with a double-buffered display list.

The fast scrolling tile map works with structures (objects) of the following
layout:

- Word0: Used tile map's pointer
- Word1: Used destination surface's pointer
- Word2: Visible height (number of rows used on the display list, 1 - 400)
- Word3: Base source offset (destination surface's partition base)
- Word4: Current top-left rendered X tile position
- Word5: Current top-left rendered Y tile position
- Word6: Full update request flag & Render command configuration
- Word7: Display list column to use (1 - 3 / 7 / 15 / 31)
- Word8: Display list Y start offset (0 - 399)
- Word9: Source offset range (destination surface's partition range)

The bits of Full update request flag & Render command configuration are as
follows:

- bit 13-15: Source definition select (Render command, bits 13-15)
- bit 10-12: High half-palette select (Render command, bits 10-12)
- bit  3- 9: Unused
- bit     2: Expect Y scroll only if set
- bit     1: Expect X scroll only if set
- bit     0: Full update request if set


Basic operation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The tile map is initialized with the us_tmap_getacc function, so always having
an origin of 0:0 on the destination. This ensures that any given tile
coordinate maps to the same destination coordinate, so when scrolling only the
newly scrolled in data has to be rendered.

The destination wraps around on all edges as the tile map is scrolled.

The renderer always keeps the following area of tiles rendered:

- Left X: Current top-left rendered X tile position (Word4)
- Top Y: Current top-left rendered Y tile position (Word5)
- Width in tiles: Visible width rounded up to tile boundary + 1 tile
- Height in tiles: Visible height rounded up to tile boundary + 1 tile

The current top-left rendered tile positions are calculated from the desired
new location (passed as parameters to us_fastmap_draw) by taking the whole
result after dividing them with the tile dimensions.

The Width in tiles is calculated without the 1 tile addition if the Expect Y
scroll only flag is set. The Height in tiles likewise is calculated without
the 1 tile addition if the Expect X scroll only flag is set.

If either the Visible width or Visible height is zero, nothing is rendered.

For the proper operation, the Graphics Display Generator also has to be set
up, display lists with the double buffering manager (us_dbuf_init).

For the Graphics Display Generator, the appropriate Source definition
register has to be set up to cover the destination proper. It must be a
shift source (so destination width must be a matching power of 2). The related
Shift mode region register also should be set up to properly constrain the
width of the scrolling area if necessary (this determines the Visible width).

The display lists are filled according to the requested new location (passed
as parameters to us_fastmap_draw). The X position is in pixels (a cell is 8
pixels wide). The fill is performed so the appropriate region of the
destination is shown, as if on the tile map, the visible area's upper left
corner was at the desired X:Y position.


Limitations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Adhering the following limitations are mandatory. Not following these the
behavior is undefined (producing no display may be a typical result):

- Visible height (Word2) must be at most 400.

- Display list Y start offset (Word8) must be between 0 and 399.

- Visible height and Display list Y start offset added together must be less
  or equal than 400.

- Display list column to use must be a valid non-background (nonzero) column
  on the target display list.

The following limitations should be adhered to produce the intended result
(functional scrolling tile map):

- The destination surface must be single buffered, and must have power of 2
  dimensions.

- The destination surface must be large enough so the area of tiles defined in
  Basic operation fits into it.

- The Source definition covering the destination surface must be set up
  according to the destination's dimensions.

- The used tileset's tile width must be a power of 2 which is a divisor of the
  destination surface's width.

- The used tiles must be opaque (otherwise older tiles on the destination will
  show through as the tile map scrolls).

- The desired X:Y positions (parameters of us_fastmap_draw) should not be
  wrapped around. If an infinitely scrolling tile map is necessary, they
  should be taken modulo by the tile map's total size in pixels (causing a
  full update when wrapping this way) to achieve a clean result. Tile maps
  having power of 2 pixel dimensions always wrap around cleanly (with a
  probable full update on the wrap).

Additional notes:

- The tileset's Y dimension has no restrictions relative to the destination
  height as long as the destination's dimensions meet the requirements.

- The tile map may have arbitrary dimensions, even smaller than those of the
  targeted visible area.

- There is no limitation on the number of fast scrolling tile maps present on
  a display.




Functions
------------------------------------------------------------------------------


0xE0CA: Set up fast scrolling tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_new
- Cycles: 300 + Wait for frame end
- Param0: Target fast scrolling tile map pointer (10 words)
- Param1: Tile map to use (tile map structure pointer)
- Param2: Destination surface to use (destination surface pointer)
- Param3: Display list column to use
- Param4: Display list start Y location
- Param5: Display list count of used rows
- Param6: Render command configuration & X / Y scroll expectations

Sets up a fast scrolling tile map using the given parameters. The parameters
are used as-is, except for the full update request flag (one bit), which is
set.

Note that source offset parameters are taken from the specified destination
surface, which may cause the function waiting for frame end.


0xE0CC: Mark fast scrolling tile map for update
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_mark
- Cycles: 25
- Param0: Target fast scrolling tile map pointer (10 words)

Marks the fast scrolling tile map for full update, which will happen next time
when us_fastmap_draw is called.


0xE0CE: Get visible dimensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_gethw
- Cycles: 200 + Tileset Height:Width request function call
- Param0: Fast scrolling tile map pointer (10 words)
- Ret. C: Visible height in tiles
- Ret.X3: Visible width in tiles

Requests the visible dimensions of the fast scrolling tile map. These are
constant as long as the count of used rows on the display list, and the
appropriate Graphics Display Generator shift mode region registers are
unchanged. The returned dimensions are calculated according to the description
at Basic operation.


0xE0D0: Get current tile top-left coordinates
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_getyx
- Cycles: 30
- Param0: Fast scrolling tile map pointer (10 words)
- Ret. C: Tile Y position of topmost displayed tile
- Ret.X3: Tile X position of leftmost displayed tile

Requests the current top-left tile coordinate pair. Along with
us_fastmap_gethw this function can be used to request the area currently
visible, which could be used to perform specific tile updates on it (instead
of requesting a full update).

Note that for animating tiles, the tileset's blitter should be designed to
perform in one pass, otherwise the animation may flicker (the surface is
single buffered).


0xE0D2: Set display list Y parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_setdly
- Cycles: 170 + Tileset Height:Width request function call
- Param0: Fast scrolling tile map pointer (10 words)
- Param1: New display list start Y location
- Param2: New display list count of used rows

Changes the display list's output parameters, so the scrolling surface
appears elsewhere on Y. It may cause a full update (setting the full update
request flag) if the new count of used rows grows larger so it needs more
tiles to be rendered than before.


0xE0D4: Render the fast scrolling tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_fastmap_draw
- Cycles: Depends on what has to be rendered
- Param0: Fast scrolling tile map pointer (10 words)
- Param1: New left X position (in pixels)
- Param2: New top Y position (in rows)

Scrolls to the requested new position, and ensures that the visible area is
properly filled by the appropriate section of the tile map, as described in
Basic operation.

There may be at about 1000 cycles of overhead excluding a call to the tile
set's accelerator setup and height:width request functions.

Unless the full update request flag is set, tiles are rendered only to cover
the area difference between the old and new locations. The tile rendering is
done using the us_tmap_blit function. The full update request flag is always
cleared after a render.

The selected display list rows (Visible height; display list used rows) are
filled at a 20 cycle per row rate.

The render waits for frame end as necessary (using us_dbuf_getlist).

PRAM pointers 2 and 3 are used and not preserved.




Entry point table of Fast scrolling tile map functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- F: Additional callback cycles.
- S: For cycle counts see function's description.
- W: May wait for a specific event.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE0CA |       300 + W | 7 |      | us_fastmap_new                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE0CC |            25 | 1 |      | us_fastmap_mark                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0CE |       200 + F | 1 | C:X3 | us_fastmap_gethw                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE0D0 |            30 | 1 | C:X3 | us_fastmap_getyx                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE0D2 |       170 + F | 3 |      | us_fastmap_setdly                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0D4 |             S | 3 |      | us_fastmap_draw                        |
+--------+---------------+---+------+----------------------------------------+
