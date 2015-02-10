
RRPGE User Library - Unicode assistance functions
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The functions in this component provide basic assistance for working with
Unicode character data, converting between UTF-8 and UTF-32, and mapping
Unicode characters to identifiers (useful for character output by a limited
character set).




Functions
------------------------------------------------------------------------------


0xE0E8: Convert UTF-8 character to UTF-32
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_utf32f8
- Cycles: 40 / 200
- Param0: UTF-8 first byte
- Param1: UTF-8 second byte
- Param2: UTF-8 third byte
- Param3: UTF-8 fourth byte
- Ret. C: UTF-32 character, high
- Ret.X3: UTF-32 character, low

Converts a UTF-8 sequence to a UTF-32 character. Variable number of parameters
are accepted (second, third, and fourth bytes may be omitted). The first
parameter must be the first character of a valid UTF-8 sequence.

If the UTF-8 sequence is invalid (or would be longer than 4 bytes), zero is
returned.

For 7 bit ASCII inputs, 40 cycles are taken. Other cases up to 200 cycles may
be taken (more for longer sequences).


0xE0EA: Converts UTF-32 character to UTF-8
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_utf8f32
- Cycles: 50 / 120
- Param0: UTF-32 character, high
- Param1: UTF-32 character, low
- Ret. C: UTF-8 sequence bytes, high
- Ret.X3: UTF-8 sequence bytes, low

Converts a UTF-32 character to a UTF-8 sequence. The return value is populated
from bottom to up, that is, an ASCII-7 return will pass in the low 7 bits of
X3 (the rest are zero), and for example a 3 byte UTF-8 will get the first byte
on the low 8 bits of C, the second byte on the high 8 bits of X3, and the
third byte on the low 8 bits of X3.

For 7 bit ASCII inputs, 50 cycles are taken. Other cases up to 120 cycles may
be taken (more for longer sequences).


0xE0EC: Get UTF-8 length of UTF-32 character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_utf8len
- Cycles: 60
- Param0: UTF-32 character, high
- Param1: UTF-32 character, low
- Ret.X3: Length of UTF-8 sequence

Returns the length of a UTF-32 character if represented in UTF-8. Returns up
to 4, if the UTF-32 character is larger than what can be converted to 4 UTF-8
bytes, returns zero (for the UTF-32 input of zero, the return value is 1).


0xE0EE: Converts UTF-32 to identifier by table
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_idfutf32
- Cycles: 40 / S
- Param0: PRAM word pointer of conversion table, high
- Param1: PRAM word pointer of conversion table, low
- Param2: UTF-32 character, high
- Param3: UTF-32 character, low
- Ret.X3: Conversion value

Converts a UTF-32 input character to an identifier by table. For ASCII-7
values (UTF-32 character is less than 128) the return is direct (table is not
used), otherwise the table is used for converting.

The format of the table:

- 1 word: Unknown character's conversion value
- 1 word: Number of entries in the table
- 3 words / entry: Conversion entries in ascending order. First two words are
  the UTF-32 source (Big Endian), last word is the conversion value.

The function performs a logarithmic search in the table, finding the
conversion value or it's absence in log2 iterations relative to the table's
length (count of entries). One iteration takes 80 cycles, and there are 200
cycles of overhead in addition to the search. If the input is an ASCII-7
character, the return is produced in 40 cycles.

A zero length table is allowed (which table is 2 words long). This case the
return for non-ASCII-7 characters is the unknown character's conversion
value.




Entry point table of Mathematics functions
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
| 0xE0E8 |      40 / 200 | 4 | C:X3 | us_utf32f8                             |
+--------+---------------+---+------+----------------------------------------+
| 0xE0EA |      50 / 120 | 2 | C:X3 | us_utf8f32                             |
+--------+---------------+---+------+----------------------------------------+
| 0xE0EC |            60 | 2 |  X3  | us_utf8len                             |
+--------+---------------+---+------+----------------------------------------+
| 0xE0EE |        40 / S | 4 |  X3  | us_idfutf32                            |
+--------+---------------+---+------+----------------------------------------+
