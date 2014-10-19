
RRPGE system memory maps
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
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
peripheral. The addresses are given here with bit 15 set (not actually
belonging to the address) as this is how they have to be passed through the
FIFO's address word (bit 15 zero requests the FIFO to skip). The 16 Mixer DMA
registers repeat through the entire address space of the FIFO.

The register's descriptions may be found in the Mixer's documentation
("mix_arch.rst").

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x8000 | Amplitude source partition select                                 |
+--------+-------------------------------------------------------------------+
| 0x8001 | Amplitude source start, whole                                     |
+--------+-------------------------------------------------------------------+
| 0x8002 | Amplitude source start, fraction                                  |
+--------+-------------------------------------------------------------------+
| 0x8003 | Frequency for AM source read, whole                               |
+--------+-------------------------------------------------------------------+
| 0x8004 | Frequency for AM source read, fraction                            |
+--------+-------------------------------------------------------------------+
| 0x8005 | Partitioning settings                                             |
+--------+-------------------------------------------------------------------+
| 0x8006 | 64 KCell bank selection settings                                  |
+--------+-------------------------------------------------------------------+
| 0x8007 | Destination partition select                                      |
+--------+-------------------------------------------------------------------+
| 0x8008 | Destination start                                                 |
+--------+-------------------------------------------------------------------+
| 0x8009 | Amplitude multiplier                                              |
+--------+-------------------------------------------------------------------+
| 0x800A | Sample source partition select                                    |
+--------+-------------------------------------------------------------------+
| 0x800B | Sample source start, whole                                        |
+--------+-------------------------------------------------------------------+
| 0x800C | Sample source start, fraction                                     |
+--------+-------------------------------------------------------------------+
| 0x800D | Frequency, whole                                                  |
+--------+-------------------------------------------------------------------+
| 0x800E | Frequency, fraction                                               |
+--------+-------------------------------------------------------------------+
| 0x800F | Mode & start trigger                                              |
+--------+-------------------------------------------------------------------+




Graphics FIFO memory map
------------------------------------------------------------------------------


The Graphics FIFO is capable to access the registers of the Graphics
Accelerator peripheral and it's Reindex table. The addresses are given here
with bit 15 set (not actually belonging to the address) as this is how they
have to be passed through the FIFO's address word (bit 15 zero requests the
FIFO to skip).

The register's descriptions may be found in the Accelerator's documentation
("acc_arch.rst").

Only the 9 low bits of the address word are effective for addressing,
providing a repeating pattern every 0x0200 addresses. The first 0x200 (512)
words of these are described below.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x8000 |                                                                   |
| \-     | Accelerator registers. They repeat every 32 words in this range   |
| 0x80FF |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x8100 |                                                                   |
| \-     | Reindex table                                                     |
| 0x81FF |                                                                   |
+--------+-------------------------------------------------------------------+

The Accelerator registers:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x8000 | PRAM write mask high                                              |
+--------+-------------------------------------------------------------------+
| 0x8001 | PRAM write mask low                                               |
+--------+-------------------------------------------------------------------+
| 0x8002 | Unused                                                            |
+--------+-------------------------------------------------------------------+
| 0x8003 | Unused                                                            |
+--------+-------------------------------------------------------------------+
| 0x8004 | Source bank select                                                |
+--------+-------------------------------------------------------------------+
| 0x8005 | Destination bank select                                           |
+--------+-------------------------------------------------------------------+
| 0x8006 | Source partition select                                           |
+--------+-------------------------------------------------------------------+
| 0x8007 | Destination partition select                                      |
+--------+-------------------------------------------------------------------+
| 0x8008 | Partitioning settings                                             |
+--------+-------------------------------------------------------------------+
| 0x8009 | Substitution flags & Source barrel rotate                         |
+--------+-------------------------------------------------------------------+
| 0x800A | Source AND mask and Colorkey                                      |
+--------+-------------------------------------------------------------------+
| 0x800B | Reindex bank select                                               |
+--------+-------------------------------------------------------------------+
| 0x800C | Blit control flags                                                |
+--------+-------------------------------------------------------------------+
| 0x800D | Count of rows to process                                          |
+--------+-------------------------------------------------------------------+
| 0x800E | Count of 4 bit pixels to process per row                          |
+--------+-------------------------------------------------------------------+
| 0x800F | Pattern for Line & Filler mode & start trigger                    |
+--------+-------------------------------------------------------------------+
| 0x8010 | Source Y whole                                                    |
+--------+-------------------------------------------------------------------+
| 0x8011 | Source Y fraction                                                 |
+--------+-------------------------------------------------------------------+
| 0x8012 | Source Y increment whole                                          |
+--------+-------------------------------------------------------------------+
| 0x8013 | Source Y increment fraction                                       |
+--------+-------------------------------------------------------------------+
| 0x8014 | Source Y post-add whole                                           |
+--------+-------------------------------------------------------------------+
| 0x8015 | Source Y post-add fraction                                        |
+--------+-------------------------------------------------------------------+
| 0x8016 | Source X whole                                                    |
+--------+-------------------------------------------------------------------+
| 0x8017 | Source X fraction                                                 |
+--------+-------------------------------------------------------------------+
| 0x8018 | Source X increment whole                                          |
+--------+-------------------------------------------------------------------+
| 0x8019 | Source X increment fraction                                       |
+--------+-------------------------------------------------------------------+
| 0x801A | Source X post-add whole                                           |
+--------+-------------------------------------------------------------------+
| 0x801B | Source X post-add fraction                                        |
+--------+-------------------------------------------------------------------+
| 0x801C | Destination whole                                                 |
+--------+-------------------------------------------------------------------+
| 0x801D | Destination fraction                                              |
+--------+-------------------------------------------------------------------+
| 0x801E | Destination increment whole                                       |
+--------+-------------------------------------------------------------------+
| 0x801F | Destination post-add whole                                        |
+--------+-------------------------------------------------------------------+
