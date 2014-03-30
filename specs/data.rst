
RRPGE constant data blocks
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction, the types and roles of constant data in RRPGE
------------------------------------------------------------------------------


The RRPGE system provides some important blocks of constant data for the
convenience of application developers. Most notably these cover the following
fields:

- Palette data providing useful basic color sets for both the 4 bit (16 color)
  and 8 bit (256 color) modes.

- Small and large sine tables for use with various algorithms requiring
  trigonometric functions (such as rotations and navigation in a three
  dimensional space).

- Basic waveforms and noise data for composing simple generated audio samples.

- Musical logarithmic table for outputting audio samples at tone frequencies.

In the following chapters these data blocks and their uses are described. On
the end of this document a memory map is provided showing their mapping at
boot. Note that not all of them are located in read only memory, so they may
be overwritten by the application.




RRPGE Incremental palette
------------------------------------------------------------------------------


The RRPGE Incremental palette is a generic any bit depth suitable palette. It
is constructed by two major design goals:

- Make it useful for most applications providing "good" colors.
- Organize the colors in the most useful manner.

The first goal is met for any bit depth while the second only for the first 64
(6 bit depth) colors.

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


5 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

From color index 16 to 19 the grayscale ramp is extended, also providing an
additional black which may be used for colorkeying. The extra colors are laid
out so they roughly align with the colors of the 4 bit palette where possible.


6 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When using 64 colors with the Graphics Accelerator's destination based
reindexing mode, it is impossible to distinguish by bit 5 (since this mode can
only use up to 5 bits from the destination). To make this reindexing mode
useful while retaining 64 colors, similar colors should be mapped on the upper
half of the palette to the lower half.

The palette is designed with this in mind having most of it's colors properly
aligned for such use. The following indices do not match or match very poorly
by this scheme and may be avoided when using the palette this way:

- 33 (cyan, poor match with 1: bright gray)
- 34 (cyan, does not match with 2: dark gray)
- 47 (purple, does not match with 15: brown)
- 48 (dark purple, poor match with 16: black)
- 49 (dark cyan, poor match with 17: mid gray)
- 50 (purple, does not match with 18: dark gray)
- 51 (bright green, does not match with 19: bright gray)
- 58 (dark brown, poor match with 26: mid brown)
- 63 (bright purple, does not match with 15: bright brown)


7 bit and 8 bit
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additional colors are allocated in an incremental manner to cover the color
space the best with each added color.


Data dump
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette dump below provides the 256 colors of RRPGE Incremental as 0RGB
4-4-4-4 bit entries, suitable for inclusion in C code: ::

    0x000U, 0xAAAU, 0x555U, 0xFFFU, 0x229U, 0x593U, 0xA42U, 0xEDBU,
    0x35EU, 0x85FU, 0x430U, 0xBC4U, 0x6AFU, 0x073U, 0x730U, 0xA83U,
    0x000U, 0x777U, 0x222U, 0xCCCU, 0x24AU, 0x6B1U, 0xD84U, 0x73DU,
    0x38EU, 0xEABU, 0x760U, 0x140U, 0x8CFU, 0x180U, 0xB68U, 0xBA4U,
    0x025U, 0x6AAU, 0x197U, 0xFF6U, 0x009U, 0x790U, 0xC53U, 0xED5U,
    0x057U, 0x62AU, 0x300U, 0x9A2U, 0x68EU, 0x450U, 0x602U, 0x708U,
    0x407U, 0x479U, 0xA18U, 0x8F6U, 0x079U, 0x6D4U, 0xEA5U, 0xA3BU,
    0x2BBU, 0xF6FU, 0x533U, 0x250U, 0x5ECU, 0x490U, 0xB4DU, 0xC6FU,
    0x65EU, 0xE4CU, 0xA60U, 0x0A1U, 0x20AU, 0xA6EU, 0xA20U, 0x51BU,
    0x020U, 0x9FFU, 0x900U, 0xA8AU, 0x748U, 0x1B5U, 0xDEFU, 0x356U,
    0x90AU, 0x772U, 0x5CFU, 0x279U, 0xBE4U, 0x102U, 0x8D3U, 0x427U,
    0x037U, 0x305U, 0x940U, 0x884U, 0x3D6U, 0x650U, 0x3BDU, 0x04AU,
    0x29BU, 0x3C1U, 0xD28U, 0x18CU, 0xDA2U, 0xC81U, 0x81CU, 0xDC2U,
    0xA3EU, 0x16CU, 0xA1CU, 0xAB0U, 0x2DDU, 0x627U, 0xDF5U, 0xB9FU,
    0x470U, 0x509U, 0xB70U, 0x574U, 0xC15U, 0x43EU, 0x045U, 0x733U,
    0x68AU, 0x955U, 0xFB4U, 0x7FEU, 0xE39U, 0x990U, 0xD63U, 0xD9FU,
    0x906U, 0x3F9U, 0x70CU, 0x2AEU, 0x92EU, 0x7F3U, 0x614U, 0xAC9U,
    0xD3FU, 0x6D1U, 0x7BBU, 0x0BCU, 0xF77U, 0xF63U, 0xBF7U, 0x5A7U,
    0xF4FU, 0xC1AU, 0x06AU, 0xCA7U, 0x30CU, 0x031U, 0xC40U, 0xFAFU,
    0x274U, 0x3A8U, 0x84AU, 0x1D7U, 0x02CU, 0x9A6U, 0x95BU, 0x382U,
    0xCC7U, 0xE1DU, 0x13EU, 0x27FU, 0xAD0U, 0xDE1U, 0x62FU, 0xF82U,
    0x440U, 0x088U, 0x972U, 0x31DU, 0x415U, 0xACEU, 0x0C0U, 0x7B5U,
    0xFB8U, 0x0AAU, 0x9F2U, 0x500U, 0x242U, 0xD01U, 0xDF9U, 0x00DU,
    0x827U, 0x8AFU, 0x204U, 0xB93U, 0x5A0U, 0x2FFU, 0x487U, 0xB1EU,
    0x5FFU, 0x05EU, 0xE50U, 0x51FU, 0x1F9U, 0xA0EU, 0x3E0U, 0xEB0U,
    0x5F1U, 0x247U, 0xFF1U, 0x330U, 0xC88U, 0x0ECU, 0x861U, 0x969U,
    0xA29U, 0xD90U, 0x0CEU, 0x751U, 0x73AU, 0xB02U, 0x253U, 0xB8DU,
    0x460U, 0xF11U, 0xF8BU, 0x0D3U, 0x70AU, 0xD0EU, 0xC76U, 0x896U,
    0x077U, 0x09FU, 0xF30U, 0xF60U, 0x80FU, 0x0FFU, 0x07FU, 0x10FU,
    0x934U, 0x638U, 0xEFCU, 0x57FU, 0x757U, 0x212U, 0x207U, 0x05BU,
    0x437U, 0xA4FU, 0x32FU, 0x423U, 0x068U, 0x46BU, 0x57BU, 0x9C7U,
    0x98FU, 0x028U, 0x96CU, 0x7F0U, 0x780U, 0xA0BU, 0x359U, 0x0AEU




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
A4 (440Hz) for a 256 byte sample at 48KHz can be produced using offset 0x8E in
this table.

