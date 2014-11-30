
RRPGE User Library - Mathematics
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The mathematics component adds some commonly used or costly to implement math
related functions. These have no state, however some of them use some CPU RAM
areas populated as defined in "data.rst":

- Large sine table at 0xFE00 - 0xFFFF
- Musical logarithmic table at 0xFC00 - 0xFDFF




Functions
------------------------------------------------------------------------------


0xF080: Sine of high precision angle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sin
- Cycles: 100
- Param0: Angle
- Ret.X3: Sine in signed 2's complement 2.14 fixed point

The angle uses the full 16 bit range to represent all the possible angles, so
giving about 7-8 fraction bits for an angle in degrees. The result comes in
the format the sine table provides it. Between the table entries coarse linear
interpolation is used.


0xF082: Cosine of high precision angle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_cos
- Cycles: 100
- Param0: Angle
- Ret.X3: Cosine in signed 2's complement 2.14 fixed point

The angle uses the full 16 bit range to represent all the possible angles, so
giving about 7-8 fraction bits for an angle in degrees. The result comes in
the format the sine table provides it. Between the table entries coarse linear
interpolation is used.


0xF084: Sine & Cosine of high precision angle
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sincos
- Cycles: 220
- Param0: Angle
- Ret. C: Cosine in signed 2's complement 2.14 fixed point
- Ret.X3: Sine in signed 2's complement 2.14 fixed point

The angle uses the full 16 bit range to represent all the possible angles, so
giving about 7-8 fraction bits for an angle in degrees. The result comes in
the format the sine table provides it. Between the table entries coarse linear
interpolation is used.


0xF086: Frequency of tone
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_tfreq
- Cycles: 50 / 140
- Param0: Tone
- Ret. C: High (whole) part of frequency
- Ret.X3: Low (fractional) part of frequency

Returns the frequency (add value) to be used for outputting a tone with the
Mixer. The high 8 bits of the tone parameter are "whole" semitones (as in
twelve tone equal temperament). The low 8 bits are a fractional part which may
be used for frequency sweeping. "Whole" semitones are returned directly from
the Musical logarithmic table (50 cycles), fractions are calculated using
linear interpolation (140 cycles).


0xF088: 32 bit multiply
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_mul32
- Cycles: 100
- Param0: Operand 1, high
- Param1: Operand 1, low
- Param2: Operand 2, high
- Param3: Operand 2, low
- Ret. C: Result, high
- Ret.X3: Result, low

Calculates C:X3 = (P0:P1) * (P2:P3)


0xF08A: 32 bit divide
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_div32
- Cycles: 600
- Param0: Operand 1, high
- Param1: Operand 1, low
- Param2: Operand 2, high
- Param3: Operand 2, low
- Ret. C: Result, high
- Ret.X3: Result, low

Calculates C:X3 = (P0:P1) / (P2:P3). The result is 0 for division by zero.


0xF08C: 16 bit reciprocal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_rec16
- Cycles: 70
- Param0: Number to take reciprocal of
- Ret. C: Remainder
- Ret.X3: Result

Calculates X3 = 0x10000 / P0. The result and the remainder are both 0 for
division by zero or if the number is 1.


0xF08E: 32 bit reciprocal
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_rec32
- Cycles: 470
- Param0: Number to take reciprocal of, high
- Param1: Number to take reciprocal of, low
- Ret. C: Result, high
- Ret.X3: Result, low

Calculates C:X3 = 0x100000000 / (P0:P1). The result is 0 for division by zero
or if the number is 1.

Note that to meet the cycle requirement using only the features of RRPGE as
defined in this specification, a complex algorithm has to be implemented.
Details on this algorithm may be found in the reference User Library
implementation.


0xF090: 16 bit square root
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sqrt16
- Cycles: 260
- Param0: Number to take square root of
- Ret.X3: Result

Calculates X3 = sqrt(P0).


0xF092: 32 bit square root
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_sqrt32
- Cycles: 650
- Param0: Number to take square root of, high
- Param1: Number to take square root of, low
- Ret.X3: Result

Calculates X3 = sqrt(P0:P1).




Entry point table of Mathematics functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xF080 |           100 | 1 |  X3  | us_sin                                 |
+--------+---------------+---+------+----------------------------------------+
| 0xF082 |           100 | 1 |  X3  | us_cos                                 |
+--------+---------------+---+------+----------------------------------------+
| 0xF084 |           220 | 1 | C:X3 | us_sincos                              |
+--------+---------------+---+------+----------------------------------------+
| 0xF086 |      50 / 140 | 1 | C:X3 | us_tfreq                               |
+--------+---------------+---+------+----------------------------------------+
| 0xF088 |           100 | 4 | C:X3 | us_mul32                               |
+--------+---------------+---+------+----------------------------------------+
| 0xF08A |           600 | 4 | C:X3 | us_div32                               |
+--------+---------------+---+------+----------------------------------------+
| 0xF08C |            70 | 1 | C:X3 | us_rec16                               |
+--------+---------------+---+------+----------------------------------------+
| 0xF08E |           470 | 2 | C:X3 | us_rec32                               |
+--------+---------------+---+------+----------------------------------------+
| 0xF090 |           260 | 1 |  X3  | us_sqrt16                              |
+--------+---------------+---+------+----------------------------------------+
| 0xF092 |           650 | 2 |  X3  | us_sqrt32                              |
+--------+---------------+---+------+----------------------------------------+
