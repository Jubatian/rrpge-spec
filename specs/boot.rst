
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


- Most of the user register set (a, b, c, d, x0, x1, x2, x3, xh) is zeroed.
- The pointer mode register (xm) is set 0x6666 (16 bit pre-incrementing).
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

The display list is up as follows:

- Double scanned mode.
- 8 entries / line.
- Video RAM page 127; Offset 0x000 (as seen by the CPU).

The display list is populated the following way:

- Entry 0 gets the value zero (reset pattern).
- Entry 1 defines a display surface.
- Entries 2-7 are set zero so they are unused.

Source definitions are set up as follows:

- Source 0: 0x0450
- Source 1: 0x4A04
- Source 2: 0x8A04
- Source 3: 0xCA04
- Source 4: 0x1308
- Source 5: 0x9308
- Source 6: 0x1B08
- Source 7: 0x9B08

Source 0 sets up positioned source on Video RAM bank 0 of 80 cells width. This
is useful for non-scrolling display.

Sources 1-3 are set up 4 cell wide (16 pixels in 8 bit mode, 32 pixels in 4
bit mode) positioned source on Video RAM bank 1, in a manner they may be
continuously accessed through display list entries, useful for small sprites.

Sources 4-7 are set up 8 cell wide (32 pixels in 8 bit mode, 64 pixels in 4
bit mode) positioned source on Video RAM banks 2 and 3, in a manner they may
be continuously accessed through display list entries, useful for larger
sprites.

Entry 1 of the display list is populated as follows:

Line 0 gets the value 0x0000C000. Line 1 is 0x0005C000. Subsequent lines get
their entry values in a similar manner, adding 0x50000 to the previous line.
This layout produces a simple 320x200 surface in the beginning of the Video
RAM (which is banked in the CPU's address space initially).

Note that only the valid lines of the display list are populated (so 200
lines), the rest of the area of the display list remains zero.

The mask / colorkey definitions are set up as follows:

- Definition 0: 0x1020
- Definition 1: 0x4080


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
reindex map except the VRAM write masks, which are all set (both 0xFFFF).


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

    0x41C0U, 0x7FFFU, 0x807FU, 0x4001U, 0x4002U, 0x4003U, 0x4004U, 0x4005U,
    0x8000U, 0x8001U, 0x8002U, 0x8003U, 0x8004U, 0x8005U, 0x8006U, 0x8007U,
    0x7FFFU, 0x7FFFU, 0x807FU, 0x4001U, 0x4002U, 0x4003U, 0x4004U, 0x4005U,
    0x8000U, 0x8001U, 0x8002U, 0x8003U, 0x8004U, 0x8005U, 0x8006U, 0x8007U,

0xD20 - 0xD47: 0

0xD48: ::

    0x6666U

0xD49 - 0xD56: 0

0xD57: ::

    0x0001U

0xD58 - 0xD6F: 0

0xD70 - 0xD7F: ::

    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0xD000U, 0x01FCU, 0x1020U, 0x4080U,
    0x0450U, 0x4A04U, 0x8A04U, 0xCA04U, 0x1308U, 0x9308U, 0x1B08U, 0x9B08U,

0xD80 - 0xECF: 0

0xEC0 - 0xEDF: ::

    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0xFF80U, 0x0001U, 0x0000U, 0x0000U, 0x000CU, 0x000DU,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x6667U,
    0x0000U, 0x0000U, 0x0100U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,

0xEE0 - 0xEFF: ::

    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0xFFFFU, 0xFFFFU, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,
    0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U, 0x0000U,

0xF00 - 0xFFF: 0
