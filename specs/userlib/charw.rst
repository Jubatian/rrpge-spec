
RRPGE User Library - Character writers
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The character writers provide implementations for the extended character
writer interface, to support writing to the following destinations:

- Byte data in the CPU RAM, using the us_idfutf32 function.
- Byte data in the Peripheral RAM, using the us_idfutf32 function.
- UTF-8 data in the CPU RAM.
- UTF-8 data in the Peripheral RAM.

Using the extended character writer's us_cwr_nextid function the produced
buffers may be used later with the matching character readers.

Output styles and output initializators from the character writer interface
are not implemented (not necessary for these targets).

All the writers support termination offsets, reaching which the character
output stops (the us_cw_setnc function having no effect). This feature is also
supported with a terminating zero, which is placed at the last valid index in
the target.




Data structures
------------------------------------------------------------------------------


The following sections describe the object structures used for each of the
four character writers.


Byte data, CPU RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character writer extended interface>
- Word1: <Character writer extended interface>
- Word2: <Character writer extended interface>
- Word3: <Character writer extended interface>
- Word4: Pointer (8 bit) of next character to write, high (1 bit).
- Word5: Pointer (8 bit) of next character to write, low.
- Word6: Pointer (8 bit) of termination, high (1 bit).
- Word7: Pointer (8 bit) of termination, low.
- Word8: PRAM pointer (word) of conversion table, high.
- Word9: PRAM pointer (word) of conversion table, low.


Byte data, PRAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character writer extended interface>
- Word1: <Character writer extended interface>
- Word2: <Character writer extended interface>
- Word3: <Character writer extended interface>
- Word4: Pointer (bit) of next character to write, high.
- Word5: Pointer (bit) of next character to write, low.
- Word6: Pointer (bit) of termination, high.
- Word7: Pointer (bit) of termination, low.
- Word8: PRAM pointer (word) of conversion table, high.
- Word9: PRAM pointer (word) of conversion table, low.


UTF-8 data, CPU RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character writer extended interface>
- Word1: <Character writer extended interface>
- Word2: <Character writer extended interface>
- Word3: <Character writer extended interface>
- Word4: Pointer (8 bit) of next character to write, high (1 bit).
- Word5: Pointer (8 bit) of next character to write, low.
- Word6: Pointer (8 bit) of termination, high (1 bit).
- Word7: Pointer (8 bit) of termination, low.


UTF-8 data, PRAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Word0: <Character writer extended interface>
- Word1: <Character writer extended interface>
- Word2: <Character writer extended interface>
- Word3: <Character writer extended interface>
- Word4: Pointer (bit) of next character to write, high.
- Word5: Pointer (bit) of next character to write, low.
- Word6: Pointer (bit) of termination, high.
- Word7: Pointer (bit) of termination, low.




Functions
------------------------------------------------------------------------------


0xE10C: Byte, CPU RAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cbyte_new
- Cycles: 130
- Param0: Character writer structure pointer
- Param1: Index (word offset) to start at
- Param2: Index (word offset) of termination
- Param3: PRAM word pointer of conversion table, high
- Param4: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is. The termination
index is exclusive (the first address not written).


0xE10E: Byte, CPU RAM set up with terminator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cbyte_newz
- Cycles: 150
- Param0: Character writer structure pointer
- Param1: Index (word offset) to start at
- Param2: Index (word offset) of termination
- Param3: PRAM word pointer of conversion table, high
- Param4: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is, with an enforced
terminating zero. The termination index is exclusive (the first address not
written), the zero is placed at the index before that, and the object
structure is set up with one smaller index for terminator.


0xE110: Byte, CPU RAM set next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cbyte_setnc
- Cycles: 180 / S
- Param0: Character writer structure pointer
- Param1: UTF-32 character value, high
- Param2: UTF-32 character value, low

Implements us_cw_setnc in the character writer interface.

If the character to write is an ASCII-7 character, takes 180 cycles, otherwise
it depends on the table used (see us_idfutf32).

Uses PRAM pointer 3, which is not preserved.


0xE112: Byte, CPU RAM get next index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cbyte_nextsi
- Cycles: 80
- Param0: Character writer structure pointer
- Ret.X3: Index for next string

Implements us_cwr_nextsi in the extended character writer interface.

The index is simply a word pointer into the CPU RAM. It is provided by seeking
to the next word boundary as necessary. Note that if the terminator is already
reached, the index returned will be the terminator (if the object was set up
with a terminating zero, this will point to that zero).


0xE114: Byte, PRAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_pbyte_new
- Cycles: 130
- Param0: Character writer structure pointer
- Param1: PRAM bank
- Param2: Index (32 bit offset) to start at
- Param3: Index (32 bit offset) of termination
- Param4: PRAM word pointer of conversion table, high
- Param5: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is. The termination
index is exclusive (the first address not written). Providing zero for
termination sets it up at the end of the bank.


0xE116: Byte, PRAM set up with terminator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_pbyte_newz
- Cycles: 170
- Param0: Character writer structure pointer
- Param1: PRAM bank
- Param2: Index (32 bit offset) to start at
- Param3: Index (32 bit offset) of termination
- Param4: PRAM word pointer of conversion table, high
- Param5: PRAM word pointer of conversion table, low

Sets up a character reader using the given parameters as-is, with an enforced
terminating zero. The termination index is exclusive (the first address not
written), the zero is placed at the index before that, and the object
structure is set up with one smaller index for terminator. Providing zero for
termination sets it up at the end of the bank (the last in-bank index will
receive the zero, and the stored terminator will point here).

Uses PRAM pointer 3, which is not preserved.


