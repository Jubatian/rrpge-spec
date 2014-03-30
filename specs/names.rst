
RRPGE name and ID formatting specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction, the usage of names
------------------------------------------------------------------------------


In the RRPGE system there are two important locations where special long IDs
are used. These are the 8 word user ID and the 3 word nonvolatile save ID. The
interpretation of these are outlined in the following chapters.




Common representation conventions
------------------------------------------------------------------------------


The ID should be interpreted as a sequential set of 6 bit units in Big Endian
order (so the first unit is bits 10-15 of the first word). Each such unit's
values represent the following characters:

- 0x00: '_', whitespace, or padding depending on context
- 0x01: '-'
- 0x02 - 0x0B: '0' - '9'
- 0x0C - 0x25: 'a' - 'z'
- 0x26 - 0x3F: 'A' - 'Z'

The 'a' - 'z' and 'A' - 'Z' ranges include the characters in a normal ASCII
set in their usual order.

Further in this document these units are referred as "character units".




Nonvolatile save IDs
------------------------------------------------------------------------------


The 3 words represent 8 character units. The character unit value of 0x00
should be interpreted as '_'. Any combination of the character units compose a
valid nonvolatile save ID.




User IDs
------------------------------------------------------------------------------


The 8 words of the user ID is broken down in the following manner:

- Words 0-5 compose a 16 character unit User Name.
- Word 6 and bits 15-2 of Word 7 compose a 5 character unit extra name.
- Word 7, Bit 1 indicates Artificial Intelligence if set.
- Word 7, Bit 0 indicates Server if set.

Within the 16 character unit User Name the value of 0x00 should be interpreted
as follows:

- Used as the first character of the name, it should show as '_'.
- Used anywhere else it should show as ' '.
- After the last non 0x00 value, it may be treated as padding.
- If all values are 0x00, the first may show as '_', the rest may be padding.

This name should normally be unique, the property of the user using it.

The 5 character unit extra name is an arbitrary local extension for the user
name. When displaying, it should be shown as a separate entity, such as
separated by a '/' from the main User Name.

For displaying the same rules apply for it like the User Name, except that if
all values are 0x00, the extra name should be omitted.

The Artificial Intelligence and Server flags suggest interactive properties of
the particular user which may be used to assist in the usage of network
applications.
