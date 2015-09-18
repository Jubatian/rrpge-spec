
RRPGE constant data blocks
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, the types and roles of constant data in RRPGE
------------------------------------------------------------------------------


The RRPGE system provides some important blocks of constant data for the
convenience of application developers. Most notably these cover the following
fields:

- Palette data providing an useful basic color set.

- Small and large sine tables for use with various algorithms requiring
  trigonometric functions (such as rotations and navigation in a three
  dimensional space).

- Basic waveforms and noise data for composing simple generated audio samples.

- Musical logarithmic table for outputting power of 2 sized audio samples at
  tone frequencies.

In the following chapters these data blocks and their uses are described. On
the end of this document a memory map is provided showing their mapping at
boot.




RRPGE Incremental palette
------------------------------------------------------------------------------


The RRPGE Incremental palette is a basic 4 bit palette laid out in an
incremental manner, that is also providing an useful color set for 1 bit,
2 bit and 3 bit modes.

Following some characteristics are described at each bit depth:


1 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A black and a light gray color is provided. Light gray is chosen to bright
white to reduce contrast and to provide a more useful layout at further bit
depths.


2 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A grayscale ramp is provided where bit 1 is the least significant, and bit 0
is the most significant (so light gray and dark gray are swapped).


3 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Four colors are added to the grayscale ramp laid out so they roughly align by
intensity. Color intensities are chosen so they complement the grayscale ramp
well.


4 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additional colors are selected to cover the color space acceptably, tuned to
select more useful colors. The eight additional colors compared to the 3 bit
palette are laid out aligned with those, roughly representing darker tones of
them except for the blues which are aligned so they produce a ramp using bit 2
and bit 3.


Complete palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As complete, the palette provides 8 sets of 8 colors each, suitable for half
palette selection. The 8 sets are as follows:

- 0: Low half, the 3 bit RRPGE Incremental with colors for a grey ramp.
- 1: High half, complementing for 4 bit RRPGE Incremental, generic palette.
- 2: Low half, alternative of 0 in a brownish tone.
- 3: High half, providing more greens.
- 4: High half, greyscale(ish) ramp.
- 5: High half, providing primarily browns and yellows.
- 6: Sky-blue ramp, suitable to be used for background pattern.
- 7: High half, providing primarily blues.

The palette is designed so either low half can be combined with either high
half to produce a sensible set of colors.


Data dump
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette dump below provides the 64 colors of RRPGE Incremental as 0RGB
4-4-4-4 bit entries, suitable for inclusion in C code: ::

    0x000U, 0xAAAU, 0x555U, 0xFFFU, 0x229U, 0x593U, 0xA42U, 0xEDBU,
    0x35EU, 0x85FU, 0x430U, 0xCC4U, 0x6AFU, 0x073U, 0x730U, 0xA83U,
    0x000U, 0xCA8U, 0x754U, 0xFFCU, 0x134U, 0x783U, 0x953U, 0xFD9U,
    0x351U, 0xC74U, 0x520U, 0x9C5U, 0x7A2U, 0x471U, 0x443U, 0xA76U,
    0x211U, 0x332U, 0x443U, 0x456U, 0x666U, 0x877U, 0x998U, 0xCCCU,
    0xB64U, 0xC87U, 0x331U, 0xCB6U, 0x994U, 0x572U, 0x540U, 0x863U,
    0x38CU, 0x49DU, 0x5AEU, 0x7BFU, 0x9CFU, 0xADFU, 0xCEFU, 0xDFFU,
    0x56BU, 0xA8BU, 0x023U, 0x8CCU, 0x6A8U, 0x366U, 0x34AU, 0x58DU




Large sine table
------------------------------------------------------------------------------


A large sine table for 512 distinct angles is provided. This table contains
2's complement signed values ranging from -0x4000 to 0x4000 (-16384 - 16384),
representing the sine of the angle as a signed .14 fixed point number.

The key values of the sine table are as follows:

- Offset 0x000: 0x0000
- Offset 0x080: 0x4000
- Offset 0x100: 0x0000
- Offset 0x180: 0xC000 (-0x4000)

