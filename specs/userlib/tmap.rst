
RRPGE User Library - Tile map manager
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The tile map manager provides basic support for working with Graphics
Accelerator based tile maps. Combined with a Destination surface and a Tileset
(tileset interface) it can be used to produce graphics directly.

The tile map manager works with tile map structures (objects). The structure
is formed as follows:

- Word0: Used tileset's pointer
- Word1: Width of tile map in tiles
- Word2: Height of tile map in tiles
- Word3: Word offset of tile map start in PRAM, high
- Word4: Word offset of tile map start in PRAM, low

CPU RAM locations are used for supporting blitting (set by the us_tmap_acc
and derivative functions):

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFD95 | Destination width (cells).                                        |
+--------+-------------------------------------------------------------------+
| 0xFD96 | Destination height.                                               |
+--------+-------------------------------------------------------------------+
| 0xFD97 | Tile width (cells).                                               |
+--------+-------------------------------------------------------------------+
| 0xFD98 | Tile height.                                                      |
+--------+-------------------------------------------------------------------+
| 0xFD99 | X origin fraction.                                                |
+--------+-------------------------------------------------------------------+
| 0xFD9A | X origin.                                                         |
+--------+-------------------------------------------------------------------+
| 0xFD9B | Y origin.                                                         |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xE0B6: Set up tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_new
- Cycles: 80
- Param0: Target tile map pointer (8 words)
- Param1: Tileset to use (tileset interface structure pointer)
- Param2: Width of tile map in tiles
- Param3: Height of tile map in tiles
- Param4: Word offset of tile map in PRAM, high
- Param5: Word offset of tile map in PRAM, low

Sets up a tile map structure using the given parameters as-is.


0xE0B8: Accelerator setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_acc
- Cycles: 340 + Wait for frame end + Function calls
- Param0: Source tile map pointer
- Param1: Destination surface pointer

Sets up for tile map blitting, using the given tile map and destination
surface. The origin coordinates are set zero.

This function calls the Tileset accelerator init function and the Tileset
Height:Width request function to generate contents for the appropriate fields
in the CPU RAM.

The destination height is calculated from the result of us_dsurf_getpw. If the
width is a power of 2 smaller than the partition size, the height gets the
expectable total cells divided by width. Otherwise the height is the largest
even number which multiplied with width produces a smaller or equal result as
the total cell count of the destination.


0xE0BA: Accelerator setup with boundary origin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_accxy
- Cycles: 350 + Wait for frame end + Function calls
- Param0: Source tile map pointer
- Param1: Destination surface pointer
- Param2: X origin (cells)
- Param3: Y origin

Works the same way like us_tmap_getacc, but with setting up an origin.

The origin can be used to scroll the tile map on the destination. This
function does not provide a fraction for X, so enforcing block boundary (which
is faster). It may be used for vertical scrollers, or scrollers combined with
Display list manipulation where full updates are necessary.


0xE0BC: Accelerator setup with origin
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_accxfy
- Cycles: 360 + Wait for frame end + Function calls
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


0xE0BE: Blit tile map region
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_blit
- Cycles: 440 + 60 / tile + Tile blit function calls
- Param0: Source tile map pointer
- Param1: Top-left tile X
- Param2: Top-left tile Y
- Param3: Width in tiles
- Param4: Height in tiles

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

Note that no boundary checks are done, the offset translation is performed as
written, so if the destination width is not a multiple of the tile width, or
the destination size does not match DestWidth * DestHeight, appropriate
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


0xE0C0: Get height and width of tile map
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_gethw
- Cycles: 40
- Param0: Tile map pointer
- Ret. C: Height in tiles
- Ret.X3: Width in tiles

Returns the width and height of the tile map.


0xE0C2: Get height and width of tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tmap_gettilehw
- Cycles: 20 + Tileset Height:Width request function call
- Param0: Tile map pointer
- Ret. C: Height in rows
- Ret.X3: Width in cells

Returns the width and height of the tileset used by the tile map.


0xE0C4: Get tile index
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


0xE0C6: Set tile index
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


0xE0C8: Setup PRAM pointer for tile map access
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
| 0xE0B6 |            80 | 6 |      | us_tmap_new                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0B8 |   340 + W + F | 2 |      | us_tmap_acc                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0BA |   350 + W + F | 4 |      | us_tmap_accxy                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE0BC |   360 + W + F | 5 |      | us_tmap_accxfy                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE0BE | 60U + 440 + F | 5 |      | us_tmap_blit                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE0C0 |            40 | 1 | C:X3 | us_tmap_gethw                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE0C2 |        20 + F | 1 | C:X3 | us_tmap_gettilehw                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0C4 |           170 | 3 |  X3  | us_tmap_gettile                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0C6 |           180 | 4 |      | us_tmap_settile                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0C8 |           130 | 2 | C:X3 | us_tmap_setptr                         |
+--------+---------------+---+------+----------------------------------------+
