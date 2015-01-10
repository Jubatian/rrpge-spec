
RRPGE User Library - Tile map manager
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The tile map manager provides basic support for working with Graphics
Accelerator based tile maps. Combined with a Destination surface and a Tileset
it can be used to produce graphics directly.

The tile map manager works with tile map structures (objects). The structure
is formed as follows:

- Word0: Used tileset's pointer
- Word1: Width of tile map in tiles
- Word2: Height of tile map in tiles
- Word3: Word offset of tile map start in PRAM, high
- Word4: Word offset of tile map start in PRAM, low
- Word5: Blit function of tileset
- Word6: Acceletator init function of tileset
- Word7: Height:Width request function of tileset

For the functions, normally the us_tile_blit, us_tile_getacc and us_tile_gethw
functions may be used from the tileset manager, however through those, using
an identical interface, user-created tileset managers may also be used with
the tile map manager. A blit function does not necessarily have to have a
destination start offset fraction parameter, however then the
us_tmap_getaccxfy function's X fraction parameter will have no effect.

CPU RAM locations are used for supporting blitting (set by the us_tmap_getacc
and derivative functions):

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFAA0 | Width of tile map in tiles (memorized Word1).                     |
+--------+-------------------------------------------------------------------+
| 0xFAA1 | Height of tile map in tiles (memorized Word2).                    |
+--------+-------------------------------------------------------------------+
| 0xFAA2 | Word offset of tile map start in PRAM, high (memorized Word3).    |
+--------+-------------------------------------------------------------------+
| 0xFAA3 | Word offset of tile map start in PRAM, low (memorized Word4).     |
+--------+-------------------------------------------------------------------+
| 0xFAA4 | Blit function of tileset (memorized Word5).                       |
+--------+-------------------------------------------------------------------+
| 0xFAA5 | Destination width (cells).                                        |
+--------+-------------------------------------------------------------------+
| 0xFAA6 | Destination height.                                               |
+--------+-------------------------------------------------------------------+
| 0xFAA7 | Tile width (cells).                                               |
+--------+-------------------------------------------------------------------+
| 0xFAA8 | Tile height.                                                      |
+--------+-------------------------------------------------------------------+
| 0xFAA9 | X origin fraction.                                                |
+--------+-------------------------------------------------------------------+
| 0xFAAA | X origin.                                                         |
+--------+-------------------------------------------------------------------+
| 0xFAAB | Y origin.                                                         |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xF0B2: Set up tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_set
- Cycles: 120
- Param0: Target tile map pointer (8 words)
- Param1: Tileset to use (tileset structure pointer)
- Param2: Width of tile map in tiles
- Param3: Height of tile map in tiles
- Param4: Word offset of tile map in PRAM, high
- Param5: Word offset of tile map in PRAM, low

Sets up a tile map structure using the given parameters as-is. For the
functions, us_tile_blit, us_tile_getacc and us_tile_gethw are used.


0xF0B4: Set up tile map with tileset functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_set
- Cycles: 130
- Param0: Target tile map pointer (8 words)
- Param1: Tileset to use (tileset structure pointer)
- Param2: Width of tile map in tiles
- Param3: Height of tile map in tiles
- Param4: Word offset of tile map in PRAM, high
- Param5: Word offset of tile map in PRAM, low
- Param6: Tileset blit function
- Param7: Tileset accelerator init function
- Param8: Tileset Height:Width request function

Sets up a tile map structure using the given parameters as-is.


0xF0B6: Accelerator setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_getacc
- Cycles: 400 + Wait for frame end + Function calls
- Param0: Source tile map pointer
- Param1: Destination surface pointer

Sets up for tile map blitting, using the given tile map and destination
surface. The origin coordinates are set zero.

This function calls the Tileset acceleator init function and the Tileset
Height:Width request function to generate contents for the appropriate fields
in the CPU RAM.

The destination height is calculated from the result of us_dsurf_getpw. If the
width is a power of 2 smaller than the partition size, the height gets the
expectable total cells divided by width. Otherwise the height is the largest
even number which multiplied with width produces a smaller or equal result as
the total cell count of the destination.


0xF0B8: Accelerator setup with boundary origin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_getaccxy
- Cycles: 410 + Wait for frame end + Function calls
- Param0: Source tile map pointer
- Param1: Destination surface pointer
- Param2: X origin (cells)
- Param3: Y origin

Works the same way like us_tmap_getacc, but with setting up an origin.

The origin can be used to scroll the tile map on the destination. This
function does not provide a fraction for X, so enforcing block boundary (which
is faster). It may be used for vertical scrollers, or scrollers combined with
Display list manipulation where full updates are necessary.


0xF0BA: Accelerator setup with origin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_getaccxfy
- Cycles: 410 + Wait for frame end + Function calls
- Param0: Source tile map pointer
- Param1: Destination surface pointer
- Param2: X origin (cells)
- Param3: X origin (fraction)
- Param4: Y origin

Works the same way like us_tmap_getacc, but with setting up an origin.