0xE118: Byte, PRAM set next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_pbyte_setnc
- Cycles: 180 / S
- Param0: Character writer structure pointer
- Param1: UTF-32 character value, high
- Param2: UTF-32 character value, low

Implements us_cw_setnc in the character writer interface.

If the character to write is an ASCII-7 character, takes 180 cycles, otherwise
it depends on the table used (see us_idfutf32).

Uses PRAM pointer 3, which is not preserved.


0xE11A: Byte, PRAM get next index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_pbyte_nextsi
- Cycles: 100
- Param0: Character writer structure pointer
- Ret.X3: Index for next string

Implements us_cwr_nextsi in the extended character writer interface.

The index is simply a 32 bit pointer into the selected PRAM bank. It is
provided by seeking to the next 32 bit boundary as necessary. Note that if the
terminator is already reached, the index returned will be the terminator (if
the object was set up with a terminating zero, this will point to that zero).


0xE11C: UTF-8, CPU RAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cutf8_new
- Cycles: 110
- Param0: Character writer structure pointer
- Param1: Index (word offset) to start at
- Param2: Index (word offset) of termination

Sets up a character reader using the given parameters as-is. The termination
index is exclusive (the first address not written).


0xE11E: UTF-8, CPU RAM set up with terminator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cutf8_newz
- Cycles: 130
- Param0: Character writer structure pointer
- Param1: Index (word offset) to start at
- Param2: Index (word offset) of termination

Sets up a character reader using the given parameters as-is, with an enforced
terminating zero. The termination index is exclusive (the first address not
written), the zero is placed at the index before that, and the object
structure is set up with one smaller index for terminator.


0xE120: UTF-8, CPU RAM set next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cutf8_setnc
- Cycles: 180 / 500
- Param0: Character writer structure pointer
- Param1: UTF-32 character value, high
- Param2: UTF-32 character value, low

Implements us_cw_setnc in the character writer interface.

If the character to write is an ASCII-7 character, takes 180 cycles, otherwise
up to 500 cycles.


0xE122: UTF-8, CPU RAM get next index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_cutf8_nextsi
- Cycles: 80
- Param0: Character writer structure pointer
- Ret.X3: Index for next string

Implements us_cwr_nextsi in the extended character writer interface.

The index is simply a word pointer into the CPU RAM. It is provided by seeking
to the next word boundary as necessary. Note that if the terminator is already
reached, the index returned will be the terminator (if the object was set up
with a terminating zero, this will point to that zero).


0xE124: UTF-8, PRAM set up
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_putf8_new
- Cycles: 110
- Param0: Character writer structure pointer
- Param1: PRAM bank
- Param2: Index (32 bit offset) to start at
- Param3: Index (32 bit offset) of termination

Sets up a character reader using the given parameters as-is. The termination
index is exclusive (the first address not written). Providing zero for
termination sets it up at the end of the bank.


0xE126: UTF-8, PRAM set up with terminator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_putf8_newz
- Cycles: 150
- Param0: Character writer structure pointer
- Param1: PRAM bank
- Param2: Index (32 bit offset) to start at
- Param3: Index (32 bit offset) of termination

Sets up a character reader using the given parameters as-is, with an enforced
terminating zero. The termination index is exclusive (the first address not
written), the zero is placed at the index before that, and the object
structure is set up with one smaller index for terminator. Providing zero for
termination sets it up at the end of the bank (the last in-bank index will
receive the zero, and the stored terminator will point here).

Uses PRAM pointer 3, which is not preserved.


0xE128: UTF-8, PRAM set next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_putf8_setnc
- Cycles: 180 / 500
- Param0: Character writer structure pointer
- Param1: UTF-32 character value, high
- Param2: UTF-32 character value, low

Implements us_cw_setnc in the character writer interface.

If the character to write is an ASCII-7 character, takes 180 cycles, otherwise
up to 500 cycles.

Uses PRAM pointer 3, which is not preserved.


0xE12A: UTF-8, PRAM get next index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_putf8_nextsi
- Cycles: 100
- Param0: Character writer structure pointer
- Ret.X3: Index for next string

Implements us_cwr_nextsi in the extended character writer interface.

The index is simply a 32 bit pointer into the selected PRAM bank. It is
provided by seeking to the next 32 bit boundary as necessary. Note that if the
terminator is already reached, the index returned will be the terminator (if
the object was set up with a terminating zero, this will point to that zero).




Entry point table of Character writer functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- S: For cycle counts see function's description.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE10C |           130 | 5 |      | us_cwr_cbyte_new                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE10E |           150 | 5 |      | us_cwr_cbyte_newz                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE110 |       180 / S | 3 |      | us_cwr_cbyte_setnc                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE112 |            80 | 1 |  X3  | us_cwr_cbyte_nextsi                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE114 |           130 | 6 |      | us_cwr_pbyte_new                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE116 |           170 | 6 |      | us_cwr_pbyte_newz                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE118 |       180 / S | 3 |      | us_cwr_pbyte_setnc                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE11A |           100 | 1 |  X3  | us_cwr_pbyte_nextsi                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE11C |           110 | 3 |      | us_cwr_cutf8_new                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE11E |           130 | 3 |      | us_cwr_cutf8_newz                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE120 |     180 / 500 | 3 |      | us_cwr_cutf8_setnc                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE122 |            80 | 1 |  X3  | us_cwr_cutf8_nextsi                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE124 |           110 | 4 |      | us_cwr_putf8_new                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE126 |           150 | 4 |      | us_cwr_putf8_newz                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE128 |     180 / 500 | 3 |      | us_cwr_putf8_setnc                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE12A |           100 | 1 |  X3  | us_cwr_putf8_nextsi                    |
+--------+---------------+---+------+----------------------------------------+
