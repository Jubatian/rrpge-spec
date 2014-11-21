
RRPGE User Library - Display List sprite management
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


Two distinct managers are provided for working with sprites on double buffered
display lists (so operating over the functionality provided by "Double
buffering management" described in "dlist.rst").

The managers assist in automatically selecting a free display list column to
draw the sprite's content in it, in two different manners:

- The simple manager is fast, but will use one display list column per sprite.
- The multiplexing manager is slow, but will select a free display list column
  for every line, making it able to produce lots of sprites.

Both sprite managers use data resources located in the CPU RAM.




Common properties of the managers
------------------------------------------------------------------------------


Both managers are operated through completely identical interfaces. They are
singletons, however the two managers may operate in parallel on distinct
column sets of the same display list.

To initialize and operate either, the double buffering manager must be
initialized, paying attention to adding an appropriate Display list clear. The
clear should target all columns selected to be managed by the sprite manager.
Since the Display list clear is capable to clear only up to 24 columns in
single scan modes, and 48 columns in double scanned modes, without other
provisions at most only this many columns may be used by the sprite managers.

Note that not only the number of colums available can constrain the display of
sprites. The Graphics Display Generator has a maximum amount of cycles which
it can utilize to render a line, if the content added for a line exceeds this,
excess (topmost) elements will not render, or will render only partially. See
"vid_arch.rst" for details.

The managers need to have a page flip hook installed in which they clear their
internal structures containing informations on the usage of the display list.
Normally these hooks are installed (see "ulboot.rst" for details).


Display list column usage
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An appropriate "setbounds" function is provided for both, which is used to set
the display list columns the manager should operate on. The set values are not
adjusted in any manner, however as the manager operates, it will respect the
display list's boundaries even if the set bounds exceed those. There is no
guard against allowing the sprite manager to use the background column (column
zero): if the bounds are initialized so, a messed up background layer will be
the result (as the commands added will show as background pattern).


Top and bottom adding
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sprites may be added either to the "top" or "bottom" end of the display list
column set associated with the sprite manager.

Adding to the top will use the topmost free column of the list, making the
sprite appear above anything added to the bottom, however adding subsequent
sprites will appear below it.

Adding to the bottom will use the bottommost free column of the list, making
the sprite appear below anything added to the top, however adding subsequent
sprites will appear above it.

If the display list column set associated with the manager has no more free
columns, no sprite will be added (note that this condition check is line based
in the case of the multiplexing manager, possibly resulting in partially drawn
sprites).




Simple sprite manager
------------------------------------------------------------------------------


The simple sprite manager is capable to produce at most as many sprites as
many display list columns are reserved for it.

This component uses some CPU RAM locations to perform it's functions. These
locations are not meant to be accessed directly by applications.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFAD8 | Current first non-occupied column on the bottom                   |
+--------+-------------------------------------------------------------------+
| 0xFAD9 | Current first occupied column on the top                          |
+--------+-------------------------------------------------------------------+
| 0xFADA | Count of columns to use                                           |
+--------+-------------------------------------------------------------------+
| 0xFADB | First column to use                                               |
+--------+-------------------------------------------------------------------+
| 0xFADC | Occupation data dirty flag (bit 0 set if *not*(!) dirty)          |
+--------+-------------------------------------------------------------------+

All these locations are zero-initialized.


0xF070: Reset display list occupation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sprite_reset
- Cycles: 20 / 100

Clears display list occupation data (0xFAD8 and 0xFAD9) according to the set
bounds (0xFADA, 0xFADB). This clearing respects the display list configuration
(display list size) as set in the Graphics Display Generator.

Sets the dirty flag (indicating *not* dirty), so if no simple sprite manager
functions were called between two calls to this function, it executes on a
shortcut path (20 cycles).


0xF071: Set sprite area bounds
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sprite_setbounds
- Cycles: 40
- Param0: First display list column to use
- Param1: Count of display list columns to use

The parameters are directly loaded into the appropriate locations (0xFADA,
0xFADB). Clears the dirty flag (indicating dirty).


0xF072: Add graphics component to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sprite_add
- Cycles: 510 + 15 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: Y position to start at (signed 2's complement, can be off-display)

Selects the column to add the sprite to by the current column locations
(0xFAD8 and 0xFAD9), updates the appropriate location (increments the current
first non-occupied on the bottom location if added to the bottom, decrements
the current first occupied on the top location if added to the top), clears
the dirty flag (indicating dirty), then trasfers to us_dlist_db_add.

If the two locations are equal when calling, no sprite is added.

PRAM pointers 2 and 3 are used and not preserved.


0xF073: Add graphics component at X:Y to list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sprite_addxy
- Cycles: 610 + 15 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: X position to start at (signed 2's complement, can be off-display)
- Param5: Y position to start at (signed 2's complement, can be off-display)

Processes identically to us_sprite_add except that it transfers to
us_dlist_db_addxy if the sprite can be added.

PRAM pointers 2 and 3 are used and not preserved.


0xF074: Add render command list to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sprite_addlist
- Cycles: 580 + 19 / line + Wait for frame end
- Param0: PRAM word offset of render command list, high
- Param1: PRAM word offset of render command list, low
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: Y position to start at (signed 2's complement, can be off-display)

Processes identically to us_sprite_add except that it transfers to
us_dlist_db_addlist if the sprite can be added.

PRAM pointers 1, 2 and 3 are used and not preserved.




Multiplexing sprite manager
------------------------------------------------------------------------------


The multiplexing sprite manager keeps track of display list column usage for
every display list row, adding sprites row by row, this way being capable to
output more sprites than how many display list columns are reserved for it
(provided the sprites show on different vertical positions).

This component uses some CPU RAM locations to perform it's functions. These
locations are not meant to be accessed directly by applications.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xF800 | Occupation data, current first non-occupied column on the bottom  |
| \-     | for each display list row. Byte (8 bit) data.                     |
| 0xF8C7 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xF8C8 | Occupation data, current first occupied column on the top for     |
| \-     | each display list row. Byte (8 bit) data.                         |
| 0xF98F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFADD | Occupation data dirty flag (bit 0 set if *not*(!) dirty)          |
+--------+-------------------------------------------------------------------+
| 0xFADE | Count of columns to use                                           |
+--------+-------------------------------------------------------------------+
| 0xFADF | First column to use                                               |
+--------+-------------------------------------------------------------------+

All these locations are zero-initialized.


0xF078: Reset display list occupation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_smux_reset
- Cycles: 20 / 1800

Clears display list occupation data (0xF800 - 0xF98F) according to the set
bounds (0xFADE, 0xFADF). This clearing respects the display list configuration
(display list size) as set in the Graphics Display Generator.

Sets the dirty flag (indicating *not* dirty), so if no multiplexing sprite
manager functions were called between two calls to this function, it executes
on a shortcut path (20 cycles).


0xF079: Set sprite area bounds
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_smux_setbounds
- Cycles: 40
- Param0: First display list column to use
- Param1: Count of display list columns to use

The parameters are directly loaded into the appropriate locations (0xFADA,
0xFADB). Clears the dirty flag (indicating dirty).


0xF07A: Add graphics component to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_smux_add
- Cycles: 470 + 70 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: Y position to start at (signed 2's complement, can be off-display)

Clears the dirty flag (indicating dirty). For the purpose of rendering the
srpite, the operation matches that of us_dlist_add. The display list column to
use is selected on every display list row using the appropriate row of the
occupation data (0xF800 - 0xF98F), operating by the same principles described
at us_sprite_add. If the locations are equal, only the affected row of the
sprite is skipped.

PRAM pointer 3 is used and not preserved.


0xF07B: Add graphics component at X:Y to list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_smux_addxy
- Cycles: 570 + 70 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: X position to start at (signed 2's complement, can be off-display)
- Param5: Y position to start at (signed 2's complement, can be off-display)

Processes identically to us_smux_add except that it operates according to
us_dlist_addxy for rows on which the sprite can be added.

PRAM pointer 3 is used and not preserved.


0xF07C: Add render command list to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_smux_addlist
- Cycles: 540 + 75 / line + Wait for frame end
- Param0: PRAM word offset of render command list, high
- Param1: PRAM word offset of render command list, low
- Param2: Height in lines
- Param3: Bit 0 nonzero: Add to top, otherwise: Add to bottom
- Param4: Y position to start at (signed 2's complement, can be off-display)

Processes identically to us_smux_add except that it operates according to
us_dlist_addlist for rows on which the sprite can be added.

PRAM pointers 2 and 3 are used and not preserved.




Entry point table of Display List sprite management functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.
- W: May wait for a specific event.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xF070 |      20 / 100 | 0 |      | us_sprite_reset                        |
+--------+---------------+---+------+----------------------------------------+
| 0xF071 |            40 | 2 |      | us_sprite_setbounds                    |
+--------+---------------+---+------+----------------------------------------+
| 0xF072 | 15U + 510 + W | 5 |      | us_sprite_add                          |
+--------+---------------+---+------+----------------------------------------+
| 0xF073 | 15U + 610 + W | 6 |      | us_sprite_addxy                        |
+--------+---------------+---+------+----------------------------------------+
| 0xF074 | 19U + 580 + W | 5 |      | us_sprite_addlist                      |
+--------+---------------+---+------+----------------------------------------+
| 0xF075 |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
| 0xF076 |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
| 0xF077 |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
| 0xF078 |     20 / 1800 | 0 |      | us_smux_reset                          |
+--------+---------------+---+------+----------------------------------------+
| 0xF079 |            40 | 2 |      | us_smux_setbounds                      |
+--------+---------------+---+------+----------------------------------------+
| 0xF07A | 70U + 470 + W | 5 |      | us_smux_add                            |
+--------+---------------+---+------+----------------------------------------+
| 0xF07B | 70U + 570 + W | 6 |      | us_smux_addxy                          |
+--------+---------------+---+------+----------------------------------------+
| 0xF07C | 75U + 540 + W | 5 |      | us_smux_addlist                        |
+--------+---------------+---+------+----------------------------------------+
| 0xF07D |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
| 0xF07E |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
| 0xF07F |               |   |      | <not used>                             |
+--------+---------------+---+------+----------------------------------------+
