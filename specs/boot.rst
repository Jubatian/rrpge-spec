
RRPGE application bootup state
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


When starting an application, the RRPGE system starts up in a definite state.
This state is described here most importantly detailing the reset state of all
registers and memories.

Unless otherwise mentioned any memory area must be reset to zero. This
includes the Data, the Stack, the Graphics FIFO, and the Video memories. Note
that some parts of the Data and Video memories however contain initial data.




RRPGE CPU reset state
------------------------------------------------------------------------------


- The user register set (a, b, c, d, x0, x1, x2, x3, xm, xh) is zeroed.
- The stack is empty (sp and bp is zeroed, the entire stack memory is zero).
- The program counter (pc) points at offset 0x0000.

The CPU's address space is initialized to a suitable initial configuration as
follows:

- 8 stack pages are made available (0x8000 words of stack space).
- Code space is filled incrementally with the application's code pages.
- If there are less than 16 application code pages, the last page repeats.

Data Read and Data Write pages are set up according to the following table:

+----------+--------------------------------+--------------------------------+
| CPU page | Read page assigned             | Write page assigned            |
+==========+================================+================================+
|        0 | ROPD (0x40E0)                  | User peripheral area (0x7FFF)  |
+----------+--------------------------------+--------------------------------+
|        1 | User peripheral area (0x7FFF)  | User peripheral area (0x7FFF)  |
+----------+--------------------------------+--------------------------------+
|        2 | Video memory page 127 (0x807F) | Video memory page 127 (0x807F) |
+----------+--------------------------------+--------------------------------+
|        3 | Data memory page 1 (0x4001)    | Data memory page 1 (0x4001)    |
+----------+--------------------------------+--------------------------------+
|        4 | Data memory page 2 (0x4002)    | Data memory page 2 (0x4002)    |
+----------+--------------------------------+--------------------------------+
|        5 | Data memory page 3 (0x4003)    | Data memory page 3 (0x4003)    |
+----------+--------------------------------+--------------------------------+
|        6 | Data memory page 4 (0x4004)    | Data memory page 4 (0x4004)    |
+----------+--------------------------------+--------------------------------+
|        7 | Data memory page 5 (0x4005)    | Data memory page 5 (0x4005)    |
+----------+--------------------------------+--------------------------------+
|        8 | Video memory page 0 (0x8000)   | Video memory page 0 (0x8000)   |
+----------+--------------------------------+--------------------------------+
|        9 | Video memory page 1 (0x8001)   | Video memory page 1 (0x8001)   |
+----------+--------------------------------+--------------------------------+
|       10 | Video memory page 2 (0x8002)   | Video memory page 2 (0x8002)   |
+----------+--------------------------------+--------------------------------+
|       11 | Video memory page 3 (0x8003)   | Video memory page 3 (0x8003)   |
+----------+--------------------------------+--------------------------------+
|       12 | Video memory page 4 (0x8004)   | Video memory page 4 (0x8004)   |
+----------+--------------------------------+--------------------------------+
|       13 | Video memory page 5 (0x8005)   | Video memory page 5 (0x8005)   |
+----------+--------------------------------+--------------------------------+
|       14 | Video memory page 6 (0x8006)   | Video memory page 6 (0x8006)   |
+----------+--------------------------------+--------------------------------+
|       15 | Video memory page 7 (0x8007)   | Video memory page 7 (0x8007)   |
+----------+--------------------------------+--------------------------------+




Kernel reset state
------------------------------------------------------------------------------


- No kernel tasks are running.
- Network availability is unset (0).
- Network buffers are empty.
- Input buffers (text input FIFO) are empty.
- User specific information (names, language preferences, colors) set up.

Note that user specific information not necessarily have to be set up
explicitly as they may change during the application's lifetime. Their exact
behavior is implementation defined.




Video reset state
------------------------------------------------------------------------------


Display layout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Display lists are set up to be found on the following locations:

- Background list: Video RAM page 127; Offset 0x800.
- Layer 0 list: Video RAM page 127; Offset 0x400.
- Layer 1 list: Video RAM page 127; Offset 0x000.

The offsets are word (16 bit) offsets (as seen by the CPU).

