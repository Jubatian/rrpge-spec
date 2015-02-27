
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


0xE000: 1 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set1i
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit address, high
- Param2: Start bit address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE002: 1 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set1w
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit address, high
- Param2: Start bit address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE004: 2 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set2i
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE006: 2 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set2w
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE008: 4 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set4i
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE00A: 4 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set4w
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE00C: 8 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set8i
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE00E: 8 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set8w
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE010: 16 bit incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set16i
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE012: 16 bit inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_set16w
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE014: Word incrementing PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setwi
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE016: Word inc. on write PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setww
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Ret.X3: Points to the Read/Write register of the pointer


0xE018: Generic word PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgenwi
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Param3: Increment high (in word units)
- Param4: Increment low (in word units)
- Ret.X3: Points to the Read/Write register of the pointer


0xE01A: Generic word setup for inc. on write
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgenww
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start word address, high
- Param2: Start word address, low
- Param3: Increment high (in word units)
- Param4: Increment low (in word units)
- Ret.X3: Points to the Read/Write register of the pointer

Sets the pointer up for increment on write only mode.


0xE01C: Generic PRAM pointer setup
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_ptr_setgen
- Cycles: 120
- Param0: Target pointer (only low 2 bits used)
- Param1: Start bit(!) address, high
- Param2: Start bit(!) address, low
- Param3: Increment high (in bit units)
- Param4: Increment low (in bit units)
- Param5: Data unit size & Increment on write only flag
- Ret.X3: Points to the Read/Write register of the pointer

Param5 must be formatted according to the Data unit size register of the
pointers.




Copy functions
------------------------------------------------------------------------------


0xE020: PRAM <= CPU RAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_pfc
- Cycles: 200 + 10 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Source CPU RAM word address
- Param3: Count of words to copy (0 is also valid)

Copies data from CPU RAM into PRAM. Uses PRAM pointer 3 for this, which is not
preserved (not restored after the copy).


0xE022: CPU RAM <= PRAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_cfp
- Cycles: 200 + 10 / word
- Param0: Target CPU RAM word address
- Param1: Source PRAM word address, high
- Param2: Source PRAM word address, low
- Param3: Count of words to copy (0 is also valid)

Copies data from PRAM into CPU RAM. Uses PRAM pointer 3 for this, which is not
preserved (not restored after the copy).


0xE024: PRAM <= PRAM copy
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


0xE026: CPU RAM <= CPU RAM copy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_copy_cfc
- Cycles: 200 + 10 / word
- Param0: Target CPU RAM word address
- Param1: Source CPU RAM word address
- Param3: Count of words to copy (0 is also valid)

Copies data from CPU RAM into CPU RAM.


0xE02C: Long PRAM <= PRAM copy
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


0xE028: PRAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_p
- Cycles: 200 + 6 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Value to set the area to
- Param3: Length of the area in words (0 is valid)

Sets the given PRAM memory area to the given value. Uses PRAM pointer 3 for
this, which is not preserved (not restored after the set).


0xE02A: CPU RAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_c
- Cycles: 200 + 6 / word
- Param0: Target CPU RAM word address
- Param1: Value to set the area to
- Param2: Length of the area in words (0 is valid)

Sets the given CPU RAM memory area to the given value.


0xE02E: Long PRAM set
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_set_p_l
- Cycles: 300 + 6 / word
- Param0: Target PRAM word address, high
- Param1: Target PRAM word address, low
- Param2: Value to set the area to
- Param3: Length of the area in words, high (0 is valid)
- Param4: Length of the area in words, low (0 is valid)

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
| 0xE000 |       120 | 3 |  X3  | us_ptr_set1i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE002 |       120 | 3 |  X3  | us_ptr_set1w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE004 |       120 | 3 |  X3  | us_ptr_set2i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE006 |       120 | 3 |  X3  | us_ptr_set2w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE008 |       120 | 3 |  X3  | us_ptr_set4i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE00A |       120 | 3 |  X3  | us_ptr_set4w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE00C |       120 | 3 |  X3  | us_ptr_set8i                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE00E |       120 | 3 |  X3  | us_ptr_set8w                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE010 |       120 | 3 |  X3  | us_ptr_set16i                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xE012 |       120 | 3 |  X3  | us_ptr_set16w                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xE014 |       120 | 3 |  X3  | us_ptr_setwi                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE016 |       120 | 3 |  X3  | us_ptr_setww                               |
+--------+-----------+---+------+--------------------------------------------+
| 0xE018 |       120 | 5 |  X3  | us_ptr_setgenwi                            |
+--------+-----------+---+------+--------------------------------------------+
| 0xE01A |       120 | 5 |  X3  | us_ptr_setgenww                            |
+--------+-----------+---+------+--------------------------------------------+
| 0xE01C |       120 | 6 |  X3  | us_ptr_setgen                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xE01E |           |   |      | <not used>                                 |
+--------+-----------+---+------+--------------------------------------------+
| 0xE020 | 10U + 200 | 4 |      | us_copy_pfc                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xE022 | 10U + 200 | 4 |      | us_copy_cfp                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xE024 | 10U + 200 | 5 |      | us_copy_pfp                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xE026 | 10U + 200 | 3 |      | us_copy_cfc                                |
+--------+-----------+---+------+--------------------------------------------+
| 0xE028 |  6U + 200 | 4 |      | us_set_p                                   |
+--------+-----------+---+------+--------------------------------------------+
| 0xE02A |  6U + 200 | 3 |      | us_set_c                                   |
+--------+-----------+---+------+--------------------------------------------+
| 0xE02C | 10U + 300 | 6 |      | us_copy_pfp_l                              |
+--------+-----------+---+------+--------------------------------------------+
| 0xE02E |  6U + 300 | 5 |      | us_set_p_l                                 |
+--------+-----------+---+------+--------------------------------------------+
