
RRPGE application boot up state
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


When starting an application, the RRPGE system starts up in a definite state.
This state is described here most importantly detailing the reset state of all
registers and memories.

Unless otherwise mentioned any memory area must be reset to zero. This
includes the CPU Data, the CPU Stack, and the Peripheral memories. Note that
some parts of the CPU Data and Peripheral memories however contain initial
data.




RRPGE CPU reset state
------------------------------------------------------------------------------


- Most of the user register set (a, b, c, d, x0, x1, x2, x3, xb) is zeroed.
- The pointer mode register (xm) is set 0x6666 (16 bit post-incrementing).
- The stack is empty (sp zeroed, bp set up, the entire stack memory is zero).
- The program counter (pc) points at offset 0x0000.

The CPU's address space is initialized to a suitable initial configuration as
follows:

- Stack is set up according to the application's requirements.
- 64 KWords of data space is made available to the application.
- User Peripheral Area set up to data address range 0x0000 - 0x003F.
- Code space is filled with the application's code area.
- If there is less than 64 KWords of code, the Code space is padded with 0.




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




Reset state of memories
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


CPU RAM (Data memory)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 |                                                                   |
| \-     | User Peripheral Area (not RAM).                                   |
| 0x003F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0040 |                                                                   |
| \-     | Application Data from the Application Binary (see "bin_rpa.rst"). |
| D.End  |                                                                   |
+--------+-------------------------------------------------------------------+
| D.End  |                                                                   |
| \-     | Zero.                                                             |
| 0xFAB3 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAB4 |                                                                   |
| \-     | Initial data (see "data.rst").                                    |
| 0xFAFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFB00 |                                                                   |
| \-     | User Library (see "userlib/ulboot.rst").                          |
| 0xFDFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFE00 |                                                                   |
| \-     | Initial data (see "data.rst").                                    |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+


Code memory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 |                                                                   |
| \-     | Application Code from the Application Binary (see "bin_rpa.rst"). |
| C.End  |                                                                   |
+--------+-------------------------------------------------------------------+
| C.End  |                                                                   |
| \-     | Zero.                                                             |
| 0xDFFF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xE000 |                                                                   |
| \-     | User Library.                                                     |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+

The Application Code may override the User Library if it is sufficiently
large. This case the User Library becomes inaccessible.


PRAM (Peripheral memory)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------+------------------------------------------------------------------+
| Range   | Description                                                      |
+=========+==================================================================+
| 0x00000 |                                                                  |
| \-      | Zero.                                                            |
| 0xF3FFF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xF4000 |                                                                  |
| \-      | Display list 1 (up_dlist1), for double-buffering. Zero.          |
| 0xF77FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xF7800 |                                                                  |
| \-      | Right audio buffer (up_au_rt), 4096 samples. 0x80008000.         |
| 0xF7FFF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xF8000 |                                                                  |
| \-      | Display list 0 (up_dlist0), see Video reset state.               |
| 0xFB7FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFB800 |                                                                  |
| \-      | Left / Mono audio buffer (up_au_lf), 4096 samples. 0x80008000.   |
| 0xFBFFF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFC000 |                                                                  |
| \-      | Graphics FIFO, 8192 entries. Zero.                               |
| 0xFDFFF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFE000 |                                                                  |
| \-      | Mixer FIFO, 512 entries. Zero.                                   |
| 0xFE1FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFE200 |                                                                  |
| \-      | User Library (see "userlib/ulboot.rst").                         |
| 0xFF7FF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFF800 |                                                                  |
| \-      | Reserved for user 256 byte samples. 0x80808080.                  |
| 0xFFBFF |                                                                  |
+---------+------------------------------------------------------------------+
| 0xFFC00 |                                                                  |
| \-      | Initial data (see "data.rst").                                   |
| 0xFFFFF |                                                                  |
+---------+------------------------------------------------------------------+

Addresses are in cell (32 bit) units.




Video reset state
------------------------------------------------------------------------------


Display layout
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The display list is up as follows:

- 4 entries / line (smallest).
- Peripheral RAM offset 0xF8000.

The display list is populated the following way:

- Entry 0 gets the value zero (reset pattern).
- Entry 1 defines a display surface.
- Entries 2-7 are set zero so they are unused.

Source definitions are set up as follows:

- Source A0: 0x0028
- Source A1: 0x0000
- Source A2: 0x0000
- Source A3: 0x0000
- Source B0: 0x0000
- Source B1: 0x0000
- Source B2: 0x0000
- Source B3: 0x0000

The colorkey value registers are all set zero.

Source 0 sets up positioned source on Peripheral RAM bank 0 of 80 cells width,
with zero as low half-palette and X expansion turned off. This is useful for
non-scrolling display.

The remainig sources remain uninitialized (PRAM bank 0, 128 cells wide
positioned sources, zero as low half-palette, X expansion off).

Entry 1 of the display list is populated as follows:

Line 0 gets the value 0x00000400. Line 1 is 0x00500400. Subsequent lines get
their entry values in a similar manner, adding 0x500000 to the previous line.
This layout produces a simple 640x400 surface in the beginning of the
Peripheral RAM using the first 16 colors of the palette.

