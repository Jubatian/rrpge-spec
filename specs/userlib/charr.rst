
RRPGE User Library - Character readers
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The character readers provide implementations for the character reader
interface, to support reading from the following sources:

- Byte data in the CPU RAM, using an UTF conversion table.
- Byte data in the Peripheral RAM, using an UTF conversion table.
- UTF-8 data in the CPU RAM.
- UTF-8 data in the Peripheral RAM.

For CPU RAM sources, the index supplied to us_cr_setsi is a word pointer into
the CPU RAM (so it can address the whole CPU RAM memory).

For PRAM sources, the index supplied to us_cr_setsi is a 32 bit unit pointer,
which can access a bank of Peripheral RAM (256 KBytes). The bank to use is
selectable by an additional function.

With all the readers strings can only start at the respective boundaries (CPU
RAM: Word, PRAM: 32 bit unit boundary).

CPU RAM locations are used for providing initialized objects for character
reading from the CPU RAM:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFD88 | UTF-8 data, CPU RAM reader, initialized to CPU RAM position       |
| \-     | 0x0040. (up_cr_utf8)                                              |
| 0xFD8B |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD8C | Byte data, CPU RAM reader, using the Code Page 437 to UTF         |
| \-     | conversion table defined in "fontdata.rst", initialized to CPU    |
| 0xFD91 | RAM position 0x0040. (up_cr_byte)                                 |
+--------+-------------------------------------------------------------------+




Data structures
------------------------------------------------------------------------------


For the Byte data sources, a conversion table is used to map the upper 128
values to UTF-32 characters (the lower 128 values map directly as ASCII-7).
This conversion table in the PRAM simply contains 128 32 bit values (Big
Endian) for byte values 128 - 255.

The following sections describe the object structures used for each of the
four character readers.


Byte data, CPU RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character reader interface>
- Word1: <Character reader interface>
- Word2: Word pointer of next character to read.
- Word3: Word pointer fraction of next character to read (on bit 3).
- Word4: PRAM pointer (word) of conversion table, high.
- Word5: PRAM pointer (word) of conversion table, low.


Byte data, PRAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character reader interface>
- Word1: <Character reader interface>
- Word2: PRAM pointer (bit) of next character to read, high.
- Word3: PRAM pointer (bit) of next character to read, low.
- Word4: PRAM pointer (word) of conversion table, high.
- Word5: PRAM pointer (word) of conversion table, low.


UTF-8 data, CPU RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character reader interface>
- Word1: <Character reader interface>
- Word2: Word pointer of next character to read.
- Word3: Word pointer fraction of next character to read (on bit 3).


UTF-8 data, PRAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character reader interface>
- Word1: <Character reader interface>
- Word2: PRAM pointer (bit) of next character to read, high.
- Word3: PRAM pointer (bit) of next character to read, low.




Functions
------------------------------------------------------------------------------


0xE0F0: Byte, CPU RAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cbyte_new
- Cycles: 130
- Param0: Character reader structure pointer
- Param1: Index to set up
- Param2: PRAM word pointer of conversion table, high
- Param3: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is. Parameters 1 - 3
may be omitted, which case the default values will be as follows:

- Param1: Zero
- Param2 and Param3: The code page 437 to UTF transformation table (up_uf437_h
  and up_uf437_l). If Param3 is omitted, the default will be used for Param2
  as well.


0xE0F2: Byte, CPU RAM set string index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cbyte_setsi
- Cycles: 50
- Param0: Character reader structure pointer
- Param1: Index to set up

Implements us_cr_setsi in the character reader interface.

The index is simply a word pointer into the CPU RAM.


0xE0F4: Byte, CPU RAM get next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cbyte_getnc
- Cycles: 110 / 190
- Param0: Character reader structure pointer
- Ret. C: UTF-32 character value, high
- Ret.X3: UTF-32 character value, low

Implements us_cr_getnc in the character reader interface.

If the next available character is an ASCII-7 character, takes 110 cycles,
otherwise 190.

Uses PRAM pointer 3, which is not preserved.


0xE0F6: Byte, PRAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_pbyte_new
- Cycles: 180
- Param0: Character reader structure pointer
- Param1: PRAM bank to use initially
- Param2: Index to set up
- Param3: PRAM word pointer of conversion table, high
- Param4: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is. Parameters 2 - 4
may be omitted, which case the default values will be as follows:

- Param2: Zero
- Param3 and Param4: The code page 437 to UTF transformation table (up_uf437_h
  and up_uf437_l). If Param4 is omitted, the default will be used for Param3
  as well.


