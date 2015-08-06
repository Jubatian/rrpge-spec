
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
    0x000U, 0x000U, 0x000U, 0x000U, 0x000U, 0x000U, 0x000U, 0x000U




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

    0x0000U, 0x00C9U, 0x0192U, 0x025BU, 0x0323U, 0x03ECU, 0x04B5U, 0x057DU,
    0x0645U, 0x070DU, 0x07D5U, 0x089CU, 0x0964U, 0x0A2AU, 0x0AF1U, 0x0BB6U,
    0x0C7CU, 0x0D41U, 0x0E05U, 0x0EC9U, 0x0F8CU, 0x104FU, 0x1111U, 0x11D3U,
    0x1294U, 0x1354U, 0x1413U, 0x14D1U, 0x158FU, 0x164CU, 0x1708U, 0x17C3U,
    0x187DU, 0x1937U, 0x19EFU, 0x1AA6U, 0x1B5DU, 0x1C12U, 0x1CC6U, 0x1D79U,
    0x1E2BU, 0x1EDCU, 0x1F8BU, 0x2039U, 0x20E7U, 0x2192U, 0x223DU, 0x22E6U,
    0x238EU, 0x2434U, 0x24DAU, 0x257DU, 0x261FU, 0x26C0U, 0x275FU, 0x27FDU,
    0x2899U, 0x2934U, 0x29CDU, 0x2A65U, 0x2AFAU, 0x2B8EU, 0x2C21U, 0x2CB2U,
    0x2D41U, 0x2DCEU, 0x2E5AU, 0x2EE3U, 0x2F6BU, 0x2FF1U, 0x3076U, 0x30F8U,
    0x3179U, 0x31F7U, 0x3274U, 0x32EEU, 0x3367U, 0x33DEU, 0x3453U, 0x34C6U,
    0x3536U, 0x35A5U, 0x3612U, 0x367CU, 0x36E5U, 0x374BU, 0x37AFU, 0x3811U,
    0x3871U, 0x38CFU, 0x392AU, 0x3983U, 0x39DAU, 0x3A2FU, 0x3A82U, 0x3AD2U,
    0x3B20U, 0x3B6CU, 0x3BB6U, 0x3BFDU, 0x3C42U, 0x3C84U, 0x3CC5U, 0x3D02U,
    0x3D3EU, 0x3D77U, 0x3DAEU, 0x3DE2U, 0x3E14U, 0x3E44U, 0x3E71U, 0x3E9CU,
    0x3EC5U, 0x3EEBU, 0x3F0EU, 0x3F2FU, 0x3F4EU, 0x3F6AU, 0x3F84U, 0x3F9CU,
    0x3FB1U, 0x3FC3U, 0x3FD3U, 0x3FE1U, 0x3FECU, 0x3FF4U, 0x3FFBU, 0x3FFEU

Note that the quarter can not simply be doubly-mirrored to produce the full
sine table, see the key values for guides. To mirror by value, the value has
to be subtracted from 0. The following guides may be used to confirm proper
generation:

- Offset 0x001: 0x00C9
- Offset 0x081: 0x3FFE
- Offset 0x101: 0xFF37 (-0x00C9)
- Offset 0x181: 0xC002 (-0x3FFE)




Waveform data
------------------------------------------------------------------------------


Eight 256 byte samples are provided mostly for use in simple audio tasks. Note
that the samples are stored in Big Endian byte order, conforming with the way
the components access memory.


Square wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Offset 0x00 - 0x7F: 0xFF
- Offset 0x80 - 0xFF: 0x00


Sine wave
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Uses the following sine table for the first quarter (offsets 0x00 - 0x3F): ::

    0x81U, 0x84U, 0x87U, 0x8AU, 0x8EU, 0x91U, 0x94U, 0x97U,
    0x9AU, 0x9DU, 0xA0U, 0xA3U, 0xA6U, 0xA9U, 0xACU, 0xAFU,
    0xB2U, 0xB5U, 0xB7U, 0xBAU, 0xBDU, 0xC0U, 0xC2U, 0xC5U,
    0xC8U, 0xCAU, 0xCDU, 0xCFU, 0xD2U, 0xD4U, 0xD6U, 0xD9U,
    0xDBU, 0xDDU, 0xDFU, 0xE1U, 0xE3U, 0xE5U, 0xE7U, 0xE9U,
    0xEAU, 0xECU, 0xEEU, 0xEFU, 0xF1U, 0xF2U, 0xF3U, 0xF5U,
    0xF6U, 0xF7U, 0xF8U, 0xF9U, 0xFAU, 0xFBU, 0xFCU, 0xFCU,
    0xFDU, 0xFDU, 0xFEU, 0xFEU, 0xFFU, 0xFFU, 0xFFU, 0xFFU

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




Musical logarithmic table
------------------------------------------------------------------------------


The Musical logarithmic table is meant to be used with the Audio mixer to
assist in outputting power of 2 sized samples at given musical frequencies. An
A4 (440Hz) for a 256 byte sample at 48KHz can be produced using offset 0x76 in
this table.

The table contains 32 bit 16.16 fixed point sample pointer increments, whole
part first. It's size is 512 words.

There are 12 tones within an octave, the octave above or below may be obtained
by multiplying or dividing the table entries by two respectively. For the
greatest accuracy the top 12 entries are provided (offsets 0xF4 - 0xFF); lower
octaves then may be obtained by right shifting these values one by one for
each until generating the whole table.

The high 12 entries (offsets 0x1E8 - 0x1FF) of the table as 32bit values are
as follows: ::

    55678343U,  58989149U,  62496826U,  66213081U,  70150316U,  74321671U,
    78741067U,  83423255U,  88383859U,  93639437U,  99207528U, 105106715U

Lower octaves must be obtained by right shifting these values, discarding any
one bits falling off on the right.




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
| 0xF800 |                                                                   |
| \-     | Musical logarithmic table.                                        |
| 0xF9FF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFA00 |                                                                   |
| \-     | RRPGE Incremental palette.                                        |
| 0xFA0F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFA10 |                                                                   |
| \-     | Zero.                                                             |
| 0xFAFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFB00 |                                                                   |
| \-     | User Library data (see "userlib/ulboot.rst").                     |
| 0xFDFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE00 |                                                                   |
| \-     | Large sine table.                                                 |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+


Peripheral RAM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the Peripheral RAM the offsets specified are within the highest 64 KCell
bank, as 32 bit offsets. That is the base of these offsets is 0xF0000.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFE00 |                                                                   |
| \-     | Square wave.                                                      |
| 0xFE3F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE40 |                                                                   |
| \-     | Sine wave.                                                        |
| 0xFE7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE80 |                                                                   |
| \-     | Triangle wave.                                                    |
| 0xFEBF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFEC0 |                                                                   |
| \-     | Spiked wave.                                                      |
| 0xFEFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFF00 |                                                                   |
| \-     | Incremental sawtooth.                                             |
| 0xFF3F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFF40 |                                                                   |
| \-     | Decremental sawtooth.                                             |
| 0xFF7F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFF80 |                                                                   |
| \-     | Noise 1.                                                          |
| 0xFFBF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFFC0 |                                                                   |
| \-     | Noise 2.                                                          |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+
