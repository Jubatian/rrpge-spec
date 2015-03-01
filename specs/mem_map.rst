
RRPGE system memory maps
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The RRPGE CPU is capable to access memory in three distinct address spaces:
Code, Stack, and Data. Of these the Code and Stack address spaces have no
special regions, while in the Data address space, which is normally used to
access 64 KWords of CPU RAM, has memory mapped peripherals accessible on it's
first 64 words.

Other areas of interest are the Mixer DMA and the Graphics Accelerator which
are accessible through their respective FIFOs only, and so have distinct
address spaces.




Data address space
------------------------------------------------------------------------------


This address space is reached by the Data addressing mode of the RRPGE CPU.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 |                                                                   |
| \-     | User peripheral area (Memory mapped registers).                   |
| 0x003F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0040 |                                                                   |
| \-     | Data memory (CPU RAM).                                            |
| 0xFFFF |                                                                   |
+--------+-------------------------------------------------------------------+




User peripheral area
------------------------------------------------------------------------------


The first 64 words of the Data address space.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 | Unused, writes ignored, reads 0x0000 ("NULL" pointer target)      |
+--------+-------------------------------------------------------------------+
| 0x0001 | 187.5Hz clock (48KHz / 256; incrementing counter), writes ignored |
+--------+-------------------------------------------------------------------+
| 0x0002 | Audio DMA sample counter / next read offset (see "snd_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0x0003 | Audio DMA base clock (see "snd_arch.rst")                         |
+--------+-------------------------------------------------------------------+
| 0x0004 | Audio left channel DMA start offset bits (see "snd_arch.rst")     |
+--------+-------------------------------------------------------------------+
| 0x0005 | Audio right channel DMA start offset bits (see "snd_arch.rst")    |
+--------+-------------------------------------------------------------------+
| 0x0006 | Audio DMA buffer size mask bits (see "snd_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0x0007 | Audio clock divider (see "snd_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x0008 | Mixer FIFO location & size (see "fifo.rst")                       |
+--------+-------------------------------------------------------------------+
| 0x0009 | Mixer FIFO status flags (see "fifo.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x000A | Mixer FIFO address word (see "fifo.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x000B | Mixer FIFO data word & store trigger (see "fifo.rst")             |
+--------+-------------------------------------------------------------------+
| 0x000C | Graphics FIFO location & size (see "fifo.rst")                    |
+--------+-------------------------------------------------------------------+
| 0x000D | Graphics FIFO status flags (see "fifo.rst")                       |
+--------+-------------------------------------------------------------------+
| 0x000E | Graphics FIFO address word (see "fifo.rst")                       |
+--------+-------------------------------------------------------------------+
| 0x000F | Graphics FIFO data word & store trigger (see "fifo.rst")          |
+--------+-------------------------------------------------------------------+
| 0x0010 | GDG Mask / Colorkey definition 0 (see "vid_arch.rst")             |
+--------+-------------------------------------------------------------------+
| 0x0011 | GDG Mask / Colorkey definition 1 (see "vid_arch.rst")             |
+--------+-------------------------------------------------------------------+
| 0x0012 | GDG Mask / Colorkey definition 2 (see "vid_arch.rst")             |
+--------+-------------------------------------------------------------------+
| 0x0013 | GDG Mask / Colorkey definition 3 (see "vid_arch.rst")             |
+--------+-------------------------------------------------------------------+
| 0x0014 | GDG Shift mode region A (see "vid_arch.rst")                      |
+--------+-------------------------------------------------------------------+
| 0x0015 | GDG Shift mode region B (see "vid_arch.rst")                      |
+--------+-------------------------------------------------------------------+
| 0x0016 | GDG Display list clear controls (see "vid_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0x0017 | GDG Display list definition & process flags (see "vid_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0x0018 | GDG Source definition A0 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x0019 | GDG Source definition A1 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001A | GDG Source definition A2 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001B | GDG Source definition A3 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001C | GDG Source definition B0 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001D | GDG Source definition B1 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001E | GDG Source definition B2 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x001F | GDG Source definition B3 (see "vid_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x0020 | Pointer 0 Address, high (see "pointer.rst")                       |
+--------+-------------------------------------------------------------------+
| 0x0021 | Pointer 0 Address, low (see "pointer.rst")                        |
+--------+-------------------------------------------------------------------+
| 0x0022 | Pointer 0 Post-increment, high (see "pointer.rst")                |
+--------+-------------------------------------------------------------------+
| 0x0023 | Pointer 0 Post-increment, low (see "pointer.rst")                 |
+--------+-------------------------------------------------------------------+
| 0x0024 | Pointer 0 Data unit size (see "pointer.rst")                      |
+--------+-------------------------------------------------------------------+
| 0x0025 | Unused, writes ignored, reads 0x0000                              |
+--------+-------------------------------------------------------------------+
| 0x0026 | Pointer 0 Read / Write without post-increment (see "pointer.rst") |
+--------+-------------------------------------------------------------------+
| 0x0027 | Pointer 0 Read / Write with post-increment (see "pointer.rst")    |
+--------+-------------------------------------------------------------------+
| 0x0028 |                                                                   |
| \-     | Pointer 1 (identical layout to Pointer 0, see "pointer.rst")      |
| 0x002F |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0030 |                                                                   |
| \-     | Pointer 2 (identical layout to Pointer 0, see "pointer.rst")      |
| 0x0037 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0038 |                                                                   |
| \-     | Pointer 3 (identical layout to Pointer 0, see "pointer.rst")      |
| 0x003F |                                                                   |
+--------+-------------------------------------------------------------------+




Mixer FIFO memory map
------------------------------------------------------------------------------


The Mixer FIFO is capable to access the 16 registers of the Mixer DMA
peripheral. The 16 Mixer DMA registers repeat through the entire address space
of the FIFO.

The register's descriptions may be found in the Mixer's documentation
("mix_arch.rst").

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 | Amplitude source partition select                                 |
+--------+-------------------------------------------------------------------+
| 0x0001 | Amplitude source start, whole                                     |
+--------+-------------------------------------------------------------------+
| 0x0002 | Amplitude source start, fraction                                  |
+--------+-------------------------------------------------------------------+
| 0x0003 | Frequency for AM source read, whole                               |
+--------+-------------------------------------------------------------------+
| 0x0004 | Frequency for AM source read, fraction                            |
+--------+-------------------------------------------------------------------+
| 0x0005 | Partitioning settings                                             |
+--------+-------------------------------------------------------------------+
| 0x0006 | 64 KCell bank selection settings                                  |
+--------+-------------------------------------------------------------------+
| 0x0007 | Destination partition select                                      |
+--------+-------------------------------------------------------------------+
| 0x0008 | Destination start                                                 |
+--------+-------------------------------------------------------------------+
| 0x0009 | Amplitude multiplier                                              |
+--------+-------------------------------------------------------------------+
| 0x000A | Sample source partition select                                    |
+--------+-------------------------------------------------------------------+
| 0x000B | Sample source start, whole                                        |
+--------+-------------------------------------------------------------------+
| 0x000C | Sample source start, fraction                                     |
+--------+-------------------------------------------------------------------+
| 0x000D | Frequency, whole                                                  |
+--------+-------------------------------------------------------------------+
| 0x000E | Frequency, fraction                                               |
+--------+-------------------------------------------------------------------+
| 0x000F | Mode & start trigger                                              |
+--------+-------------------------------------------------------------------+




Graphics FIFO memory map
------------------------------------------------------------------------------


The Graphics FIFO is capable to access the registers of the Graphics
Accelerator peripheral and it's Reindex table.

The register's descriptions may be found in the Accelerator's documentation
("acc_arch.rst").

Only the 9 low bits of the address word are effective for addressing,
providing a repeating pattern every 0x0200 addresses. The first 0x200 (512)
words of these are described below.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 |                                                                   |
| \-     | Accelerator registers. They repeat every 32 words in this range   |
| 0x00FF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0100 |                                                                   |
| \-     | Reindex table                                                     |
| 0x01FF |                                                                   |
+--------+-------------------------------------------------------------------+

The Accelerator registers:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x0000 | Peripheral RAM write mask, high                                   |
+--------+-------------------------------------------------------------------+
| 0x0001 | Peripheral RAM write mask, low                                    |
+--------+-------------------------------------------------------------------+
| 0x0002 | Destination bank select & Partition size                          |
+--------+-------------------------------------------------------------------+
| 0x0003 | Destination partition select                                      |
+--------+-------------------------------------------------------------------+
| 0x0004 | Destination post-add whole part                                   |
+--------+-------------------------------------------------------------------+
| 0x0005 | Destination post-add fractional part                              |
+--------+-------------------------------------------------------------------+
| 0x0006 | Count post-add whole part                                         |
+--------+-------------------------------------------------------------------+
| 0x0007 | Count post-add fractional part                                    |
+--------+-------------------------------------------------------------------+
| 0x0008 | Pointer Y post-add whole part                                     |
+--------+-------------------------------------------------------------------+
| 0x0009 | Pointer Y post-add fractional part                                |
+--------+-------------------------------------------------------------------+
| 0x000A | Pointer X post-add whole part                                     |
+--------+-------------------------------------------------------------------+
| 0x000B | Pointer X post-add fractional part                                |
+--------+-------------------------------------------------------------------+
| 0x000C | Pointer Y increment whole part                                    |
+--------+-------------------------------------------------------------------+
| 0x000D | Pointer Y increment fractional part                               |
+--------+-------------------------------------------------------------------+
| 0x000E | Pointer X increment whole part                                    |
+--------+-------------------------------------------------------------------+
| 0x000F | Pointer X increment fractional part                               |
+--------+-------------------------------------------------------------------+
| 0x0010 | Pointer Y whole part                                              |
+--------+-------------------------------------------------------------------+
| 0x0011 | Pointer Y fractional part                                         |
+--------+-------------------------------------------------------------------+
| 0x0012 | Source bank select                                                |
+--------+-------------------------------------------------------------------+
| 0x0013 | Source partition select                                           |
+--------+-------------------------------------------------------------------+
| 0x0014 | Source partitioning settings                                      |
+--------+-------------------------------------------------------------------+
| 0x0015 | Blit control flags & Source barrel rotate                         |
+--------+-------------------------------------------------------------------+
| 0x0016 | Source AND mask & Colorkey                                        |
+--------+-------------------------------------------------------------------+
| 0x0017 | Count of rows to blit                                             |
+--------+-------------------------------------------------------------------+
| 0x0018 | Count of cells / pixels to blit, whole part                       |
+--------+-------------------------------------------------------------------+
| 0x0019 | Count of cells / pixels to blit, fractional part                  |
+--------+-------------------------------------------------------------------+
| 0x001A | Source X whole part                                               |
+--------+-------------------------------------------------------------------+
| 0x001B | Source X fractional part                                          |
+--------+-------------------------------------------------------------------+
| 0x001C | Destination whole part                                            |
+--------+-------------------------------------------------------------------+
| 0x001D | Destination fractional part                                       |
+--------+-------------------------------------------------------------------+
| 0x001E | Reindexing & Pixel OR mask                                        |
+--------+-------------------------------------------------------------------+
| 0x001F | Start on write & Pattern                                          |
+--------+-------------------------------------------------------------------+
