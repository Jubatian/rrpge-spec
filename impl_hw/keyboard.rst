
RRPGE Keyboard
==============================================================================

.. image:: https://cdn.rawgit.com/Jubatian/rrpge-spec/00.013.002/logo_txt.svg
   :align: center
   :width: 100%

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


This document provides an overview on how a physical RRPGE keyboard should
look like, and how it should be implemented.

In overall the RRPGE keyboard aims for simplicity, to be compact without many
buttons, yet offering the features of a normal keyboard. Some of the design
elements were taken from microcomputers of the 80's, particularly the
Commodore 64 and the Enterprise 128K, integrating these while considering some
of today's needs. The layout of most alternate characters follow that of a
normal Hungarian keyboard which was more fitting due to the placement of zero.

The presented keyboard should be used on a microcomputer realization having a
similar form to the aforementioned machines (the whole system integrated
within a keyboard).




Keyboard layout
-----------------------------------------------------------------------------


The overall shape of the keyboard and the naming of the buttons should
follow the layout guide below (and on the keyboard.png image): ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | + | - | / | * |BKS|
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    | TAB | Q | W | E | R | T | Y | U | I | O | P | [ | ] |     ||INS|   |ESC|
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    | SHLC | A | S | D | F | G | H | J | K | L | : | ; | ENTER  ||DEL|   |FN |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    | SHIFT  | Z | X | C | V | B | N | M | , | . | = |  SHIFT   |    | A |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |  CTRL  |  ALT   |          SPACE        |  ALT   |  CTRL  || < | V | > |
    +--------+--------+-----------------------+--------+--------++---+---+---+

The keys marked on the following chart also have international roles: ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    |   |   |   |   |   |   |   |   |   |   | X | X | X | X |   |
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    |     |   |   |   |   |   |   |   |   |   |   | X | X |     ||   |   |   |
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    |      |   |   |   |   |   |   |   |   |   | X | X |        ||   |   |   |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    |        |   |   |   |   |   |   |   | X | X | X |          |    |   |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |        |        |                       |        |        ||   |   |   |
    +--------+--------+-----------------------+--------+--------++---+---+---+

These 11 keys normally should not produce any text input, their naming is
rather after their ALT codes. Their normal and SHIFT function is reserved for
producing international characters, depending on the target language. They
should be sufficient to serve most languages using Latin alphabet.

The purpose of this design is that so international variations of the keyboard
can remain position compatible with each other, especially regarding the
location of non-alphanumeric characters (accessible using ALT or SHIFT).

Normal function (no ALT, no SHIFT): ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |   |   |   |   |BKS|
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    | TAB | q | w | e | r | t | y | u | i | o | p |   |   |     ||INS|   |ESC|
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    | SHLC | a | s | d | f | g | h | j | k | l |   |   | ENTER  ||DEL|   |FN |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    | SHIFT  | z | x | c | v | b | n | m |   |   |   |  SHIFT   |    | A |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |  CTRL  |  ALT   |          SPACE        |  ALT   |  CTRL  || < | V | > |
    +--------+--------+-----------------------+--------+--------++---+---+---+

SHIFT function: ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | § | ' | " | ? | ! | % |   |   | ( | ) |   |   |   |   |BKS|
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    | TAB | Q | W | E | R | T | Y | U | I | O | P |   |   |     ||INS|   |ESC|
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    | SHLC | A | S | D | F | G | H | J | K | L |   |   | ENTER  ||DEL|   |FN |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    | SHIFT  | Z | X | C | V | B | N | M |   |   |   |  SHIFT   |    | A |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |  CTRL  |  ALT   |          SPACE        |  ALT   |  CTRL  || < | V | > |
    +--------+--------+-----------------------+--------+--------++---+---+---+

ALT function: ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    |   | ~ |   | ^ |   | ° |   | ` | < | > | + | - | / | * |BKS|
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    | TAB | \ | | |   |   |   |   |   | $ |   |   | [ | ] |     ||INS|   |ESC|
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    | SHLC |   |   |   |   |   |   |   |   |   | : | ; | ENTER  ||DEL|   |FN |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    | SHIFT  |   | # | & | @ | { | } | _ | , | . | = |  SHIFT   |    | A |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |  CTRL  |  ALT   |          SPACE        |  ALT   |  CTRL  || < | V | > |
    +--------+--------+-----------------------+--------+--------++---+---+---+

Holding down the FN key provides function keys on the top (numeric) row, from
F0 to F14, and provides Page Up, Page Down, Home and End on the appropriate
directional keys.




Keyboard matrix
------------------------------------------------------------------------------


The keyboard matrix is designed so using a keyfoil the keyboard can still
substitute a gamepad without problem, having the keys serving for gamepad
controls on one row (allowing the detection of any combination). These keys
are selected in such a manner that they are also conveniently accessible for
playing games.

The "keyboard.png" image provides the conceptual layout of this matrix, its
original GIMP source ("keyboard.xcf") is also provided in which the keyboard,
the columns and the rows are accessible as separate layers.

Row 6, column 5 is reserved, later it may service a button placed between
DEL and FN.

The complete matrix is laid out as follows:

+----+--------+--------+--------+--------+--------+--------+--------+--------+
|    | 0      | 1      | 2      | 3      | 4      | 5      | 6      | 7      |
+====+========+========+========+========+========+========+========+========+
| 0  | 0      | 2      | 4      | 6      | 8      | +      | /      | BKS    |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 1  | Q      | 1      | 3      | 5      | 7      | 9      | -      | *      |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 2  | A      | W      | R      | Y      | U      | O      | [      | ]      |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 3  | S      | E      | T      | H      | J      | I      | P      | ;      |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 4  | Z      | D      | F      | G      | K      | L      | :      | =      |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 5  | X      | C      | V      | B      | M      | ,      | .      | N      |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 6  | TAB    | SHLOCK | FN     | INS    | DEL    |        | ENTER  | SHIFT  |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
| 7  | CTRL   | SPACE  | ESC    | UP     | RIGHT  | DOWN   | LEFT   | ALT    |
+----+--------+--------+--------+--------+--------+--------+--------+--------+
