
RRPGE user library boot up state
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
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

- CPU RAM memory range 0xF800 - 0xFAFF.
- Peripheral RAM memory range 0xFC000 - 0xFDFFF.

Unless otherwise specified, these areas should be initialized to zero.




Double buffering management initialization
------------------------------------------------------------------------------


Display lists are not initialized, starting out being zero.

Surfaces are not initialized, starting out being zero.

The flip performed flag is cleared (zero).

The page flip hook list contains the following functions:

- 0xF06C: us_sprite_reset
- 0xF06E: us_smux_reset
- 0xF0A4: us_dsurf_flip

The absolute offset of it's first free slot at 0xFAEF is set 0xFAF3
(indicating three functions loaded).

The frame end hook list is empty. The absolute offset of it's first free slot
at 0xFAEE is set 0xFAE0 (indicating empty).

The init hook list contains the following functions:

- 0xF0A2: us_dsurf_init

The absolute offset of it's first free slot at 0xFADF is set 0xFAD1
(indicating one function loaded).




Sprite manager initialization
------------------------------------------------------------------------------


Sprite managers start out uninitialized, all associated locations set zero.




Destination surface initialization
------------------------------------------------------------------------------


Destination surfaces start out uninitialized, the flipflop at 0xFABF set zero.
They are usable for single buffered surfaces in this state.

The default surface (up_dsurf at 0xFAC0) is set up as follows:

- PRAM write mask is all set (no masking).
- Surface A and Surface B bank and partition selects are zero.
- Width of destination is 80 cells.
- Partition size is 15 (64K cells, full bank).

This corresponds with the display surface defined in the initial display list.




Tileset manager initialization
------------------------------------------------------------------------------


The tileset manager starts out uninitialized, so the FABC - FABE range set
zero. The default tilesets however are initialized as follows:

- 0xFAAC: Normal 4 bit font. Tile index layout 1, No colorkey.
- 0xFAB0: Inverted 4 bit font. Tile index layout 5, Pixel AND mask colorkey.
- 0xFAB4: Normal 8 bit font. Tile index layout 1, No colorkey.
- 0xFAB8: Inverted 8 bit font. Tile index layout 5, Pixel AND mask colorkey.




CPU RAM user library range fill map
------------------------------------------------------------------------------


The following table provides the initial fill data to be used for the range
0xF800 - 0xFAFF in the CPU RAM.

+--------+-------------------------------------------------------------------+
| Range  | Fill data                                                         |
+========+===================================================================+
| 0xF800 |                                                                   |
| \-     | 0                                                                 |
| 0xFAAB |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAAC |                                                                   |
| \-     | 0x020C, 0x000F, 0xC800, 0x0020                                    |
| 0xFAAF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB0 |                                                                   |
| \-     | 0x020C, 0x000F, 0xC800, 0x01AA                                    |
| 0xFAB3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB4 |                                                                   |
| \-     | 0x040C, 0x000F, 0xC800, 0x0031                                    |
| 0xFAB7 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB8 |                                                                   |
| \-     | 0x040C, 0x000F, 0xC800, 0x01BD                                    |
| 0xFABB |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFABC |                                                                   |
| \-     | 0                                                                 |
| 0xFABF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAC0 | 0xFFFF                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAC1 | 0xFFFF                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAC2 |                                                                   |
| \-     | 0                                                                 |
| 0xFAC5 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAC6 | 0x0050                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAC7 | 0x00F0                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAC8 |                                                                   |
| \-     | 0                                                                 |
| 0xFACF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAD0 | 0xF0A2                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAD1 |                                                                   |
| \-     | 0                                                                 |
| 0xFADE |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFADF | 0xFAD1                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAE0 |                                                                   |
| \-     | 0                                                                 |
| 0xFAED |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAEE | 0xFAE0                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAEF | 0xFAF3                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF0 | 0xF06C                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF1 | 0xF06E                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF2 | 0xF0A4                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF3 |                                                                   |
| \-     | 0                                                                 |
| 0xFAFF |                                                                   |
+--------+-------------------------------------------------------------------+




Peripheral RAM user library range fill map
------------------------------------------------------------------------------

The following table provides the initial fill data to be used for the range
0xFC000 - 0xFDFFF in the Peripheral RAM.

+---------+------------------------------------------------------------------+
| Range   | Fill data                                                        |
+=========+==================================================================+
| 0xFC000 |                                                                  |
| \-      | 0                                                                |
| 0xFC7FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFC800 |                                                                  |
| \-      | User Library font, see "fontdata.rst".                           |
| 0xFDFFF |                                                                  |
+---------+------------------------------------------------------------------+
