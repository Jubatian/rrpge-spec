
RRPGE User Library - Tileset based Character writer
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The tileset based character writer can be used to output monospace font (using
a tileset by the tileset interface) on display surfaces. It supports changing
font output style by style attributes (primarily may be used for setting
color, depending on the used tileset, may be used to change font face, too).

The object structure is as follows:

- Word0: <Character writer interface>
- Word1: <Character writer interface>
- Word2: <Character writer interface>
- Word3: X cell position for next character.
- Word4: Y position expressed as Y * surface_width.
- Word5: Width of tileset
- Word6: Height of tileset * surface_width
- Word7: Effective width of surface
- Word8: PRAM pointer (word) of conversion table, high
- Word9: PRAM pointer (word) of conversion table, low
- Word10: Color shift (low 4 bits effective only)
- Word11: Current color (on high bits, low bits are zero)
- Word12: Tileset object pointer
- Word13: Destination surface object pointer

The Effective width of surface might not equal the true surface width as read
from the destination surface object: it is the largest multiple of the font
width which fits in the destination surface width.




Functions
------------------------------------------------------------------------------


0xE12C: Set up tile character writer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_tile_new
- Cycles: 250 + Tileset us_tile_gethw implementation
- Param0: Character writer structure pointer
- Param1: Initial color (appropriate low bits used)
- Param2: Color shift (low 4 bits used)
- Param3: Used tileset's object pointer (tileset interface descendant)
- Param4: Used destination surface's object pointer
- Param5: Conversion table PRAM word pointer, high
- Param6: Conversion table PRAM word pointer, low

Sets up a tileset based character writer by the passed parameters.

The conversion table is used with us_idfutf32 to convert non-ASCII-7
characters to tile indices for the text output (the table does not need to
be restricted to 256 characters).

The color shift is applied to the initial color, and any subsequent color set
by us_cw_tile_setst: the color value is shifted left by this amount before OR
combining with the acquired tile index, before forwarding it to the tileset's
blit function.


0xE12E: Next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_tile_setnc
- Cycles: Depends on character + Tileset us_tile_blit implementation
- Param0: Character writer structure pointer
- Param1: UTF-32 character value, high
- Param2: UTF-32 character value, low

Implements us_cw_setnc in the character writer interface.

If the character is a non-ASCII-7 character, the us_idfutf32 function is used
to convert it to a suitable tile index for blitting.

In the ASCII-7 range some characters are used as control characters, so that
they won't produce a blit, but have a certain effect instead. These are the
followings:

- New line (ASCII 0x0A): Increments Y to the next character row, then Carriage
  Return.
- Carriage return (ASCII 0x0D): Resets X to character column zero.
- Horizontal TAB (ASCII 0x09): Moves X to the next character column which is
  a multiple of 8. May cause a new line.
- Backspace (ASCII 0x08): Moves X back one character column if possible.

The X:Y coordinates are to be interpreted as character columns and rows,
calculated by the tileset's width and height. Initially (when the character
writer is set up) the X:Y location is 0:0. The X dimension (character column)
is constrained to be within the destination width. The Y dimension is not
constrained, it simply wraps around.

The routine has 120 cycles of overhead in addition to us_idfutf32 and the
tileset's us_tile_blit implementation. Control characters may have slightly
different timing with no us_tile_blit call.


0xE130: Set output style
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_tile_setst
- Cycles: 50
- Param0: Character writer structure pointer
- Param1: Style attribute to set (only 'c' is accepted)
- Param2: Value to set

Implements us_cw_setst in the character writer interface.

Changes the output color of the text if the style attribute is 'c' (ASCII
0x63). Otherwise does nothing.


0xE132: Initializes for output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_tile_init
- Cycles: 200 + Wait for frame end + Tileset us_tile_acc implementation
- Param0: Character writer structure pointer

Implements us_cw_init in the character writer interface.

Waits for end of frame if necessary by us_dsurf_getacc, and sets up the
Graphics Accelerator for a sequence of character outputs.


0xE134: Sets character output location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_tile_setxy
- Cycles: 100
- Param0: Character writer structure pointer
- Param1: New character column (X)
- Param2: New character row (Y)

Sets up the X:Y character location on the destination surface to output
characters at. Note that new lines and carriage returns always jump back to
character column 0.




Entry point table of Tileset based character writer functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- F: Additional callback cycles.
- S: For cycle counts see function's description.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE12C |       250 + F | 7 |      | us_cw_tile_new                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE12E |             S | 3 |      | us_cw_tile_setnc                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE130 |            50 | 3 |      | us_cw_tile_setst                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE132 |   200 + W + F | 1 |      | us_cw_tile_init                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE134 |           100 | 3 |      | us_cw_tile_setxy                       |
+--------+---------------+---+------+----------------------------------------+
