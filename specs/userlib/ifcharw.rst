
RRPGE User Library - Character writer interfaces
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, data structures
------------------------------------------------------------------------------


The character writer interfaces provide a common solution for writing
character data, supporting up to 32 bits wide source values as characters.

Most commonly it may be used to consume an UTF-32 stream to generate an
arbitrary destination format, but other uses are possible.

The object structure is as follows:

- Word0: Set next character function implementation
- Word1: Initialize for output function implementation
- Word2: Set output style function implementation

The initialize for output and the set output style functions not necessarily
have to be provided. If not, they have to be loaded with zeros, so the
respective handlers will do nothing.

An extended character writer interface is also provided for supporting targets
which may be read back later using an appropriate reader. The structure of the
extended interface introduces the following member:

- Word3: Get next index function implementation

The get next index function is meant for setting up the next suitable index
for later reader access (with the us_cr_setsi function), and return that.




Functions
------------------------------------------------------------------------------


0xE0DC: Set up character writer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_new
- Cycles: 80
- Param0: Character writer structure pointer
- Param1: Set next character function implementation
- Param2: Initialize for output function implementation
- Param3: Set output style function implementation
- Ret.X3: Character writer structure pointer + 3

Sets up a character writer interface structure using the given parameters
as-is. This should be called by character writer implementations in their own
setup ("new") functions, to add the appropriate handlers in the object.

Parameter 3, or both Parameter 2 and parameter 3 may be omitted, leaving the
Set output style and the Initialize for output functions unimplemented (same
effect like providing zero on these locations).


0xE0DE: Set next character
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_setnc
- Cycles: 15 + F
- Param0: Character writer structure pointer
- Param1: Character value to set, high
- Param2: Character value to set, low

Calls the set next character function of the character writer.

The set next character function can take three parameters, the same provided
to this function. No return value is accepted from it.


0xE0E0: Set output style
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_setst
- Cycles: 30 + F
- Param0: Character writer structure pointer
- Param1: Style attribute to set
- Param2: Style value to set

Calls the set output style function of the character writer.

The Style value to set parameter may be omitted (Param2), which case the
previous or a default style should be set for the passed attribute, depending
on the implementation.

The set output style function may be left unimplemented which case this
function does nothing. Otherwise the implementation gets the parameters as-is,
and no return value is accepted from it.


0xE0E2: Initialize for output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cw_init
- Cycles: 30 + F
- Param0: Character writer structure pointer

Calls the initialize for output function of the character writer.

This function should set up for producing a stream of characters (a sequence
of us_cw_setnc calls, with optionally some us_cw_setst). Normally this should
mean the initialization of the Accelerator for writing characters onto a
graphics surface.

The initialize for output function may be left unimplemented which case this
function does nothing. Otherwise the implementation gets the parameters as-is,
and no return value is accepted from it.


0xE0E4: Set up extended character writer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_new
- Cycles: 110
- Param0: Character writer structure pointer
- Param1: Set next character function implementation
- Param2: Initialize for output function implementation
- Param3: Set output style function implementation
- Param4: Get next index function implementation
- Ret.X3: Character writer structure pointer + 4

Sets up an extended character writer interface structure using the given
parameters as-is. This should be called by character writer implementations in
their own setup ("new") functions, to add the appropriate handlers in the
object.

Parameter 3, or both Parameter 2 and parameter 3 may be omitted, leaving the
Set output style and the Initialize for output functions unimplemented (same
effect like providing zero on these locations). The Get next index function
is always the last parameter.


0xE0E6: Get next index
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cwr_nextsi
- Cycles: 20 + F
- Param0: Character writer structure pointer
- Ret.X3: Next index for the reader

Calls the get next index function of the character writer.

This function should produce a next index suitable for an appropriate
character reader by whatever means necessary, and return it. Subsequent writes
to the character writer should expand the string at this index.

The get next index function can take one parameter, the character writer
structure pointer, which is always provided. The return value is passed back
as-is.




Entry point table of Character writer interface functions
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
| 0xE0DC |            80 | 4 |  X3  | us_cw_new                              |
+--------+---------------+---+------+----------------------------------------+
| 0xE0DE |        15 + F | 3 |      | us_cw_setnc                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0E0 |        30 + F | 3 |      | us_cw_setst                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE0E2 |        30 + F | 1 |      | us_cw_init                             |
+--------+---------------+---+------+----------------------------------------+
| 0xE0E4 |           110 | 5 |  X3  | us_cwr_new                             |
+--------+---------------+---+------+----------------------------------------+
| 0xE0E6 |        20 + F | 1 |  X3  | us_cwr_nextsi                          |
+--------+---------------+---+------+----------------------------------------+