The first quarter of the table, suitable for inclusion in C code: ::

    0x0000U, 0x00C9U, 0x0192U, 0x025BU, 0x0324U, 0x03EDU, 0x04B5U, 0x057EU,
    0x0646U, 0x070EU, 0x07D6U, 0x089DU, 0x0964U, 0x0A2BU, 0x0AF1U, 0x0BB7U,
    0x0C7CU, 0x0D41U, 0x0E06U, 0x0ECAU, 0x0F8DU, 0x1050U, 0x1112U, 0x11D3U,
    0x1294U, 0x1354U, 0x1413U, 0x14D2U, 0x1590U, 0x164CU, 0x1709U, 0x17C4U,
    0x187EU, 0x1937U, 0x19EFU, 0x1AA7U, 0x1B5DU, 0x1C12U, 0x1CC6U, 0x1D79U,
    0x1E2BU, 0x1EDCU, 0x1F8CU, 0x203AU, 0x20E7U, 0x2193U, 0x223DU, 0x22E7U,
    0x238EU, 0x2435U, 0x24DAU, 0x257EU, 0x2620U, 0x26C1U, 0x2760U, 0x27FEU,
    0x289AU, 0x2935U, 0x29CEU, 0x2A65U, 0x2AFBU, 0x2B8FU, 0x2C21U, 0x2CB2U,
    0x2D41U, 0x2DCFU, 0x2E5AU, 0x2EE4U, 0x2F6CU, 0x2FF2U, 0x3076U, 0x30F9U,
    0x3179U, 0x31F8U, 0x3274U, 0x32EFU, 0x3368U, 0x33DFU, 0x3453U, 0x34C6U,
    0x3537U, 0x35A5U, 0x3612U, 0x367DU, 0x36E5U, 0x374BU, 0x37B0U, 0x3812U,
    0x3871U, 0x38CFU, 0x392BU, 0x3984U, 0x39DBU, 0x3A30U, 0x3A82U, 0x3AD3U,
    0x3B21U, 0x3B6DU, 0x3BB6U, 0x3BFDU, 0x3C42U, 0x3C85U, 0x3CC5U, 0x3D03U,
    0x3D3FU, 0x3D78U, 0x3DAFU, 0x3DE3U, 0x3E15U, 0x3E45U, 0x3E72U, 0x3E9DU,
    0x3EC5U, 0x3EEBU, 0x3F0FU, 0x3F30U, 0x3F4FU, 0x3F6BU, 0x3F85U, 0x3F9CU,
    0x3FB1U, 0x3FC4U, 0x3FD4U, 0x3FE1U, 0x3FECU, 0x3FF5U, 0x3FFBU, 0x3FFFU

Note that the quarter can not simply be doubly-mirrored to produce the full
sine table, see the key values for guides. To mirror by value, the value has
to be subtracted from 0. The following guides may be used to confirm proper
generation:

- Offset 0x001: 0x00C9
- Offset 0x081: 0x3FFF
- Offset 0x101: 0xFF37 (-0x00C9)
- Offset 0x181: 0xC001 (-0x3FFF)




Waveform data
------------------------------------------------------------------------------


Eight 256 byte samples and their reduced variants are provided mostly for use
in simple audio tasks. Note that the samples are stored in Big Endian byte
order, conforming with the way the components access memory.


Square wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Offset 0x00 - 0x7F: 0xFF
- Offset 0x80 - 0xFF: 0x00


Sine wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Uses the following sine table for the first quarter (offsets 0x00 - 0x3F): ::

    0x81U, 0x84U, 0x87U, 0x8AU, 0x8EU, 0x91U, 0x94U, 0x97U,
    0x9AU, 0x9DU, 0xA0U, 0xA3U, 0xA6U, 0xA9U, 0xACU, 0xAFU,
    0xB2U, 0xB5U, 0xB8U, 0xBAU, 0xBDU, 0xC0U, 0xC3U, 0xC5U,
    0xC8U, 0xCAU, 0xCDU, 0xCFU, 0xD2U, 0xD4U, 0xD6U, 0xD9U,
    0xDBU, 0xDDU, 0xDFU, 0xE1U, 0xE3U, 0xE5U, 0xE7U, 0xE9U,
    0xEBU, 0xECU, 0xEEU, 0xEFU, 0xF1U, 0xF2U, 0xF4U, 0xF5U,
    0xF6U, 0xF7U, 0xF8U, 0xF9U, 0xFAU, 0xFBU, 0xFCU, 0xFCU,
    0xFDU, 0xFEU, 0xFEU, 0xFEU, 0xFFU, 0xFFU, 0xFFU, 0xFFU

The remaining three quarters can be produced by appropriately double-mirroring
this table. To mirror by value, the value has to be subtracted from 0xFF. The
following guides may be used to confirm proper generation:

- Offset 0x08: 0x9A
- Offset 0x48: 0xFC
- Offset 0x88: 0x65
- Offset 0xC8: 0x03


Triangle wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Begins with 0x80, incrementing to match the waveform of the sine wave. The
following key offsets define it:

- Offset 0x00: 0x80
- Offset 0x01: 0x82
- Offset 0x3F: 0xFE
- Offset 0x40: 0xFF
- Offset 0x41: 0xFD
- Offset 0xBF: 0x01
- Offset 0xC0: 0x00
- Offset 0xC1: 0x02
- Offset 0xFF: 0x7E


Spiked wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This waveform is produced from composing the quarters of the sine wave in the
following way:

- Offset 0x00 - 0x3F: Quarter 3, values incremented by 0x80.
- Offset 0x40 - 0x7F: Quarter 2, values incremented by 0x80.
- Offset 0x80 - 0xBF: Quarter 1, values decremented by 0x80.
- Offset 0xC0 - 0xBF: Quarter 0, values decremented by 0x80.


