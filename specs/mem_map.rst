
RRPGE system memory maps
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction
------------------------------------------------------------------------------


The RRPGE CPU is capable to access memory in four distinct address spaces:
Code, Stack, Data read and Data write. Of these the Code and Stack spaces are
fixed through the application's lifetime and have no special regions. The Data
read and write spaces support banking through kernel calls, so for them
providing a complete memory map for reference is beneficial.

The memory map first describes the reachable page regions, then, where
appropriate, the contents of a page are described in addition.

No distinction is made for Data read and Data write address spaces since their
source is a common pool of pages. Some pages however are not accessible for
writing, and Video RAM pages have special rules for writing.




Page map
------------------------------------------------------------------------------


The following table lists the accessible pages from the common page pool. Note
that since the kernel performs the page switches in a controlled way, it is
not possible to bank in inappropriate pages, so page contents and usage need
not be defined for those areas.

The stall field indicates which stall region the pages belong to. DMA
processes operating on a given region will stall the CPU if accessing the
same region. Mainly it indicates the area the Video hardware affects when it
blocks.

+--------+-----+-------+-----------------------------------------------------+
| Range  | R/W | Stall | Description                                         |
+========+=====+=======+=====================================================+
| 0x0000 |     |       |                                                     |
| \-     | \-  |   0   | Reserved                                            |
| 0x3FFF |     |       |                                                     |
+--------+-----+       +-----------------------------------------------------+
| 0x4000 |     |       | Data memory. Audio buffers are defined in page      |
| \-     | RW  |       | 0x4000.                                             |
| 0x40DF |     |       |                                                     |
+--------+-----+       +-----------------------------------------------------+
|        |     |       | Read Only Process Descriptor (ROPD). For the        |
| 0x40E0 |  R  |       | application this is read only; the kernel may write |
|        |     |       | into it to pass information to the application.     |
+--------+-----+       +-----------------------------------------------------+
| 0x40E1 |     |       |                                                     |
| \-     | \-  |       | Reserved                                            |
| 0x7FFE |     |       |                                                     |
+--------+-----+       +-----------------------------------------------------+
|        |     |       | Audio peripheral area. This region is used to       |
| 0x7FFF | RW  |       | operate the Mixer DMA, and the first Data memory    |
|        |     |       | page is also visible here for convenient access to  |
|        |     |       | the audio output buffers.                           |
+--------+-----+-------+-----------------------------------------------------+
| 0x8000 |     |       | Video memory. Can only be banked in for writing if  |
| \-     | RW* |   1   | the same page is also banked in for read at the     |
| 0x807F |     |       | same location in the CPU's address space.           |
+--------+-----+       +-----------------------------------------------------+
| 0x8080 |     |       |                                                     |
| \-     | \-  |       | Reserved                                            |
| 0xBFFE |     |       |                                                     |
+--------+-----+       +-----------------------------------------------------+
|        |     |       | Video peripheral area. This region is used to       |
| 0xBFFF | RW* |       | configure the display and to operate the graphics   |
|        |     |       | accelerators. The last Video memory page is also    |
|        |     |       | visible here. Can only banked in for writing if it  |
|        |     |       | is also banked in for read at the same location.    |
+--------+-----+       +-----------------------------------------------------+
| 0xC000 |     |       |                                                     |
| \-     | \-  |       | Reserved                                            |
| 0xFFFF |     |       |                                                     |
+--------+-----+-------+-----------------------------------------------------+

The areas in "R/W" marked with "*" need the same page banked in for read at
the same location in the CPU's address space.




Read Only Process Descriptor (ROPD)
------------------------------------------------------------------------------


The Read Only Process Descriptor is the area where the kernel makes a part of
the system's state visible to to the running application. It also serves as a
smaller data source as it contains the application binary's header (see
"bin_rpa.rst" for details) in which data may also be stored.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  | Application binary header page, as-is, except for head bytes:     |
| \-     | they read as "RPS\n" (instead of "RPA\n" as originally found in   |
| 0xBFF  | the application binary), for state. See "bin_rpa.rst".            |
+--------+-------------------------------------------------------------------+
| 0xC00  | Video palette in 4-4-4 RGB format, high 4 bits are zero. All      |
| \-     | entries are populated even in 4bit display mode. See "Palette" in |
| 0xCFF  | "vid_arch.rst" and "Set palette entry" in "kcall.rst".            |
+--------+-------------------------------------------------------------------+
| 0xD00  |                                                                   |
| \-     | Pages mapped in the CPU's Data read address space.                |
| 0xD0F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD10  |                                                                   |
| \-     | Pages mapped in the CPU's Data write address space.               |
| 0xD1F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD20  | Raster line to fire Video line interrupt at.                      |
+--------+-------------------------------------------------------------------+
| 0xD21  |                                                                   |
| \-     | Empty (reads as 0x0000)                                           |
| 0xD2F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD30  | Video line event handler offset. 0x0000: Handler off.             |
+--------+-------------------------------------------------------------------+
| 0xD31  | Audio half-empty event handler offset. 0x0000: Handler off.       |
+--------+-------------------------------------------------------------------+
| 0xD32  |                                                                   |
| \-     | Empty (reads as 0x0000)                                           |
| 0xD3E  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD3F  | Network availability: 0: not available, 1: available              |
+--------+-------------------------------------------------------------------+
| 0xD40  |                                                                   |
| \-     | Constant data. See "data.rst" for details.                        |
| 0xDFF  |                                                                   |
+--------+-------------------------------------------------------------------+




Video peripheral area
------------------------------------------------------------------------------


The video peripheral area (one page) contains the registers of the Graphics
display & Accelerator, and provides access to part of the last Video memory
page. This last page may be used to set up and access display lists along with
the peripheral registers.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Shadow of the last Video memory page (page 0x807F).               |
| 0xDFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xE00  | Graphics display & Accelerator peripheral registers. They repeat  |
| \-     | every 32 words in this range. See the memory maps in              |
| 0xEFF  | "vid_arch.rst" and "acc_arch.rst" for details.                    |
+--------+-------------------------------------------------------------------+
| 0xF00  |                                                                   |
| \-     | Reindex table. See memory map in "acc_arch.rst" for details.      |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+




Audio peripheral area
------------------------------------------------------------------------------


The audio peripheral area (one page) contains the registers of the Mixer
peripheral, and provides access to part of the first memory page where the
audio DMA buffers are located.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  | Shadow of the first memory page (page 0x4000) where the audio     |
| \-     | buffers are located. See "DMA buffers" in "snd_arch.rst" for      |
| 0xDFF  | details. This area is also populated by important initial data,   |
|        | see "data.rst" for details.                                       |
+--------+-------------------------------------------------------------------+
| 0xE00  | Mixer peripheral registers. They repeat every 16 words in this    |
| \-     | range. See "Mixer peripheral memory map" in "mix_arch.rst" for    |
| 0xFFF  | details.                                                          |
+--------+-------------------------------------------------------------------+