0xE0F8: Byte, PRAM set bank
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_pbyte_setsb
- Cycles: 50
- Param0: Character reader structure pointer
- Param1: Bank to set up

Changes the peripheral bank to read the source from. The index (in-bank) part
of the offset is not modified.


0xE0FA: Byte, PRAM set string index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_pbyte_setsi
- Cycles: 60
- Param0: Character reader structure pointer
- Param1: Index to set up

Implements us_cr_setsi in the character reader interface.

The index is simply a 32 bit unit pointer into the selected PRAM bank.


0xE0FC: Byte, PRAM get next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_pbyte_getnc
- Cycles: 110 / 190
- Param0: Character reader structure pointer
- Ret. C: UTF-32 character value, high
- Ret.X3: UTF-32 character value, low

Implements us_cr_getnc in the character reader interface.

If the next available character is an ASCII-7 character, takes 110 cycles,
otherwise up to 190. Bank boundaries are not respected during reading (so
reading may go past a bank boundary, affecting the currently selected bank
even for the purpose of us_cr_pbyte_setsi).

Uses PRAM pointer 3, which is not preserved.


0xE0FE: UTF-8, CPU RAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cutf8_new
- Cycles: 100
- Param0: Character reader structure pointer
- Param1: Index to set up

Sets up a character reader using the given parameters as-is. Param1 may be
omitted, which case it will be initialized to zero.


0xE100: UTF-8, CPU RAM set string index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cutf8_setsi
- Cycles: 50
- Param0: Character reader structure pointer
- Param1: Index to set up

Implements us_cr_setsi in the character reader interface.

The index is simply a word pointer into the CPU RAM.


0xE102: UTF-8, CPU RAM get next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_cutf8_getnc
- Cycles: 110 / 550
- Param0: Character reader structure pointer
- Ret. C: UTF-32 character value, high
- Ret.X3: UTF-32 character value, low

Implements us_cr_getnc in the character reader interface.

If the next available character is an ASCII-7 character, takes 80 cycles,
otherwise up to 500.


0xE104: UTF-8, PRAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_putf8_new
- Cycles: 150
- Param0: Character reader structure pointer
- Param1: PRAM bank to use initially
- Param2: Index to set up

Sets up a character reader using the given parameters as-is. Param2 may be
omitted, which case it will be initialized to zero.


0xE106: UTF-8, PRAM set bank
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_putf8_setsb
- Cycles: 50
- Param0: Character reader structure pointer
- Param1: Bank to set up

Changes the peripheral bank to read the source from. The index (in-bank) part
of the offset is not modified.


0xE108: UTF-8, PRAM set string index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_putf8_setsi
- Cycles: 60
- Param0: Character reader structure pointer
- Param1: Index to set up

Implements us_cr_setsi in the character reader interface.

The index is simply a 32 bit unit pointer into the selected PRAM bank.


0xE10A: UTF-8, PRAM get next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_putf8_getnc
- Cycles: 110 / 550
- Param0: Character reader structure pointer
- Ret. C: UTF-32 character value, high
- Ret.X3: UTF-32 character value, low

Implements us_cr_getnc in the character reader interface.

If the next available character is an ASCII-7 character, takes 90 cycles,
otherwise up to 540. Bank boundaries are not respected during reading (so
reading may go past a bank boundary, affecting the currently selected bank
even for the purpose of us_cr_pbyte_setsi).

Uses PRAM pointer 3, which is not preserved.




Entry point table of Character reader functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE0F0 |           130 | 4 |      | us_cr_cbyte_new                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0F2 |            50 | 2 |      | us_cr_cbyte_setsi                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0F4 |     110 / 190 | 1 | C:X3 | us_cr_cbyte_getnc                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0F6 |           180 | 5 |      | us_cr_pbyte_new                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0F8 |            50 | 2 |      | us_cr_pbyte_setsb                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0FA |            60 | 2 |      | us_cr_pbyte_setsi                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0FC |     110 / 190 | 1 | C:X3 | us_cr_pbyte_getnc                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE0FE |           100 | 2 |      | us_cr_cutf8_new                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE100 |            50 | 2 |      | us_cr_cutf8_setsi                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE102 |     110 / 550 | 1 | C:X3 | us_cr_cutf8_getnc                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE104 |           150 | 3 |      | us_cr_putf8_new                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE106 |            50 | 2 |      | us_cr_putf8_setsb                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE108 |            60 | 2 |      | us_cr_putf8_setsi                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE10A |     110 / 550 | 1 | C:X3 | us_cr_putf8_getnc                      |
+--------+---------------+---+------+----------------------------------------+
