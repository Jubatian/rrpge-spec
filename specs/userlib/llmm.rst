
RRPGE User Library - Low Level Memory Management
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


Low level memory management functions provide basic assistance for common
memory related tasks, as follows:

- Setting up the PRAM pointers
- Copying blocks of memory (memcpy)
- Setting blocks of memory (memset)

They do not require any resource (CPU RAM or PRAM data) apart from their code.




PRAM pointer setup functions
------------------------------------------------------------------------------


0xF000: 1 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set1i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit address, high
- Param2: Start bit address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF002: 1 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set1w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit address, high
- Param2: Start bit address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF004: 2 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set2i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF006: 2 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set2w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF008: 4 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set4i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF00A: 4 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set4w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF00C: 8 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set8i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF00E: 8 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set8w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF010: 16 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set16i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF012: 16 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set16w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF018: Generic 16 bit PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgen16i
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Param3: Increment high (in word units)
- Param4: Increment low (in word units)
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer


0xF01A: Generic 16 bit setup for inc. on write
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgen16w
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Param3: Increment high (in word units)
- Param4: Increment low (in word units)
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer

Sets the pointer up for increment on write only mode.


0xF01C: Generic PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgen
- Cycles: 200
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Param3: Increment high (in bit units)
- Param4: Increment low (in bit units)
- Param5: Data unit size & Increment on write only flag
- Ret.X3: Points to the Read/Write register of the pointer
- Ret. C: Points to the Read/Write without increment register of the pointer

Param5 must be formatted according to the Data unit size register of the
pointers.




Copy functions
------------------------------------------------------------------------------


0xF020: PRAM <= CPU RAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_pfc
- Cycles: 200 + 10 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Source CPU RAM word address
- Param3: Count of words to copy (0 is also valid)

Copies data from CPU RAM into PRAM. Uses PRAM pointer 3 for this, which is not
preserved (not restored after the copy).


0xF022: CPU RAM <= PRAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_cfp
- Cycles: 200 + 10 / word
- Param0: Target CPU RAM word address
- Param1: Source PRAM word address, high
- Param2: Source PRAM word address, low
- Param3: Count of words to copy (0 is also valid)

Copies data from PRAM into CPU RAM. Uses PRAM pointer 3 for this, which is not
preserved (not restored after the copy).


0xF024: PRAM <= PRAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_pfp
- Cycles: 200 + 10 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Source PRAM word address, high
- Param3: Source PRAM word address, low
- Param4: Count of words to copy (0 is also valid)

Copies data from PRAM into PRAM. Uses PRAM pointer 2 and 3 for this, which are
not preserved (not restored after the copy).


0xF026: CPU RAM <= CPU RAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_cfc
- Cycles: 200 + 10 / word
- Param0: Target CPU RAM word address
- Param1: Source CPU RAM word address
- Param3: Count of words to copy (0 is also valid)

Copies data from CPU RAM into CPU RAM.


0xF02C: Long PRAM <= PRAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_pfp_l
- Cycles: 300 + 10 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Source PRAM word address, high
- Param3: Source PRAM word address, low
- Param4: Count of words to copy, high (0 is also valid)
- Param5: Count of words to copy, low (0 is also valid)

Copies data from PRAM into PRAM. Uses PRAM pointer 2 and 3 for this, which are
not preserved (not restored after the copy).




Fill (memory set) functions
------------------------------------------------------------------------------


0xF028: PRAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_p
- Cycles: 200 + 6 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Value to set the area to
- Param3: Length of the area (0 is valid)

Sets the given PRAM memory area to the given value. Uses PRAM pointer 3 for
this, which is not preserved (not restored after the set).


0xF02A: CPU RAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_c
- Cycles: 200 + 6 / word
- Param0: Target CPU RAM word address
- Param1: Value to set the area to
- Param2: Length of the area (0 is valid)

Sets the given CPU RAM memory area to the given value.


0xF02E: Long PRAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_p_l
- Cycles: 300 + 6 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Value to set the area to
- Param3: Length of the area, high (0 is also valid)
- Param4: Length of the area, low (0 is also valid)

Sets the given PRAM memory area to the given value. Uses PRAM pointer 3 for
this, which is not preserved (not restored after the set).




Entry point table of Low Level Memory Management functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+-----------+---+------+--------------------------------------------+
| Addr.  | Cycles    | P |   R  | Name                                       |
+========+===========+===+======+============================================+
| 0xF000 |       200 | 3 | C:X3 | us_ptr_set1i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF002 |       200 | 3 | C:X3 | us_ptr_set1w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF004 |       200 | 3 | C:X3 | us_ptr_set2i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF006 |       200 | 3 | C:X3 | us_ptr_set2w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF008 |       200 | 3 | C:X3 | us_ptr_set4i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF00A |       200 | 3 | C:X3 | us_ptr_set4w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF00C |       200 | 3 | C:X3 | us_ptr_set8i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF00E |       200 | 3 | C:X3 | us_ptr_set8w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xF010 |       200 | 3 | C:X3 | us_ptr_set16i                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xF012 |       200 | 3 | C:X3 | us_ptr_set16w                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xF014 |           |   |      | <not used>                                 |
+--------+-----------+---+------+--------------------------------------------+
| 0xF016 |           |   |      | <not used>                                 |
+--------+-----------+---+------+--------------------------------------------+
| 0xF018 |       200 | 5 | C:X3 | us_ptr_setgen16i                           |
+--------+-----------+---+------+--------------------------------------------+
| 0xF01A |       200 | 5 | C:X3 | us_ptr_setgen16w                           |
+--------+-----------+---+------+--------------------------------------------+
| 0xF01C |       200 | 6 | C:X3 | us_ptr_setgen                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xF01E |           |   |      | <not used>                                 |
+--------+-----------+---+------+--------------------------------------------+
| 0xF020 | 10U + 200 | 4 |      | us_copy_pfc                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xF022 | 10U + 200 | 4 |      | us_copy_cfp                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xF024 | 10U + 200 | 5 |      | us_copy_pfp                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xF026 | 10U + 200 | 3 |      | us_copy_cfc                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xF028 |  6U + 200 | 4 |      | us_set_p                                   |
+--------+-----------+---+------+--------------------------------------------+
| 0xF02A |  6U + 200 | 3 |      | us_set_c                                   |
+--------+-----------+---+------+--------------------------------------------+
| 0xF02C | 10U + 300 | 6 |      | us_copy_pfp_l                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xF02E |  6U + 300 | 5 |      | us_set_p_l                                 |
+--------+-----------+---+------+--------------------------------------------+
