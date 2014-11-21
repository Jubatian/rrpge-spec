
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

- Code address space: 0xF000 - 0xFFFF.
- Data address space (CPU RAM): 0xF800 - 0xFAFF.
- Peripheral RAM (PRAM): 0xFC000 - 0xFDFFF (32 bit unit address).

The loaded application may override the User Library's code by defining it's
own code being larger than 0xF000 words. This case the User Library becomes
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


File "dlist.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Low level display list management and double buffering support.


File "llmm.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Low Level Memory Management functions.


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
| 0xF000 |           120 | 3 | C:X3 | us_ptr_set1i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF002 |           120 | 3 | C:X3 | us_ptr_set1w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF004 |           120 | 3 | C:X3 | us_ptr_set2i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF006 |           120 | 3 | C:X3 | us_ptr_set2w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF008 |           120 | 3 | C:X3 | us_ptr_set4i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF00A |           120 | 3 | C:X3 | us_ptr_set4w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF00C |           120 | 3 | C:X3 | us_ptr_set8i            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF00E |           120 | 3 | C:X3 | us_ptr_set8w            | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF010 |           120 | 3 | C:X3 | us_ptr_set16i           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF012 |           120 | 3 | C:X3 | us_ptr_set16w           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF014 |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF016 |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF018 |           120 | 5 | C:X3 | us_ptr_setgen16i        | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF01A |           120 | 5 | C:X3 | us_ptr_setgen16w        | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF01C |           120 | 6 | C:X3 | us_ptr_setgen           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF01E |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF020 |     10U + 200 | 4 |      | us_copy_pfc             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF022 |     10U + 200 | 4 |      | us_copy_cfp             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF024 |     10U + 200 | 5 |      | us_copy_pfp             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF026 |     10U + 200 | 3 |      | us_copy_cfc             | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF028 |      6U + 200 | 4 |      | us_set_p                | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF02A |      6U + 200 | 3 |      | us_set_c                | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF02C |     10U + 300 | 6 |      | us_copy_pfp_l           | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF02E |      6U + 300 | 5 |      | us_set_p_l              | llmm.rst     |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF030 |           100 | 3 |  X3  | us_dloff_from           | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF032 |           100 | 1 | C:X3 | us_dloff_to             | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF034 |           230 | 3 |  X3  | us_dlist_setptr         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF036 |     15U + 430 | 6 |      | us_dlist_add            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF038 |     15U + 530 | 7 |      | us_dlist_addxy          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF03A |     11U + 380 | 5 |      | us_dlist_addbg          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF03C |     19U + 500 | 6 |      | us_dlist_addlist        | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF03E |     12U + 280 | 1 |      | us_dlist_clear          | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF040 |           100 | 1 |  X3  | us_dloff_clip           | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF042 |             W | 3 |  X3  | us_dbuf_init            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF044 |           250 | 2 |  X3  | us_dlist_sb_setptr      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF046 |     15U + 450 | 5 |      | us_dlist_sb_add         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF048 |     15U + 550 | 6 |      | us_dlist_sb_addxy       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF04A |     11U + 400 | 4 |      | us_dlist_sb_addbg       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF04C |     19U + 520 | 5 |      | us_dlist_sb_addlist     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF04E |     12U + 300 | 0 |      | us_dlist_sb_clear       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF050 |             W | 0 |      | us_dbuf_flip            | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF052 |        25 + W | 0 |  X3  | us_dbuf_getlist         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF054 |       270 + W | 2 |  X3  | us_dlist_db_setptr      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF056 | 15U + 470 + W | 5 |      | us_dlist_db_add         | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF058 | 15U + 570 + W | 6 |      | us_dlist_db_addxy       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF05A | 11U + 420 + W | 4 |      | us_dlist_db_addbg       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF05C | 19U + 540 + W | 5 |      | us_dlist_db_addlist     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF05E | 12U + 320 + W | 0 |      | us_dlist_db_clear       | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF060 |           500 | 1 |      | us_dbuf_addfliphook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF062 |           500 | 1 |      | us_dbuf_remfliphook     | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF064 |           500 | 1 |      | us_dbuf_addframehook    | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF066 |           500 | 1 |      | us_dbuf_remframehook    | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF068 |           100 | 5 |      | us_dbuf_setsurface      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF06A |            50 | 1 | C:X3 | us_dbuf_getsurface      | dlist.rst    |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF070 |      20 / 100 | 0 |      | us_sprite_reset         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF071 |            40 | 2 |      | us_sprite_setbounds     | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF072 | 15U + 510 + W | 5 |      | us_sprite_add           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF073 | 15U + 610 + W | 6 |      | us_sprite_addxy         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF074 | 19U + 580 + W | 5 |      | us_sprite_addlist       | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF075 |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF076 |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF077 |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF078 |     20 / 1800 | 0 |      | us_smux_reset           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF079 |            40 | 2 |      | us_smux_setbounds       | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07A | 70U + 470 + W | 5 |      | us_smux_add             | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07B | 70U + 570 + W | 6 |      | us_smux_addxy           | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07C | 75U + 540 + W | 5 |      | us_smux_addlist         | sprite.rst   |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07D |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07E |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
| 0xF07F |               |   |      | <not used>              |              |
+--------+---------------+---+------+-------------------------+--------------+
