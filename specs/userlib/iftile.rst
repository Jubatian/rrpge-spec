
RRPGE User Library - Tileset interface
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The tileset interface provides a common interface for tile set
implementations. Tile sets are collections of rectangular graphics objects of
identical dimensions, commonly used for tile mapping, less commonly for sprite
collections and other purposes.

The object structure is as follows:

- Word0: Blit function implementation
- Word1: Height:Width request function implementation
- Word2: Accelerator init function implementation

When working with tilesets, before blitting a run of tiles, the Accelerator
init function has to be called. This should initialize the Graphics
Accelerator in a suitable manner for the subsequent blits. It should be
effective until the accelerator is interfaced in some other manner than the
tileset's functions. The tileset implementation shouldn't interfere with the
destination surface manager, and paired with it, should completely set up the
Accelerator for the blits.




Functions
------------------------------------------------------------------------------


0xE0A6: Set up tileset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_new
- Cycles: 50
- Param0: Tileset structure pointer
- Param1: Blit function implementation
- Param2: Height:Width request function implementation
- Param3: Accelerator init function implementation
- Ret.X3: Tileset structure pointer + 3

Sets up a tileset interface structure using the given parameters as-is. This
should be called by tileset implementations in their own setup ("new")
functions, to add the appropriate handlers in the object.


0xE0A8: Initialize accelerator for tile output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_acc
- Cycles: 20 + F
- Param0: Tileset structure pointer

Calls the accelerator init function of the tileset.

The accelerator init function can take one parameter, the tileset structure
pointer, which is always provided. No return value is accepted from it.


0xE0AA: Blit tile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_blit
- Cycles: 15 + F
- Param0: Tileset structure pointer
- Param1: Tile index
- Param2: Destination start offset, whole (in cells)
- Param3: Destination start offset, fraction

Outputs the tile at an arbitrary location. The outcome of the blit should
depend on the destination's configuration.

The blit function can take either three or four parameters, the same as this
function is called with.

The destination start offset fraction may be omitted by calling this function
with only 3 parameters. This case the blit function implementation will be
called with 3 parameters.

A tileset implementation may not support blitting at fractional positions,
this case it's blit implementation may simply ignore the fraction parameter,
and assume being called with 3 parameters.


0xE0AC: Get height and width of tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_gethw
- Cycles: 20 + F
- Param0: Tileset structure pointer
- Ret. C: Height in rows
- Ret.X3: Width in cells

Returns the width and height of a tileset.

The height:width request function can take one parameter, the tileset
structure pointer, which is always provided. The return value is passed back
as-is.




Entry point table of Tileset interface functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- F: Additional callback cycles.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE0A6 |            50 | 4 |  X3  | us_tile_new                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0A8 |        20 + F | 1 |      | us_tile_acc                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0AA |        15 + F | 4 |      | us_tile_blit                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE0AC |        20 + F | 1 | C:X3 | us_tile_gethw                          |
+--------+---------------+---+------+----------------------------------------+
