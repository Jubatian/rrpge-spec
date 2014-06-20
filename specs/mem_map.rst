
RRPGE system memory maps
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




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

+--------+-----+-------------------------------------------------------------+
| Range  | R/W | Description                                                 |
+========+=====+=============================================================+
| 0x0000 |     |                                                             |
| \-     | \-  | Reserved                                                    |
| 0x3FFF |     |                                                             |
+--------+-----+-------------------------------------------------------------+
| 0x4000 |     |                                                             |
| \-     | RW  | Data memory (CPU RAM).                                      |
| 0x41BF |     |                                                             |
+--------+-----+-------------------------------------------------------------+
|        |     | Read Only Process Descriptor (ROPD). For the application    |
| 0x41C0 |  R  | this is read only. The kernel may write into it to pass     |
|        |     | information to the application.                             |
+--------+-----+-------------------------------------------------------------+
| 0x41C1 |     |                                                             |
| \-     | \-  | Reserved                                                    |
| 0x7FFE |     |                                                             |
+--------+-----+-------------------------------------------------------------+
|        |     | User peripheral page. This region provides the user         |
| 0x7FFF | RW  | accessible memory mapped registers, and part of the first   |
|        |     | Data memory page is also visible here.                      |
+--------+-----+-------------------------------------------------------------+
| 0x8000 |     | Video memory (VRAM). Can only be banked in for writing if   |
| \-     | RW* | the same page is also banked in for read at the same        |
| 0x807F |     | location in the CPU's address space.                        |
+--------+-----+-------------------------------------------------------------+
| 0x8080 |     |                                                             |
| \-     | \-  | Reserved                                                    |
| 0xFFFF |     |                                                             |
+--------+-----+-------------------------------------------------------------+

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
| 0xD20  |                                                                   |
| \-     | Empty (reads as 0x0000)                                           |
| 0xD3E  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD3F  | Network availability: 0: not available, 1: available              |
+--------+-------------------------------------------------------------------+
| 0xD40  |                                                                   |
| \-     | Constant data. See "data.rst" for details.                        |
| 0xDFF  |                                                                   |
+--------+-------------------------------------------------------------------+




User peripheral page
------------------------------------------------------------------------------


The user peripheral page contains the memory mapped registers of the DMA
peripherals, the Graphics FIFO, and the audio output on it's higher addresses.
On the low part the first Data memory page is accessible.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  | Shadow of the first memory page (page 0x4000) where the initial   |
| \-     | audio buffers are located. This area is also populated with       |
| 0xDFF  | important initial data, see "data.rst" for details.               |
+--------+-------------------------------------------------------------------+
| 0xE00  | User peripheral registers. They repeat every 32 words in this     |
| \-     | range.                                                            |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+

