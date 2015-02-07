
RRPGE User Library - Character reader interface
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The character reader interface provides a common solution for reading
character data, supporting up to 32 bits wide return values as characters.

Most commonly it may be used to produce an UTF-32 stream from an arbitrary
source, but other uses are possible.

The object structure is as follows:

- Word0: Get next character function implementation
- Word1: Set index function implementation

The set index function is meant to select a string source within the reader,
the interpretation of this may be arbitrary (for example may be translated
directly to a memory address, but may also use a table for selecting a source
by ID).




Functions
------------------------------------------------------------------------------


0xE0D6: Set up character reader
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_new
- Cycles: 50
- Param0: Character reader structure pointer
- Param1: Set index function implementation
- Param2: Get next character function implementation
- Ret.X3: Character reader structure pointer + 2

Sets up a character reader interface structure using the given parameters
as-is. This should be called by character reader implementations in their own
setup ("new") functions, to add the appropriate handlers in the object.


0xE0D8: Set string index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_setsi
- Cycles: 20 + F
- Param0: Character reader structure pointer

Calls the set string index function of the character reader.

The set string index function can take two parameters, the character reader
structure pointer and the new index to set, which are always provided. No
return value is accepted from it.


0xE0DA: Get next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cr_getnc
- Cycles: 15 + F
- Param0: Character reader structure pointer
- Ret. C: Next character, high
- Ret.X3: Next character, low

Calls the get next character function of the character reader.

The function has to provide the next character, and prepare any internal data
necessary so the next call to this function can return subsequent characters.
If the source is exhausted, it should return zero until a new appropriate
string index is set.

The get next character function can take one parameter, the character reader
structure pointer, which is always provided. The return value is passed back
as-is.




Entry point table of Character reader interface functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- F: Additional callback cycles.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE0D6 |            50 | 3 |  X3  | us_cr_new                              |
+--------+---------------+---+------+----------------------------------------+
| 0xE0D8 |        20 + F | 2 |      | us_cr_setsi                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0DA |        15 + F | 1 | C:X3 | us_cr_getnc                            |
+--------+---------------+---+------+----------------------------------------+
