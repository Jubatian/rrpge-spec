
RRPGE user library boot up state
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


When starting an application, the RRPGE system starts up in a definite state.
The core system's boot up state is described in "boot.rst", this document
complements that with defining the boot up state of memories associated with
the User Library.

The following two RAM memory areas are covered in this specification:

- CPU RAM memory range 0xFB00 - 0xFDFF.
- Peripheral RAM memory range 0xFE200 - 0xFF7FF.

Unless otherwise specified, these areas should be initialized to zero.




Double buffering management initialization
------------------------------------------------------------------------------


Display lists are not initialized, starting out being zero.

Surfaces are not initialized, starting out being zero.

The flip performed flag is cleared (zero).

The page flip hook list contains the following functions:

- 0xE06C: us_sprite_reset
- 0xE06E: us_smux_reset
- 0xE0A4: us_dsurf_flip

The absolute offset of it's first free slot at 0xFDEF is set 0xFDF3
(indicating three functions loaded).

The frame end hook list is empty. The absolute offset of it's first free slot
at 0xFDEE is set 0xFDE0 (indicating empty).

The init hook list contains the following functions:

- 0xE0A2: us_dsurf_init

The absolute offset of it's first free slot at 0xFDDF is set 0xFDD1
(indicating one function loaded).




Sprite manager initialization
------------------------------------------------------------------------------


Sprite managers start out uninitialized, all associated locations set zero.




Destination surface initialization
------------------------------------------------------------------------------


Destination surfaces start out uninitialized, the flipflop at 0xFDBF set zero.
They are usable for single buffered surfaces in this state.

The default surface (up_dsurf at 0xFDC0) is set up as follows:

- PRAM write mask is all set (no masking).
- Surface A and Surface B bank and partition selects are zero.
- Width of destination is 80 cells.
- Partition size is 15 (64K cells, full bank).

This corresponds with the display surface defined in the initial display list.




Tileset manager initializations
------------------------------------------------------------------------------


The tileset managers (btile and ftile) start out uninitialized, so the
0xFDBC - 0xFDBE and 0xFD92 - 0xFD94 ranges set zero. The default tilesets
however are initialized as follows over ftile:

- 0xFD9C: Normal font. Reindexing + 12 bits, No colorkey.
- 0xFDA4: Inverted font. Pixel OR mask + 12 bits, colorkeyed.




Tile map manager initialization
------------------------------------------------------------------------------


The tile map manager start out uninitialized, all associated locations set
zero.




Character readers and writers
------------------------------------------------------------------------------


Two pre-initialized character reader objects are provided to read the CPU RAM:
a byte reader at 0xFD8C and a utf-8 reader at 0xFD88 (see "charr.rst").




CPU RAM user library range fill map
------------------------------------------------------------------------------


The following table provides the initial fill data to be used for the range
0xFB00 - 0xFDFF in the CPU RAM.

+--------+-------------------------------------------------------------------+
| Range  | Fill data                                                         |
+========+===================================================================+
| 0xFB00 |                                                                   |
| \-     | 0                                                                 |
| 0xFD83 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD88 |                                                                   |
| \-     | 0xE102, 0xE100, 0x0040, 0                                         |
| 0xFD8B |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD8C |                                                                   |
| \-     | 0xE0F4, 0xE0F2, 0x0040, 0, 0x001F, 0xD300                         |
| 0xFD91 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD92 |                                                                   |
| \-     | 0                                                                 |
| 0xFD9B |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD9C |                                                                   |
| \-     | 0xE13A, 0xE13C, 0xE138, 0x0001, 0x000C, 0x000F, 0xEA00, 0x0020    |
| 0xFDA3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDA4 |                                                                   |
| \-     | 0xE13A, 0xE13C, 0xE138, 0x0001, 0x000C, 0x000F, 0xF100, 0x0108    |
| 0xFDAB |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDAC |                                                                   |
| \-     | 0                                                                 |
| 0xFDBF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDC0 |                                                                   |
| \-     | 0xFFFF, 0xFFFF, 0xF000, 0, 0, 0, 0, 0x0050                        |
| 0xFDC7 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDC8 |                                                                   |
| \-     | 0                                                                 |
| 0xFDCF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDD0 | 0xE0A2                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDD1 |                                                                   |
| \-     | 0                                                                 |
| 0xFDDE |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDDF | 0xFDD1                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDE0 |                                                                   |
| \-     | 0                                                                 |
| 0xFDED |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFDEE | 0xFDE0                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDEF | 0xFDF3                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDF0 | 0xE06C                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDF1 | 0xE06E                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDF2 | 0xE0A4                                                            |
+--------+-------------------------------------------------------------------+
| 0xFDF3 |                                                                   |
| \-     | 0                                                                 |
| 0xFDFF |                                                                   |
+--------+-------------------------------------------------------------------+




Peripheral RAM user library range fill map
------------------------------------------------------------------------------


The following table provides the initial fill data to be used for the range
0xFC000 - 0xFDFFF in the Peripheral RAM.

+---------+------------------------------------------------------------------+
| Range   | Fill data                                                        |
+=========+==================================================================+
| 0xFE200 |                                                                  |
| \-      | 0                                                                |
| 0xFE5FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFE600 |                                                                  |
| \-      | UTF to font transformation table, see "fontdata.rst".            |
| 0xFE97F |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFE980 |                                                                  |
| \-      | Code page 437 to UTF transformation table, see "fontdata.rst".   |
| 0xFE9FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFEA00 |                                                                  |
| \-      | Normal font, see "fontdata.rst".                                 |
| 0xFF0FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFF100 |                                                                  |
| \-      | Inverted font, see "fontdata.rst".                               |
| 0xFF7FF |                                                                  |
+---------+------------------------------------------------------------------+
