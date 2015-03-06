
RRPGE User Library - Font tileset
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The font tileset provides a simple and fast implementation for 1 bit tilesets
(see "iftile.rst"), useful for producing fonts for text output. Combined with
a Destination surface tilesets can be used to produce graphics directly.

The tileset manager works with tileset structures (objects). The structure is
formed as follows, on top of the tileset interface:

- Word0: <Tileset interface>
- Word1: <Tileset interface>
- Word2: <Tileset interface>
- Word3: Width (cells) of tiles
- Word4: Height of tiles
- Word5: Bank of source
- Word6: Start offset of source (tile index 0)
- Word7: Blit configuration

The Blit configuration composes as follows:

- bit 13-15: Bits 5 - 7 of Pixel OR mask
- bit    12: Bit 4 of Pixel OR mask or High bit of Reindex bank select
- bit  9-11: Unused
- bit     8: If set, colorkey is 1, otherwise it is 0
- bit     7: Unused
- bit     6: Reindex by destination if set (only if bit 5 is also set)
- bit     5: Tile index layout (0: OR mask + 12 bits; 1: Reindexing + 12 bits)
- bit     4: If set, 8 bit mode, otherwise 4 bit mode
- bit     3: If set, colorkey is enabled
- bit  0- 2: Unused

If bit 6 and bit 5 are both set, the high 4 bits of the tile index will act as
an OR mask, and Reindexing by destination is always applied.

If bit 6 is clear and bit 5 is set, if the high 4 bits are zero, no reindexing
is applied (the high 4 bits ranging from 1 to 15 select reindex banks).

The Index bits select a tile image within the tileset. Tile images are laid
out row by row, however they are compacted in either 4 or 8 bit planes
depending on mode (4 bitplanes in 4 bit mode, 8 bitplanes in 8 bit mode). The
start offset of a tile image can be calculated as follows: ::

    start_offset + ((index / mode) * (width * height))

Then the bit plane to use is selected with the low 2 or 3 bits (depending on
mode) of the tile index, lower values selecting the lower bit planes. So in 4
bit mode, tiles 0 - 3 occupy the same memory area, on bit planes 0 - 3
respectively.

CPU RAM locations are used for supporting blitting, and as default tilesets
for working with the built-in font:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFA92 | Tile index multiplier (Width * Height).                           |
+--------+-------------------------------------------------------------------+
| 0xFA93 | Memorized Blit configuration (Word7).                             |
+--------+-------------------------------------------------------------------+
| 0xFA94 | Memorized Start offset of source (Word6).                         |
+--------+-------------------------------------------------------------------+
| 0xFA9C |                                                                   |
| \-     | Tileset using Normal 4 bit font (up_font_4).                      |
| 0xFAA3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAA4 |                                                                   |
| \-     | Tileset using Inverted 4 bit font (up_font_4i).                   |
| 0xFAAB |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAAC |                                                                   |
| \-     | Tileset using Normal 8 bit font (up_font_8).                      |
| 0xFAB3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB4 |                                                                   |
| \-     | Tileset using Inverted 8 bit font (up_font_8i).                   |
| 0xFABB |                                                                   |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xE136: Set up tileset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ftile_new
- Cycles: 110
- Param0: Tileset structure pointer
- Param1: Width of tiles in cells
- Param2: Height of tiles in rows
- Param3: PRAM Bank of tile source (64K cell unit)
- Param4: Start offset of tile source (cells)
- Param5: Blit configuration

Sets up a tileset structure using the given parameters as-is. Width must be
between 1 and 128. Height must be between 1 and 512.


0xE138: Initialize accelerator for tile output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ftile_acc
- Cycles: 180
- Param0: Tileset structure pointer

Implements us_tile_acc in the tileset interface.

Sets up the Accelerator for blitting a tileset. This function works together
well with the us_dsurf_getacc call: this two calls together completely setting
up a blit (so blit functions may be called afterwards).

The following accelerator registers are covered:

- 0x000A: Set to the tile width
- 0x0012: Set to the PRAM bank of source
- 0x0013: Zeroed (so assumes no source partitioning)
- 0x0014: Set to 0xFF00 (no source partitioning)
- 0x0016: Set according to the Blit configuration
- 0x0017: Set to the tile height
- 0x0018: Set to the tile width
- 0x0019: Zeroed

The function also sets up the three associated CPU RAM locations (0xFA92,
0xFA93, 0xFA94).


0xE13A: Blit tile at arbitrary location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ftile_blit
- Cycles: 180
- Param0: Tileset structure pointer
- Param1: Tile index
- Param2: Destination start offset, whole
- Param3: Destination start offset, fraction

Implements us_tile_blit in the tileset interface.

Outputs the tile at an arbitrary location, suitable for sprite blitting as
well. The destination offset is simply written to Accelerator 0x001C and
0x001D.


0xE13C: Get height and width of tiles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ftile_gethw
- Cycles: 40
- Param0: Tileset structure pointer
- Ret. C: Height in rows
- Ret.X3: Width in cells

Implements us_tile_gethw in the tileset interface.

Returns the width and height of a tileset.


0xE13E: Set high bits of color or reindex bank
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ftile_setch
- Cycles: 50
- Param0: Tileset structure pointer
- Param1: New color bits (only low 4 bits used)

Allows for setting bits 4-7 of the OR mask, or the highest bit (bit 4) of the
reindex bank select for a font tileset.




Entry point table of Font tileset functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE136 |           110 | 6 |      | us_ftile_new                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE138 |           180 | 1 |      | us_ftile_acc                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE13A |           180 | 4 |      | us_ftile_blit                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE13C |            40 | 1 | C:X3 | us_ftile_gethw                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE13E |            50 | 2 |      | us_ftile_setch                         |
+--------+---------------+---+------+----------------------------------------+