The background display list is populated the following way:

- Each line gets the 32 bit value 0x40FF0000.

The meaning of the value is as follows:

- Global mask is 0xFF (all bit planes enabled in both 4 bit and 8 bit modes).
- Layer 0 is disabled.
- Layer 1 is in Colorkey mode.
- Background pattern is color index zero.

Each of the display lists are populated as follows:

- Line zero gets the 32 bit value 0x00000F00.
- Odd lines repeat the value of the previous line.
- Even lines (except line zero) increment the previous line by 0x00500000.

The first line points to the beginning of the layer's Video RAM bank. Further
lines set up double scanning, with a relative increment of 80 Video RAM cells
every second line. The colorkey is zero (which with the zero background
pattern makes this effect invisible).

Note that for all three display lists only the 400 visible lines are populated
by the above scheme. All the remaining areas of the Video RAM are left zero.

Layer 1 is pointed to Video RAM bank 0. Layer 0 is pointed to Video RAM bank
1.

This initial setup (coupled with that the first 8 Video RAM pages are banked
in the processor's address space) allows for composing simple graphics
directly after boot, without the need of altering the state of the Graphics
Display unit.


Palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette is populated initially by the RRPGE Incremental palette. See the
"RRPGE Incremental palette" section in "data.rst" for details. The full
palette is loaded even in 4 bit mode.


Video mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The starting mode is mode 1 (320x400; 8 bit).


Accelerator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All registers of the Graphics Accelerator are set zero including the whole
reindex map.


Graphics FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All registers of the Graphics FIFO are set to zero, so the Graphics FIFO is
empty and idle.


Display state
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The application may be started with the display entering in Vertical blanking
(most negative line), so the application may have time to prepare some
display. This behavior is not mandatory.




Audio reset state
------------------------------------------------------------------------------


Audio related data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Data memory page 0 is used primarily for audio. It has two major areas
initialized:

- 0x000 - 0x7FF: Filled with 0x8080, producing silence if played.
- 0x800 - 0xDFF: Populated according to the specification in "data.rst".
- 0xE00 - 0xFFF: 0

The Audio output DMA is prepared for mono 48KHz output, with the 0x000 - 0x7FF
area used for DMA buffer (for both channels).


Mixer peripheral
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most registers are initialized to zero except the followings:

- 0xEDA: 0x0100 (Amplitude)
- 0xECE: 0x000C (Frequency table whole pointer)
- 0xECF: 0x000D (Frequency table fractional pointer)
- 0xED7: 0x6667 (Partitioning: 256 samples for sources, 512 for destination)




ROPD dump memory map
------------------------------------------------------------------------------


A suitable Read Only Process Descriptor dump is provided here which fulfills
the initialization requirements. For more information on the layout of the
ROPD dump, see "ropddump.rst". Note that the 0x000 - 0xBFF area of the ROPD
dump replicates the application header.

0xC00 - 0xCFF: ::

    (See "RRPGE Incremental palette" in "data.rst")

0xD00 - 0xD1F: ::

    0x40E0U, 0x7FFFU, 0x807FU, 0x4001U, 0x4002U, 0x4003U, 0x4004U, 0x4005U,
    0x8000U, 0x8001U, 0x8002U, 0x8003U, 0x8004U, 0x8005U, 0x8006U, 0x8007U,
    0x7FFFU, 0x7FFFU, 0x807FU, 0x4001U, 0x4002U, 0x4003U, 0x4004U, 0x4005U,
    0x8000U, 0x8001U, 0x8002U, 0x8003U, 0x8004U, 0x8005U, 0x8006U, 0x8007U,

0xD20 - 0xD56: 0

0xD57: ::

    0x0001U

0xD58 - 0xECF: 0

0xEC0 - 0xEDF: ::

    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0xFF80U, 0x0001U, 0x0000U, 0x0000U, 0x000CU, 0x000DU,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x6667U,
    0x0000U, 0x0000U, 0x0100U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,

0xEE0 - 0xEFF: ::

    0xFFFFU, 0xFFFFU, 0x0000U, 0x01FEU, 0x01FDU, 0x01FCU, 0x0001U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,

0xF00 - 0xFFF: 0