Summary of the user peripheral registers. For more detailed descriptions of
the registers, see the appropriate peripheral.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xE00  | DMA source 256 word area or fill value (see "dma.rst")            |
+--------+-------------------------------------------------------------------+
| 0xE01  | CPU fill DMA target 256 word area & trigger (see "dma.rst")       |
+--------+-------------------------------------------------------------------+
| 0xE02  | CPU <=> CPU DMA target 256 word area & trigger (see "dma.rst")    |
+--------+-------------------------------------------------------------------+
| 0xE03  | CPU <=> VRAM DMA target 128 cell VRAM area, direction and trigger |
|        | (see "dma.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0xE04  | Unused, always reads zero                                         |
+--------+-------------------------------------------------------------------+
| 0xE05  | Graphics FIFO non-empty flag & start trigger (see "gfifo.rst")    |
+--------+-------------------------------------------------------------------+
| 0xE06  | Graphics FIFO command word (see "gfifo.rst")                      |
+--------+-------------------------------------------------------------------+
| 0xE07  | Graphics FIFO data word & store trigger (see "gfifo.rst")         |
+--------+-------------------------------------------------------------------+
| 0xE08  | Audio left channel DMA start offset bits (see "snd_arch.rst")     |
+--------+-------------------------------------------------------------------+
| 0xE09  | Audio right channel DMA start offset bits (see "snd_arch.rst")    |
+--------+-------------------------------------------------------------------+
| 0xE0A  | Audio DMA buffer size mask bits (see "snd_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0xE0B  | Audio clock divider (see "snd_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0C  | Audio DMA sample counter / next read offset (see "snd_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xE0D  | Audio DMA base clock (see "snd_arch.rst")                         |
+--------+-------------------------------------------------------------------+
| 0xE0E  | Mixer DMA frequency table whole pointer (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xE0F  | Mixer DMA frequency table fractional pointer (see "mix_arch.rst") |
+--------+-------------------------------------------------------------------+
| 0xE10  | Mixer DMA frequency source partition select (see "mix_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xE11  | Mixer DMA frequency source start, whole (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xE12  | Mixer DMA frequency source start, fraction (see "mix_arch.rst")   |
+--------+-------------------------------------------------------------------+
| 0xE13  | Mixer DMA amplitude source partition select (see "mix_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xE14  | Mixer DMA amplitude source start, whole (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xE15  | Mixer DMA amplitude source start, fraction (see "mix_arch.rst")   |
+--------+-------------------------------------------------------------------+
| 0xE16  | Mixer DMA frequency indices for AM / FM (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xE17  | Mixer DMA partitioning settings (see "mix_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0xE18  | Mixer DMA destination start and partition select (see             |
|        | "mix_arch.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0xE19  | Mixer DMA 64KWord bank selection settings (see "mix_arch.rst")    |
+--------+-------------------------------------------------------------------+
| 0xE1A  | Mixer DMA amplitude multiplier (see "mix_arch.rst")               |
+--------+-------------------------------------------------------------------+
| 0xE1B  | Mixer DMA sample source partition select (see "mix_arch.rst")     |
+--------+-------------------------------------------------------------------+
| 0xE1C  | Mixer DMA sample source start, whole (see "mix_arch.rst")         |
+--------+-------------------------------------------------------------------+
| 0xE1D  | Mixer DMA sample source start, fraction (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xE1E  | Mixer DMA frequency select (see "mix_arch.rst")                   |
+--------+-------------------------------------------------------------------+
| 0xE1F  | Mixer DMA mode & start trigger (see "mix_arch.rst")               |
+--------+-------------------------------------------------------------------+




Graphics FIFO memory map
------------------------------------------------------------------------------


The Graphics FIFO can write on a separate unidirectional bus (FIFO bus),
accessible only to it for writing, which bus connects to the graphics
hardware.

There are 9 address bits for this 16 bit bus, providing a range between 0x000
and 0x1FF. This range is assigned to the graphics hardware components as
follows:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  | Graphics display generator & Accelerator registers. They repeat   |
| \-     | every 32 words in this range. See the memory maps in              |
| 0x0FF  | "vid_arch.rst" and "acc_arch.rst" for details.                    |
+--------+-------------------------------------------------------------------+
| 0x100  |                                                                   |
| \-     | Reindex table. See memory map in "acc_arch.rst" for details.      |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+

Summary of the Graphics display generator & Accelerator registers. For more
detailed descriptions of the registers, see the appropriate peripheral.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  | VRAM write mask high (see "vid_arch.rst")                         |
+--------+-------------------------------------------------------------------+
| 0x001  | VRAM write mask low (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x002  | Unused                                                            |
+--------+-------------------------------------------------------------------+
| 0x003  | Background display list offset (see "vid_arch.rst")               |
+--------+-------------------------------------------------------------------+
| 0x004  | Layer 0 display list offset (see "vid_arch.rst")                  |
+--------+-------------------------------------------------------------------+
| 0x005  | Layer 1 display list offset (see "vid_arch.rst")                  |
+--------+-------------------------------------------------------------------+
| 0x006  | Layer 0 bank select (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x007  | Layer 1 bank select (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0x008  | Accelerator source bank & partition select (see "acc_arch.rst")   |
+--------+-------------------------------------------------------------------+
| 0x009  | Accelerator destination bank & partition select (see              |
|        | "acc_arch.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0x00A  | Accelerator reindex bank select & destination increment (see      |
|        | "acc_arch.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0x00B  | Accelerator source barrel rotate & partitioning settings (see     |
|        | "acc_arch.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0x00C  | Accelerator source masks (see "acc_arch.rst")                     |
+--------+-------------------------------------------------------------------+
| 0x00D  | Accelerator colorkey & control flags (see "acc_arch.rst")         |
+--------+-------------------------------------------------------------------+
| 0x00E  | Accelerator count of pixels to process (see "acc_arch.rst")       |
+--------+-------------------------------------------------------------------+
| 0x00F  | Accelerator pattern for fill mode & trigger (see "acc_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0x010  | Accelerator source Y whole (see "acc_arch.rst")                   |
+--------+-------------------------------------------------------------------+
| 0x011  | Accelerator source Y fraction (see "acc_arch.rst")                |
+--------+-------------------------------------------------------------------+
| 0x012  | Accelerator source Y increment whole (see "acc_arch.rst")         |
+--------+-------------------------------------------------------------------+
| 0x013  | Accelerator source Y increment fraction (see "acc_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0x014  | Accelerator source Y post-add whole (see "acc_arch.rst")          |
+--------+-------------------------------------------------------------------+
| 0x015  | Accelerator source Y post-add fraction (see "acc_arch.rst")       |
+--------+-------------------------------------------------------------------+
| 0x016  | Accelerator source X whole (see "acc_arch.rst")                   |
+--------+-------------------------------------------------------------------+
| 0x017  | Accelerator source X fraction (see "acc_arch.rst")                |
+--------+-------------------------------------------------------------------+
| 0x018  | Accelerator source X increment whole (see "acc_arch.rst")         |
+--------+-------------------------------------------------------------------+
| 0x019  | Accelerator source X increment fraction (see "acc_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0x01A  | Accelerator source X post-add whole (see "acc_arch.rst")          |
+--------+-------------------------------------------------------------------+
| 0x01B  | Accelerator source X post-add fraction (see "acc_arch.rst")       |
+--------+-------------------------------------------------------------------+
| 0x01C  | Accelerator destination whole (see "acc_arch.rst")                |
+--------+-------------------------------------------------------------------+
| 0x01D  | Accelerator destination fraction (see "acc_arch.rst")             |
+--------+-------------------------------------------------------------------+
| 0x01E  | Accelerator destination post-add whole (see "acc_arch.rst")       |
+--------+-------------------------------------------------------------------+
| 0x01F  | Accelerator destination post-add fraction (see "acc_arch.rst")    |
+--------+-------------------------------------------------------------------+
