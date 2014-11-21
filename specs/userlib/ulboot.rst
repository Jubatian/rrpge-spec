
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

- 0xF070: us_sprite_reset
- 0xF078: us_smux_reset

The absolute offset of it's first free slot at 0xFAEF is set 0xFAF2
(indicating two functions loaded).

The frame end hook list is empty. The absolute offset of it's first free slot
at 0xFAEE is set 0xFAE0 (indicating empty).




Sprite manager initialization
------------------------------------------------------------------------------


Sprite managers start out non-initialized, all associated locations set zero.




CPU RAM user library range fill map
------------------------------------------------------------------------------


The following table provides the initial fill data to be used for the range
0xF800 - 0xFAFF in the CPU RAM.

+--------+-------------------------------------------------------------------+
| Range  | Fill data                                                         |
+========+===================================================================+
| 0xF800 |                                                                   |
| \-     | 0                                                                 |
| 0xFAED |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAEE | 0xFAE0                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAEF | 0xFAF2                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF0 | 0xF070                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF1 | 0xF078                                                            |
+--------+-------------------------------------------------------------------+
| 0xFAF2 |                                                                   |
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
| 0xFDFFF |                                                                  |
+---------+------------------------------------------------------------------+