Incremental sawtooth
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Begins with 0x00, for each offset increment increasing by one until reaching
0xFF.


Decremental sawtooth
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Begins with 0xFF, for each offset increment decreasing by one until reaching
0x00.


Noise 1, Noise 2
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Generated using the following pseudorandom number generator ("5 byte PRNG"):

val = (((val >> 7) + (val << 1) + num1) ^ num2) & 0xFFU;

The generation starts with val = 0 at offset 0x00. The first output of the
algorithm appears at offset 0x01. The generator produces a pseudo random
permutation of all 256 possible values, so each value appears exactly once in
the sample.

For Noise 1 the parameters are:

- num1: 0xBB
- num2: 0x7F

For Noise 2 the parameters are:

- num1: 0xA3
- num2: 0xB3


Reductions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The reductions take place in the subsequent 256 bytes for each sample, to
support generating higher frequency variants. The following layout is used in
this area:

- 0x100 - 0x17F: 1/2 reduction (128 bytes)
- 0x180 - 0x1BF: 1/4 reduction (64 bytes)
- 0x1C0 - 0x1DF: 1/8 reduction (32 bytes)
- 0x1E0 - 0x1EF: 1/16 reduction (16 bytes)
- 0x1F0 - 0x1F7: 1/32 reduction (8 bytes)
- 0x1F8 - 0x1FB: 1/64 reduction (4 bytes)
- 0x1FC - 0x1FD: 1/128 reduction (2 bytes)
- 0x1FE - 0x1FF: 1/128 reduction (2 bytes)

Each reduction level is generated from the previous level by the following
rules:

- 2 consequent samples are averaged to produce a result sample.
- Rounding is upwards if the sum was larger than 0x100, downwards otherwise.




Musical logarithmic table
------------------------------------------------------------------------------


The Musical logarithmic table is meant to be used with the Audio mixer to
assist in outputting power of 2 sized samples at given musical frequencies.

If a 256 byte sample is played as-is at 48KHz, assuming it contains one
period, the resulting audible frequency is 187.5Hz. The Musical logaritmic
table contains 12 values which can be used to generate frequencies within an
octave below this frequency. These are as follows:

+-------+--------+-----------------------------------------------------------+
| Index | Value  | Note & Frequency (for 256 byte sample)                    |
+=======+========+===========================================================+
| 0     | 0x85CD | G2;   97.999Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 1     | 0x8DC2 | G#2; 103.826Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 2     | 0x9630 | A2;  110.000Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 3     | 0x9F1E | A#2; 116.541Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 4     | 0xA894 | B2;  123.471Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 5     | 0xB29A | C3;  130.813Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 6     | 0xBD39 | C#3; 138.591Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 7     | 0xC87A | D3;  146.832Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 8     | 0xD465 | D#3; 155.563Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 9     | 0xE107 | E3;  164.814Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 10    | 0xEE68 | F3;  174.614Hz                                            |
+-------+--------+-----------------------------------------------------------+
| 11    | 0xFC95 | F#3; 184.997Hz                                            |
+-------+--------+-----------------------------------------------------------+




Memory maps
------------------------------------------------------------------------------


The data blocks appear in two places within the user accessible memories: One
is the high end of the CPU Data memory (which may be overwritten with
application data if in the Application Header a large enough initial data is
specified), the other is the high end of the Peripheral RAM.


CPU data memory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFAB4 |                                                                   |
| \-     | Musical logarithmic table (up_mlogt).                             |
| 0xFABF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAC0 |                                                                   |
| \-     | RRPGE Incremental palette (up_palette).                           |
| 0xFAFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFB00 |                                                                   |
| \-     | User Library data (see "userlib/ulboot.rst").                     |
| 0xFDFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE00 |                                                                   |
| \-     | Large sine table (up_sine).                                       |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+


Peripheral RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Peripheral RAM the offsets specified are within the highest 64 KCell
bank, as 32 bit offsets. That is the base of these offsets is 0xF0000.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFC00 |                                                                   |
| \-     | Square wave (up_smp_sqr).                                         |
| 0xFC7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFC80 |                                                                   |
| \-     | Sine wave (up_smp_sine).                                          |
| 0xFCFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD00 |                                                                   |
| \-     | Triangle wave (up_smp_tri).                                       |
| 0xFD7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFD80 |                                                                   |
| \-     | Spiked wave (up_smp_spike).                                       |
| 0xFDFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE00 |                                                                   |
| \-     | Incremental sawtooth (up_smp_sawi).                               |
| 0xFE7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE80 |                                                                   |
| \-     | Decremental sawtooth (up_smp_sawd).                               |
| 0xFEFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFF00 |                                                                   |
| \-     | Noise 1 (up_smp_nois1).                                           |
| 0xFF7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFF80 |                                                                   |
| \-     | Noise 2 (up_smp_nois2).                                           |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+