The table contains 32 bit 16.16 fixed point sample pointer increments, split
in a whole and a fractional table (256 entries each) to suit the Audio mixer's
requirements.

There are 12 tones within an octave, the octave above or below may be obtained
by multiplying or dividing the table entries by two respectively. For the
greatest accuracy the top 12 entries are provided (offsets 0xF4 - 0xFF); lower
octaves then may be obtained by right shifting these values one by one for
each until generating the whole table.

The high 12 entries (offsets 0xF4 - 0xFF) of the table as 32bit values are as
follows: ::

    55678343U,  58989149U,  62496826U,  66213081U,  70150316U,  74321671U,
    78741067U,  83423255U,  88383859U,  93639437U,  99207528U, 105106715U

Lower octaves must be obtained by right shifting these values, discarding any
one bits falling off on the right.




Memory maps
------------------------------------------------------------------------------


The constant data blocks appear in two places within the user's address space:
one is the ROPD (Read Only Process Descriptor), area 0xD40 - 0xFFF, the other
is Data memory page 0 (page 0x4000), area 0x800 - 0xDFF. The latter may be
overwritten by the application. The latter area is selected since it appears
in the Audio peripheral page (page 0x7FFF), where it provides convenient
access to audio related data (the area 0x000 - 0x7FF may be occupied by the
audio output buffers).


ROPD data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xD40  |                                                                   |
|   -    | The first 64 colors of RRPGE Incremental.                         |
| 0xD7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD80  |                                                                   |
|   -    | Sine wave from Waveform data.                                     |
| 0xDFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xE00  |                                                                   |
|   -    | Large sine table.                                                 |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+


Data RAM page 0
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x800  |                                                                   |
|   -    | Square wave.                                                      |
| 0x87F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x880  |                                                                   |
|   -    | Sine wave.                                                        |
| 0x8FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x900  |                                                                   |
|   -    | Triangle wave.                                                    |
| 0x97F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x980  |                                                                   |
|   -    | Spiked wave.                                                      |
| 0x9FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xA00  |                                                                   |
|   -    | Incremental sawtooth.                                             |
| 0xA7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xA80  |                                                                   |
|   -    | Decremental sawtooth.                                             |
| 0xAFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xB00  |                                                                   |
|   -    | Noise 1.                                                          |
| 0xB7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xB80  |                                                                   |
|   -    | Noise 2.                                                          |
| 0xBFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xC00  |                                                                   |
|   -    | Musical logarithmic table, whole part.                            |
| 0xCFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD00  |                                                                   |
|   -    | Musical logarithmic table, fractional part.                       |
| 0xDFF  |                                                                   |
+--------+-------------------------------------------------------------------+
