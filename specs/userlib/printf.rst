
RRPGE User Library - Printf and string transfer
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The printf function realizes a similar functionality like standard C library
printf in a limited manner, without implementing floating point. It, however,
adds support for styling the output.

The component also adds a simple string copy function, which is similar to the
strcpy function in a standard C library.

These functions operate by character readers and writers, so the character
source and destination can be controlled freely including providing specific
implementations. With these, the string copy may also be used to convert
between formats.

Functions are provided with optional zero termination support, so they can be
used to either produce ordinary zero terminated strings or used in contexts
where outputting such a termination character is undesirable.

Note that this documentation occasionally refers to UTF (Unicode
Transformation Format), however it is not mandatory to have character formats
conforming it: there is no reliance on any character value beyond ASCII-7.




Printf format string specification
------------------------------------------------------------------------------


The printf function accepts format strings similar to the standard C library
printf. Notably, it can be used to produce decimal or hexadecimal output both
at 16 or 32 bits.


Special characters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Special characters may be output using a '\\' character (a singe ASCII 0x5C).
The following codes are recognized:

- '\\\\': Outputs a single '\\' (ASCII 0x5C).
- '\\"': Outputs a '"' (ASCII 0x22).
- '\\n': Outputs a new line (ASCII 0x0A).
- '\\r': Outputs a carriage return (ASCII 0x0D).
- '\\t': Outputs a horizontal TAB (ASCII 0x09).
- '\\b': Outputs a backspace (ASCII 0x08).

If any other character is provided after the '\\', no output is produced.


Format strings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The format strings begin with a '%' (ASCII 0x25). A '%%' sequence produces a
single '%' character. Invalid format strings produce no output.

The format string may have up to four components which must occur in the order
listed below:

- Optional flags specifying justification, sign and padding.
- Optional width specifier providing a minimal width for the output.
- Optional long specifier indicating a 32 bit source.
- Format specifier (must be present).

The flags are optional, and any combination of them in any order may be
present as long as no flag is duplicated. The following flags are available:

- '-': Left justify the content (instead of right).
- '+': Add positive sign. Note that the number zero will still lack a sign.
- ' ': Insert a space at the place of sign if the number has no sign.
- '0': Pad the number with zeroes on the left.

Left justify is ignored if zero padding is also specified.

The width specifier provides the minimal width in characters the number should
occupy. It will only take more characters if the number's digits and sign
combined (including the ' ' flag's effect if specified and applies) exceed it.
The width specifier can have arbitrary number of digits to specify larger
widths.

The long specifier is a single 'l' character which may be supplied to indicate
that a 32 bit parameter is provided. The 32 bit parameter is formed from two
16 bit function parameters, with the first of them providing the higher bits
(Big Endian).

Available format specifiers are as follows:

- 's': String. Ignores all flags, width specifier and long specifier. Takes
  two function parameters: first a character reader object pointer, then an
  index within it. The string is output using the us_strcpynz function.

- 'c': Character. Ignores all flags and the width specifier, however takes the
  long specifier. Without the long specifier, one parameter is taken and
  output as the low 16 bits of an UTF-32 character (high 16 bits zero), with
  the long specifier present, a full UTF-32 character is taken from two
  parameters (high 16 bits first).

- 'd' or 'i': Decimal (Integer). A signed 16 or 32 bit integer, 2's complement
  form. All flags and specifiers have their appropriate effects.

- 'u': Unsigned decimal. All flags and specifiers have their appropriate
  effects.

- 'x': Unsigned lowercase hexadecimal. All flags and specifiers have their
  appropriate effects.

- 'X': Unsigned uppercase hexadecimal. All flags and specifiers have their
  appropriate effects.


Attributes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Attributes are an extension to the ordinary printf functionality, not found in
C standard libraries. They are meant to support altering the appearance of the
output, such as by text color, styling and font face. Their function depends
on the used character writer's properties: the us_cw_setst function is used
when an attribute is processed.

Attributes are provided by the '$' (ASCII 0x24) character. '$$' produces a
single '$'. Any other character specifies an attribute character, which is
supplied as the second parameter of us_cw_setst.

The next characters may either form a decimal number to send as attribute
value, or a format specifier, one of 'd', 'i', 'u', 'x', 'X', or 'c',
indicating the format of the attribute value provided afterwards. Note that
'x' and 'X' have the same effect, both accepting a hexadecimal value of
arbitrary case. Negative numbers are not accepted, neither any sign (neither
'+' or '-').

The attribute is finished with a ';' character.

Malformed attributes produce no output, and no attribute set.




Functions
------------------------------------------------------------------------------


0xE140: Copy string with no zero termination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_strcpynz
- Cycles: 4U + 25 + Character reader & writer operations
- Param0: Target character writer pointer
- Param1: Source character reader pointer
- Param2: String index in character reader

Sets the provided string index (us_cr_setsi), then copies characters into the
provided character writer from the reader until the reader returns a null
character. The us_cr_getnc and the us_cw_setnc functions are used for the
copy. The target string does not receive a terminating zero character.

Note that the us_cw_init function is also called to initialize the character
writer.


0xE142: Copy string with zero termination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_strcpy
- Cycles: 4U + 50 + Character reader & writer operations
- Param0: Target character writer pointer
- Param1: Source character reader pointer
- Param2: String index in character reader

Sets the provided string index (us_cr_setsi), then copies characters into the
provided character writer from the reader until the reader returns a null
character. The us_cr_getnc and the us_cw_setnc functions are used for the
copy. The target receives a terminating zero character.

Note that the us_cw_init function is also called to initialize the character
writer.


0xE144: Printf with no zero termination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_printfnz
- Cycles: 30U + 100, depends on format strings
- Param0: Target character writer pointer
- Param1: Source character reader pointer
- Param2: String index in character reader
- Param3-: Parameters (up to 13) used depending on format strings occurring.

The printf function normally works similarly to simple string copy, however it
recognizes and evaluates format strings and special sequences described in the
Printf format string specification chapter.

Hexadecimal numeric output takes at about 40 cycles per digit excluding the
us_cw_setnc call.

Decimal numbers may be converted to BCD first. Converting numbers up to 9999
(by absolute value) take up to 1300 cycles. Larger numbers take up to about
4000 cycles. Other printf functionality has significantly less overhead per
character.


0xE146: Printf with zero termination
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_printf
- Cycles: 30U + 100, depends on format strings
- Param0: Target character writer pointer
- Param1: Source character reader pointer
- Param2: String index in character reader
- Param3-: Parameters (up to 13) used depending on format strings occurring.

Same as us_printfnz except that it outputs a terminating zero character.




Entry point table of Printf and string transfer functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- F: Additional callback cycles.
- S: For cycle counts see function's description.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE140 |   4U + 25 + F | 3 |      | us_strcpynz                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE142 |   4U + 50 + F | 3 |      | us_strcpy                              |
+--------+---------------+---+------+----------------------------------------+
| 0xE144 |             S | 3+|      | us_printfnz                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE146 |             S | 3+|      | us_printf                              |
+--------+---------------+---+------+----------------------------------------+
