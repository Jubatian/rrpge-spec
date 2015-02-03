
RRPGE User Library - Tileset manager
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The tileset manager provides support for Graphics Accelerator based tiles and
sprites, useful as a base for most 2D rendering tasks. Combined with a
Destination surface tilesets can be used to produce graphics directly.

The tileset manager works with tileset structures (objects). The structure is
formed as follows:

- Word0: Width (high 7 bits, in cells), and Height (low 9 bits) of tile
- Word1: Bank of source
- Word2: Start offset of source (tile index 0)
- Word3: Blit configuration

A zero value for Width translates to 128 cells, a zero value for Height to
512 rows.

The Blit configuration composes as follows:

- bit  9-15: Bits 1 - 7 of Pixel AND mask (bit 0 is always 1)
- bit     8: If set, colorkey is Pixel AND mask, otherwise it is 0
- bit  5- 7: Tile index layout
- bit     4: If set, 8 bit mode, otherwise 4 bit mode
- bit     3: If set, colorkey is enabled
- bit  0- 2: Pixel barrel rotate right

The tile index layout defines how the tile index word (16 bits, the first
parameter of blit functions) should be translated to a tile image. The
following layouts are possible:

- 0: bit 15: Reindex by destination enabled; bit 0-14: Index
- 1: bit 11-15: Reindex bank (0: no reindexing;  1-31); bit 0-10: Index
- 2: bit 12-15: Reindex bank (0: no reindexing; 17-31); bit 0-11: Index
- 3: bit 13-15: Reindex bank (0: no reindexing; 25-31); bit 0-12: Index
- 4: bit 12-15: OR mask (low 4 bits); bit 0-11: Index
- 5: bit  8-15: OR mask (all bits); bit 0-7: Index
- 6: bit 12-15: OR mask (high 4 bits); bit 0-11: Index
- 7: bit 13-15: OR mask (high 3 bits); bit 0-12: Index

The Index bits select a tile image within the tileset. Tiles within the
tileset are laid out row by row, each tile on one contiguous area of Width *
Height size. The offset of the tile image selected so can be calculated as: ::

    start_offset + (index * (width * height))

CPU RAM locations are used for supporting blitting, and as default tilesets
for working with the built-in font:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFAAC |                                                                   |
| \-     | Tileset using Normal 4 bit font (up_font_4).                      |
| 0xFAAF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB0 |                                                                   |
| \-     | Tileset using Inverted 4 bit font (up_font_4i).                   |
| 0xFAB3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB4 |                                                                   |
| \-     | Tileset using Normal 8 bit font (up_font_8).                      |
| 0xFAB7 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB8 |                                                                   |
| \-     | Tileset using Inverted 8 bit font (up_font_8i).                   |
| 0xFABB |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFABC | Tile index multiplier (Width * Height).                           |
+--------+-------------------------------------------------------------------+
| 0xFABD | Memorized Tile index layout on low 3 bits, other bits undefined.  |
+--------+-------------------------------------------------------------------+
| 0xFABE | Memorized Start offset of source (Word2).                         |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xE0A8: Set up tileset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_set
- Cycles: 100
- Param0: Target tileset pointer (4 words)
- Param1: Width of tiles in cells (only low 7 bits used, 0 => 128 cells)
- Param2: Height of tiles in rows (only low 9 bits used, 0 => 512 rows)
- Param3: PRAM Bank of tile source (64K cell unit)
- Param4: Start offset of tile source (cells)
- Param5: Blit configuration

Sets up a tileset structure using the given parameters as-is.


0xE0AA: Initialize accelerator for tile output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_getacc
- Cycles: 250
- Param0: Source tileset pointer (4 words)

Sets up the Accelerator for blitting a tileset. This function works together
well with the us_dsurf_getacc call: this two calls together completely setting
up a blit (so blit functions may be called afterwards).

The following accelerator registers are covered:

- 0x800A: Set to the tile width
- 0x8012: Set to the PRAM bank of source
- 0x8013: Zeroed (so assumes no source partitioning)
- 0x8015: Set according to the Blit configuration (using Block Blitter)
- 0x8016: Set according to the Blit configuration
- 0x8017: Set to the tile height
- 0x8018: Set to the tile width
- 0x8019: Zeroed

The function also sets up the three associated CPU RAM locations (0xFABC,
0xFABD, 0xFABE).


0xE0AC: Blit tile at arbitrary location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_blit
- Cycles: 150
- Param0: Tile index
- Param1: Destination start offset, whole
- Param2: Destination start offset, fraction

Outputs the tile at an arbitrary location, suitable for sprite blitting as
well. The destination offset is simply written to Accelerator 0x801C and
0x801D.

The us_tile_getacc function has to be called before this to set up a tileset
(however any number of blits may be performed after the call from the same
tileset).


0xE0AE: Blit tile at boundary
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_blitb
- Cycles: 150
- Param0: Tile index
- Param1: Destination start offset, whole

Outputs the tile at cell boundary, suitable for tile mapping. The destination
offset is simply written to Accelerator 0x801C. 0x801D is set zero.

The us_tile_getacc function has to be called before this to set up a tileset
(however any number of blits may be performed after the call from the same
tileset).


0xE0B0: Get height and width of tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tile_gethw
- Cycles: 60
- Param0: Source tileset pointer (4 words)
- Ret. C: Height in rows
- Ret.X3: Width in cells

Returns the width and height of a tileset.



Entry point table of Tileset manager functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE0A8 |           100 | 6 |      | us_tile_set                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0AA |           250 | 1 |      | us_tile_getacc                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE0AC |           150 | 3 |      | us_tile_blit                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE0AE |           150 | 2 |      | us_tile_blitb                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE0B0 |            60 | 1 | C:X3 | us_tile_gethw                          |
+--------+---------------+---+------+----------------------------------------+