The origin can be used to scroll the tile map on the destination. Note that
using fraction is slower (due to that the Accelerator has to write more
destination cells this case to blit a tile), however this may be used for
effects like parallax scrolling.


0xF0BC: Blit tile map region
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_blit
- Cycles: 400 + 60 / tile + Tile blit function calls
- Param0: Top-left tile X
- Param1: Top-left tile Y
- Param2: Width in tiles
- Param3: Height in tiles

Outputs a tile map region on the destination, supporting both destination and
tile map wrapping.

Note that row traversing incurs additional overhead cycles, included for the
first row in the overall overhead. Single column regions are optimized to
eliminate row traversing overhead.

The target position on the destination surface is calculated as follows: ::

    XPos = (TileWidth  * TileXPos + XOrigin) % DestWidth
    YPos = (TileHeight * TileYPos + YOrigin) % DestHeight

The XPos (X position on destination) is calculated in cell units. If an X
Origin fraction is set up, it is only applied to the Tile blit function,
essentially only shifting the tile map towards the right.

Note that no boundary checkings are done, the offset translation is performed
as written, so if the destination width is not a multiple of the tile width,
or the destination size does not match DestWidth * DestHeight, appropriate
artifacts will show. These should be anticipated when designing tile map
related algorithms, such as by using power of 2 dimensions for an infinite
scroller.

When either position wraps around within the region, unless the appropriate
destination dimension is a power of 2, the resulting positions are undefined.

The tile map positions are calculated as follows: ::

    TileMapX = TileXPos % TileMapWidth
    TileMapY = TileXPos % TileMapHeight

When either position wraps around within the region, unless the appropriate
tile map dimension is a power of 2, the resulting tiles to blit are undefined.

Uses PRAM pointer 3, which is not preserved.


0xF0BE: Get height and width of tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_gethw
- Cycles: 40
- Param0: Tile map pointer
- Ret. C: Height in tiles
- Ret.X3: Width in tiles

Returns the width and height of the tile map.


0xF0C0: Get height and width of tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_gettilehw
- Cycles: 25 + Tileset Height:Width request function call
- Param0: Tile map pointer
- Ret. C: Height in rows
- Ret.X3: Width in cells

Returns the width and height of the tileset used by the tile map.


0xF0C2: Get tile index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_gettile
- Cycles: 170
- Param0: Tile map pointer
- Param1: Tile X
- Param2: Tile Y
- Ret.X3: Tile index

Reads a tile index value from the tile map. The tile X and Y coordinates are
taken modulo the appropriate tile map dimensions.

Uses PRAM pointer 3, which is not preserved.


0xF0C4: Set tile index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_settile
- Cycles: 180
- Param0: Tile map pointer
- Param1: Tile X
- Param2: Tile Y
- Param3: Tile index

Sets a tile index value on the tile map. The tile X and Y coordinates are
taken modulo the appropriate tile map dimensions.

Uses PRAM pointer 3, which is not preserved.


0xF0C6: Setup PRAM pointer for tile map access
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_setptr
- Cycles: 130
- Param0: Tile map pointer
- Param1: Target pointer (only low 2 bits used)
- Ret. C: PRAM bit offset of tile map, high
- Ret.X3: PRAM bit offset of tile map, low

Sets up the target PRAM pointer for tile map accessing. The pointer is set up
for 16 bit mode, incrementing, pointing at the start of the tile map.




Entry point table of Tile map manager functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.
- W: May wait for a specific event.
- F: Additional callback cycles.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xF0B2 |           120 | 6 |      | us_tmap_set                            |
+--------+---------------+---+------+----------------------------------------+
| 0xF0B4 |           130 | 9 |      | us_tmap_setfn                          |
+--------+---------------+---+------+----------------------------------------+
| 0xF0B6 |   400 + W + F | 2 |      | us_tmap_getacc                         |
+--------+---------------+---+------+----------------------------------------+
| 0xF0B8 |   410 + W + F | 4 |      | us_tmap_getaccxy                       |
+--------+---------------+---+------+----------------------------------------+
| 0xF0BA |   420 + W + F | 5 |      | us_tmap_getaccxfy                      |
+--------+---------------+---+------+----------------------------------------+
| 0xF0BC | 60U + 400 + F | 4 |      | us_tmap_blit                           |
+--------+---------------+---+------+----------------------------------------+
| 0xF0BE |            40 | 1 |      | us_tmap_gethw                          |
+--------+---------------+---+------+----------------------------------------+
| 0xF0C0 |        25 + F | 1 |      | us_tmap_gettilehw                      |
+--------+---------------+---+------+----------------------------------------+
| 0xF0C2 |           170 | 3 |  X3  | us_tmap_gettile                        |
+--------+---------------+---+------+----------------------------------------+
| 0xF0C4 |           180 | 4 |      | us_tmap_settile                        |
+--------+---------------+---+------+----------------------------------------+
| 0xF0C6 |           130 | 2 | C:X3 | us_tmap_setptr                         |
+--------+---------------+---+------+----------------------------------------+
