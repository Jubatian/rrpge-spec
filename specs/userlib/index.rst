
RRPGE User Library specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction to the User Library
------------------------------------------------------------------------------


The User Library complements the system defined in this specification to aid
developers in realizing most common tasks.

From the perspective of RRPGE implementations (either hardware or emulation)
the User Library mostly does not need to be concerned as it may be inserted as
a binary image, developed and produced by a different organization.

The binary image of the User Library shows in the Application's Code address
space (unless it's own code overrides it), and may be accessible through a
function entry point table, which table's presence ensures binary
compatibility across probable different implementations of the library.




User Library resource usage
------------------------------------------------------------------------------


The User Library builds on the top of the initial state described in the core
specification (see "../boot.rst"). It adds binary data over this to support
the features it introduces, using the following regions:

- Code address space: 0xE000 - 0xFFFF.
- Data address space (CPU RAM): 0xF800 - 0xFAFF.
- Peripheral RAM (PRAM): 0xFC000 - 0xFDFFF (32 bit unit address).

The loaded application may override the User Library's code by defining it's
own code being larger than 0xE000 words. This case the User Library becomes
inaccessible.

Data and Peripheral RAM contents may be overridden by the application at will.
The User Library must only rely on the contents of either where the
specification allows it, so the application may reuse areas not essential to
the User Library components which it is actually using.




Calling convention
------------------------------------------------------------------------------


All user library functions use the same calling convention, described as
follows:

- Parameters are passed as function opcode parameters.
- 32 bit parameters are passed high word first.
- 16 bit return value is generated into X3.
- 32 bit return value is generated into C:X3 (C: High part, X3: Low part).
- Registers C and X3 are not preserved (even if the function does not generate
  a return value).
- Registers A, B, D, X0, X1, X2, XM and XH are always preserved.
- XM3 (Pointer mode 3) must be set Incrementing 16 bits (mode 0x6).

The usage of other resources are described in the function's specification.




Description of component documents
------------------------------------------------------------------------------


File "btile.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Basic tileset implementation for the tileset interface, for working with the
Graphics Accelerator.


File "dlist.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Low level display list management and double buffering support.


File "dsurf.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Destination surface structure for working with the Graphics Accelerator.


File "fastmap.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Fast scrolling tile mapper using the tile map manager from "tmap.rst".


File "fontdata.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Code Page 437 compliant binary font in the Peripheral RAM (0xFC800 - 0xFDFFF).


File "iftile.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Tileset interface, providing tile set support for tile mappers and similar.


File "llmm.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Low Level Memory Management functions.


File "math.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Mathematic functions.


File "sprite.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Display List based sprite system.


File "tmap.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Tile map structure and functions for working with the Graphics Accelerator.


File "ulboot.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

User Library boot state description: initial fill values to be provided for
CPU RAM and PRAM locations.




Entry point table of User Library functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.
- W: May wait for a specific event.
- F: Additional callback cycles.
- *: For cycle counts see function's description.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts. Cycle counts are omitted where they are not
possible to be summarized: this case the description of the function defines
it's minimal performance requirements.

Note that each function entry takes 2 words to accommodate for a JMA
instruction jumping to the actual handler. The second opcode of each is
formatted as a NOP. Not used handlers are filled with NOPs.

+--------+---------------+---+------+-------------------------+--------------+
| Addr.  | Cycles        | P |   R  | Name                    | Document     |
+========+===============+===+======+=========================+==============+
| 0xE000 |           120 | 3 | C:X3 | us_ptr_set1i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE002 |           120 | 3 | C:X3 | us_ptr_set1w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE004 |           120 | 3 | C:X3 | us_ptr_set2i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE006 |           120 | 3 | C:X3 | us_ptr_set2w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE008 |           120 | 3 | C:X3 | us_ptr_set4i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE00A |           120 | 3 | C:X3 | us_ptr_set4w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE00C |           120 | 3 | C:X3 | us_ptr_set8i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE00E |           120 | 3 | C:X3 | us_ptr_set8w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE010 |           120 | 3 | C:X3 | us_ptr_set16i           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE012 |           120 | 3 | C:X3 | us_ptr_set16w           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE014 |           120 | 3 | C:X3 | us_ptr_setwi            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE016 |           120 | 3 | C:X3 | us_ptr_setww            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE018 |           120 | 5 | C:X3 | us_ptr_setgenwi         | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE01A |           120 | 5 | C:X3 | us_ptr_setgenww         | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE01C |           120 | 6 | C:X3 | us_ptr_setgen           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE01E |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE020 |     10U + 200 | 4 |      | us_copy_pfc             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE022 |     10U + 200 | 4 |      | us_copy_cfp             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE024 |     10U + 200 | 5 |      | us_copy_pfp             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE026 |     10U + 200 | 3 |      | us_copy_cfc             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE028 |      6U + 200 | 4 |      | us_set_p                | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE02A |      6U + 200 | 3 |      | us_set_c                | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE02C |     10U + 300 | 6 |      | us_copy_pfp_l           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE02E |      6U + 300 | 5 |      | us_set_p_l              | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE030 |           100 | 3 |  X3  | us_dloff_from           | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE032 |           100 | 1 | C:X3 | us_dloff_to             | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE034 |           230 | 3 |  X3  | us_dlist_setptr         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE036 |     15U + 430 | 6 |      | us_dlist_add            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE038 |     15U + 530 | 7 |      | us_dlist_addxy          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE03A |     11U + 380 | 5 |      | us_dlist_addbg          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE03C |     19U + 500 | 6 |      | us_dlist_addlist        | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE03E |     12U + 280 | 1 |      | us_dlist_clear          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE040 |           100 | 1 |  X3  | us_dloff_clip           | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE042 |             W | 3 |  X3  | us_dbuf_init            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE044 |           250 | 2 |  X3  | us_dlist_sb_setptr      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE046 |     15U + 450 | 5 |      | us_dlist_sb_add         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE048 |     15U + 550 | 6 |      | us_dlist_sb_addxy       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE04A |     11U + 400 | 4 |      | us_dlist_sb_addbg       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE04C |     19U + 520 | 5 |      | us_dlist_sb_addlist     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE04E |     12U + 300 | 0 |      | us_dlist_sb_clear       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE050 |             W | 0 |      | us_dbuf_flip            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE052 |        25 + W | 0 |  X3  | us_dbuf_getlist         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE054 |       270 + W | 2 |  X3  | us_dlist_db_setptr      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE056 | 15U + 470 + W | 5 |      | us_dlist_db_add         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE058 | 15U + 570 + W | 6 |      | us_dlist_db_addxy       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE05A | 11U + 420 + W | 4 |      | us_dlist_db_addbg       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE05C | 19U + 540 + W | 5 |      | us_dlist_db_addlist     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE05E | 12U + 320 + W | 0 |      | us_dlist_db_clear       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE060 |           500 | 1 |      | us_dbuf_addfliphook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE062 |           500 | 1 |      | us_dbuf_remfliphook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE064 |           500 | 1 |      | us_dbuf_addframehook    | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE066 |           500 | 1 |      | us_dbuf_remframehook    | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE068 |           500 | 1 |      | us_dbuf_addinithook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE06A |           500 | 1 |      | us_dbuf_reminithook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE06C |      20 / 100 | 0 |      | us_sprite_reset         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE06E |     20 / 1800 | 0 |      | us_smux_reset           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE070 |            40 | 2 |      | us_sprite_setbounds     | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE072 |            40 | 2 |      | us_smux_setbounds       | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE074 | 15U + 510 + W | 5 |      | us_sprite_add           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE076 | 70U + 470 + W | 5 |      | us_smux_add             | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE078 | 15U + 610 + W | 6 |      | us_sprite_addxy         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE07A | 70U + 570 + W | 6 |      | us_smux_addxy           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE07C | 19U + 580 + W | 5 |      | us_sprite_addlist       | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE07E | 75U + 540 + W | 5 |      | us_smux_addlist         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE080 |           100 | 1 |  X3  | us_sin                  | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE082 |           100 | 1 |  X3  | us_cos                  | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE084 |           220 | 1 | C:X3 | us_sincos               | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE086 |      50 / 140 | 1 | C:X3 | us_tfreq                | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE088 |           100 | 4 | C:X3 | us_mul32                | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE08A |           600 | 4 | C:X3 | us_div32                | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE08C |            70 | 1 | C:X3 | us_rec16                | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE08E |           470 | 2 | C:X3 | us_rec32                | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE090 |           260 | 1 |  X3  | us_sqrt16               | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE092 |           650 | 2 |  X3  | us_sqrt32               | math.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE094 |           100 | 5 |      | us_dsurf_new            | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE096 |           120 | 7 |      | us_dsurf_newdbuf        | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE098 |           120 | 7 |      | us_dsurf_newm           | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE09A |           130 | 9 |      | us_dsurf_newmdbuf       | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE09C |        80 + W | 1 | C:X3 | us_dsurf_get            | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE09E |       170 + W | 1 | C:X3 | us_dsurf_getacc         | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0A0 |            50 | 1 | C:X3 | us_dsurf_getpw          | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0A2 |            20 | 0 |      | us_dsurf_init           | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0A4 |            25 | 0 |      | us_dsurf_flip           | dsurf.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0A6 |            50 | 4 |  X3  | us_tile_new             | iftile.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0A8 |        20 + F | 1 |      | us_tile_acc             | iftile.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0AA |        15 + F | 4 |      | us_tile_blit            | iftile.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0AC |        20 + F | 1 | C:X3 | us_tile_gethw           | iftile.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0AE |           110 | 6 |      | us_btile_new            | btile.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0B0 |           200 | 1 |      | us_btile_acc            | btile.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0B2 |           150 | 4 |      | us_btile_blit           | btile.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0B4 |            30 | 1 | C:X3 | us_btile_gethw          | btile.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0B6 |            80 | 6 |      | us_tmap_new             | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0B8 |   340 + W + F | 2 |      | us_tmap_acc             | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0BA |   350 + W + F | 4 |      | us_tmap_accxy           | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0BC |   360 + W + F | 5 |      | us_tmap_accxfy          | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0BE | 60U + 440 + F | 5 |      | us_tmap_blit            | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0C0 |            40 | 1 | C:X3 | us_tmap_gethw           | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0C2 |        20 + F | 1 | C:X3 | us_tmap_gettilehw       | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0C4 |           170 | 3 |  X3  | us_tmap_gettile         | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0C6 |           180 | 4 |      | us_tmap_settile         | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0C8 |           130 | 2 | C:X3 | us_tmap_setptr          | tmap.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0CA |           140 | 9 |      | us_fastmap_new          | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0CC |            25 | 1 |      | us_fastmap_mark         | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0CE |       200 + F | 1 | C:X3 | us_fastmap_gethw        | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0D0 |            30 | 1 | C:X3 | us_fastmap_getyx        | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0D2 |       170 + F | 3 |      | us_fastmap_setdly       | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0D4 |             * | 3 |      | us_fastmap_draw         | fastmap.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0D6 |            50 | 3 |  X3  | us_cr_new               | ifcharr.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0D8 |        20 + F | 2 |      | us_cr_setsi             | ifcharr.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0DA |        15 + F | 1 | C:X3 | us_cr_getnc             | ifcharr.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0DC |            80 | 4 |  X3  | us_cw_new               | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0DE |        15 + F | 3 |      | us_cw_setnc             | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0E0 |        30 + F | 3 |      | us_cw_setst             | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0E2 |        30 + F | 1 |      | us_cw_init              | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0E4 |           110 | 5 |  X3  | us_cwr_new              | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
| 0xE0E6 |        20 + F | 1 |  X3  | us_cwr_nextsi           | ifcharw.rst  |
+--------+---------------+---+------+-------------------------+--------------+