Note that only the valid lines of the display list are populated (so 400
lines).

Double scanning is disabled, however the background palette in the same
register is set up so the first 16 colors are used (0x1000).

The shift mode regions are both set up for 80 cells width, beginning at cell
0 (so filling entire display). Position mode clipping is disabled for all
sources.


Palette
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The palette is populated initially by the RRPGE Incremental palette. See the
"RRPGE Incremental palette" section in "data.rst" for details.


Accelerator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All registers of the Graphics Accelerator are set zero including the whole
reindex map except the PRAM write masks, which are all set (both 0xFFFF).


Graphics FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Internal pointers of the Graphics FIFO are set zero (so it is empty). The
FIFO's position is 0xFC000 in the Peripheral RAM, it's size is 8K cells.


Display state
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The application may be started with the display entering in Vertical blanking,
so it may have time to prepare some display. This behavior is not mandatory.




Audio reset state
------------------------------------------------------------------------------


Audio buffers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The audio output buffers are set up for mono output (left and right pointed at
the same location), at 0xFB800 in the Peripheral RAM, 2048 cells in size (4096
samples). It is filled with 0x8000, producing silence.

The Audio output DMA is prepared for 48KHz output.


Mixer peripheral
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All registers are initialized to zero.


Mixer FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Internal pointers of the Mixer FIFO are set zero (so it is empty). The FIFO's
position is 0xFE000 in the Peripheral RAM, it's size is 512 cells.




Peripheral RAM interface reset state
------------------------------------------------------------------------------


All four pointers are set to point at the beginning of the Peripheral RAM
(where the display surface is also set up). Data unit sizes are set up as
follows:

- Pointer 0: 1 bit.
- Pointer 1: 4 bits.
- Pointer 2: 8 bits.
- Pointer 3: 16 bits.

Increments are set up so they increment 1 data unit (corresponding with the
data unit size set up for the pointer).




Application state fill memory map
------------------------------------------------------------------------------


A suitable Application state fill is provided here which accords with the
initialization requirements. For more information on the layout of the
Application state, see "state.rst".

+--------+-------------------------------------------------------------------+
| Range  | Fill data                                                         |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Application header, the "RPA" heading changed to "RPS".           |
| 0x03F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x040  |                                                                   |
| \-     | 0                                                                 |
| 0x047  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x048  | 0x6666                                                            |
+--------+-------------------------------------------------------------------+
| 0x049  |                                                                   |
| \-     | 0                                                                 |
| 0x054  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x055  | 0xF800                                                            |
+--------+-------------------------------------------------------------------+
| 0x056  |                                                                   |
| \-     | 0                                                                 |
| 0x09F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0A0  | 0xFFFF                                                            |
+--------+-------------------------------------------------------------------+
| 0x0A1  | 0xFFFF                                                            |
+--------+-------------------------------------------------------------------+
| 0x0A2  |                                                                   |
| \-     | 0                                                                 |
| 0x0C3  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0C4  |                                                                   |
| \-     | 0xFB80, 0xFB80, 0xFF80, 0x0001                                    |
| 0x0C7  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0C8  | 0x1FE0                                                            |
+--------+-------------------------------------------------------------------+
| 0x0C9  |                                                                   |
| \-     | 0                                                                 |
| 0x0CB  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0CC  | 0x5FC0                                                            |
+--------+-------------------------------------------------------------------+
| 0x0CD  |                                                                   |
| \-     | 0                                                                 |
| 0x0CF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0D0  | 0x0000, 0x0000, 0x1000, 0x0000, 0x2800, 0x2800, 0xF800, 0x0000,   |
| \-     | 0x0028, 0x0000, 0x0000, 0x0000, 0x0000, 0x0000, 0x0000, 0x0000    |
| 0x0DF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0E0  |                                                                   |
| \-     | 0                                                                 |
| 0x0E2  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0E3  | 0x0001                                                            |
+--------+-------------------------------------------------------------------+
| 0x0E4  | 0x0000                                                            |
+--------+-------------------------------------------------------------------+
| 0x0E5  |                                                                   |
| \-     | 0                                                                 |
| 0x0EA  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0EB  | 0x0004                                                            |
+--------+-------------------------------------------------------------------+
| 0x0EC  | 0x0002                                                            |
+--------+-------------------------------------------------------------------+
| 0x0ED  |                                                                   |
| \-     | 0                                                                 |
| 0x0F2  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0F3  | 0x0008                                                            |
+--------+-------------------------------------------------------------------+
| 0x0F4  | 0x0003                                                            |
+--------+-------------------------------------------------------------------+
| 0x0F5  |                                                                   |
| \-     | 0                                                                 |
| 0x0FA  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0FB  | 0x0010                                                            |
+--------+-------------------------------------------------------------------+
| 0x0FC  | 0x0004                                                            |
+--------+-------------------------------------------------------------------+
| 0x0FD  |                                                                   |
| \-     | 0                                                                 |
| 0x0FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x100  |                                                                   |
| \-     | Palette, see "RRPGE Incremental palette" in "data.rst".           |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x200  |                                                                   |
| \-     | 0                                                                 |
| 0x3FF  |                                                                   |
+--------+-------------------------------------------------------------------+
