
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


File "llmm.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Low Level Memory Management functions.




Entry point table of User Library functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts. Cycle counts are omitted where they are not
possible to be summarized: this case the description of the function defines
it's minimal performance requirements.

Note that each function entry takes 2 words to accommodate for a JMA
instruction jumping to the actual handler. The second opcode of each is
formatted as a NOP. Not used handlers are filled with NOPs.

+--------+-----------+---+------+-----------------------------+--------------+
| Addr.  | Cycles    | P |   R  | Name                        | Document     |
+========+===========+===+======+=============================+==============+
| 0xF000 |       200 | 3 | C:X3 | us_ptr_set1i                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF002 |       200 | 3 | C:X3 | us_ptr_set1w                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF004 |       200 | 3 | C:X3 | us_ptr_set2i                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF006 |       200 | 3 | C:X3 | us_ptr_set2w                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF008 |       200 | 3 | C:X3 | us_ptr_set4i                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF00A |       200 | 3 | C:X3 | us_ptr_set4w                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF00C |       200 | 3 | C:X3 | us_ptr_set8i                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF00E |       200 | 3 | C:X3 | us_ptr_set8w                | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF010 |       200 | 3 | C:X3 | us_ptr_set16i               | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF012 |       200 | 3 | C:X3 | us_ptr_set16w               | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF014 |           |   |      | <not used>                  |              |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF016 |           |   |      | <not used>                  |              |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF018 |       200 | 5 | C:X3 | us_ptr_setgen16i            | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF01A |       200 | 5 | C:X3 | us_ptr_setgen16w            | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF01C |       200 | 6 | C:X3 | us_ptr_setgen               | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF01E |           |   |      | <not used>                  |              |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF020 | 10U + 200 | 4 |      | us_copy_pfc                 | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF022 | 10U + 200 | 4 |      | us_copy_cfp                 | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF024 | 10U + 200 | 5 |      | us_copy_pfp                 | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF026 | 10U + 200 | 3 |      | us_copy_cfc                 | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF028 |  6U + 200 | 4 |      | us_set_p                    | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF02A |  6U + 200 | 3 |      | us_set_c                    | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF02C | 10U + 300 | 6 |      | us_copy_pfp_l               | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
| 0xF02E |  6U + 300 | 5 |      | us_set_p_l                  | llmm.rst     |
+--------+-----------+---+------+-----------------------------+--------------+
