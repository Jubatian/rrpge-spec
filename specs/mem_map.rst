
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
| 0xE00  | User peripheral registers, Graphics. They repeat every 16 words   |
| \-     | in this range.                                                    |
| 0xEFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xF00  | User peripheral registers, Audio & DMA. They repeat every 32      |
| \-     | words in this range.                                              |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+

Summary of the user peripheral registers. For more detailed descriptions of
the registers, see the appropriate peripheral.

Graphics registers:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xE00  | Unused, always reads zero                                         |
+--------+-------------------------------------------------------------------+
| 0xE01  | Graphics FIFO busy flag & start trigger (see "gfifo.rst")         |
+--------+-------------------------------------------------------------------+
| 0xE02  | Graphics FIFO command word (see "gfifo.rst")                      |
+--------+-------------------------------------------------------------------+
| 0xE03  | Graphics FIFO data word & store trigger (see "gfifo.rst")         |
+--------+-------------------------------------------------------------------+
| 0xE04  | Shift mode region (see "vid_arch.rst")                            |
+--------+-------------------------------------------------------------------+
| 0xE05  | Display list definition (see "vid_arch.rst")                      |
+--------+-------------------------------------------------------------------+
| 0xE06  | Mask / Colorkey definition 0 (see "vid_arch.rst")                 |
+--------+-------------------------------------------------------------------+
| 0xE07  | Mask / Colorkey definition 1 (see "vid_arch.rst")                 |
+--------+-------------------------------------------------------------------+
| 0xE08  | Source definition 0 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE09  | Source definition 1 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0A  | Source definition 2 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0B  | Source definition 3 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0C  | Source definition 4 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0D  | Source definition 5 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0E  | Source definition 6 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xE0F  | Source definition 7 (see "vid_arch.rst")                          |
+--------+-------------------------------------------------------------------+

Audio & DMA registers:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xF00  | DMA source 256 word area or fill value (see "dma.rst")            |
+--------+-------------------------------------------------------------------+
| 0xF01  | CPU fill DMA target 256 word area & trigger (see "dma.rst")       |
+--------+-------------------------------------------------------------------+
| 0xF02  | CPU <=> CPU DMA target 256 word area & trigger (see "dma.rst")    |
+--------+-------------------------------------------------------------------+
| 0xF03  | CPU <=> VRAM DMA target 128 cell VRAM area, direction and trigger |
|        | (see "dma.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0xF04  |                                                                   |
| \-     | Unused, written values preserved                                  |
| 0xF07  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xF08  | Audio left channel DMA start offset bits (see "snd_arch.rst")     |
+--------+-------------------------------------------------------------------+
| 0xF09  | Audio right channel DMA start offset bits (see "snd_arch.rst")    |
+--------+-------------------------------------------------------------------+
| 0xF0A  | Audio DMA buffer size mask bits (see "snd_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0xF0B  | Audio clock divider (see "snd_arch.rst")                          |
+--------+-------------------------------------------------------------------+
| 0xF0C  | Audio DMA sample counter / next read offset (see "snd_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xF0D  | Audio DMA base clock (see "snd_arch.rst")                         |
+--------+-------------------------------------------------------------------+
| 0xF0E  | Mixer DMA frequency table whole pointer (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xF0F  | Mixer DMA frequency table fractional pointer (see "mix_arch.rst") |
+--------+-------------------------------------------------------------------+
| 0xF10  | Mixer DMA frequency source partition select (see "mix_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xF11  | Mixer DMA frequency source start, whole (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xF12  | Mixer DMA frequency source start, fraction (see "mix_arch.rst")   |
+--------+-------------------------------------------------------------------+
| 0xF13  | Mixer DMA amplitude source partition select (see "mix_arch.rst")  |
+--------+-------------------------------------------------------------------+
| 0xF14  | Mixer DMA amplitude source start, whole (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xF15  | Mixer DMA amplitude source start, fraction (see "mix_arch.rst")   |
+--------+-------------------------------------------------------------------+
| 0xF16  | Mixer DMA frequency indices for AM / FM (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xF17  | Mixer DMA partitioning settings (see "mix_arch.rst")              |
+--------+-------------------------------------------------------------------+
| 0xF18  | Mixer DMA destination start and partition select (see             |
|        | "mix_arch.rst")                                                   |
+--------+-------------------------------------------------------------------+
| 0xF19  | Mixer DMA 64KWord bank selection settings (see "mix_arch.rst")    |
+--------+-------------------------------------------------------------------+
| 0xF1A  | Mixer DMA amplitude multiplier (see "mix_arch.rst")               |
+--------+-------------------------------------------------------------------+
| 0xF1B  | Mixer DMA sample source partition select (see "mix_arch.rst")     |
+--------+-------------------------------------------------------------------+
| 0xF1C  | Mixer DMA sample source start, whole (see "mix_arch.rst")         |
+--------+-------------------------------------------------------------------+
| 0xF1D  | Mixer DMA sample source start, fraction (see "mix_arch.rst")      |
+--------+-------------------------------------------------------------------+
| 0xF1E  | Mixer DMA frequency select (see "mix_arch.rst")                   |
+--------+-------------------------------------------------------------------+
| 0xF1F  | Mixer DMA mode & start trigger (see "mix_arch.rst")               |
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
| 0x000  | Accelerator registers. They repeat every 32 words in this range.  |
| \-     | See the memory maps in "acc_arch.rst" for details.                |
| 0x0FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x100  |                                                                   |
| \-     | Reindex table. See memory map in "acc_arch.rst" for details.      |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+

Summary of the Accelerator registers. For more detailed descriptions of the
registers, see the memory maps in the Accelerator's documentation
("acc_arch.rst").

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Unused                                                            |
| 0x003  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x004  | VRAM write mask high                                              |
+--------+-------------------------------------------------------------------+
| 0x005  | VRAM write mask low                                               |
+--------+-------------------------------------------------------------------+
| 0x006  | Source bank & partition select                                    |
+--------+-------------------------------------------------------------------+
| 0x007  | Destination bank & partition select                               |
+--------+-------------------------------------------------------------------+
| 0x008  | Partitioning settings                                             |
+--------+-------------------------------------------------------------------+
| 0x009  | Reindex bank select                                               |
+--------+-------------------------------------------------------------------+
| 0x00A  | Substitution flags & source barrel rotate                         |
+--------+-------------------------------------------------------------------+
| 0x00B  | Source masks                                                      |
+--------+-------------------------------------------------------------------+
| 0x00C  | Colorkey & control flags                                          |
+--------+-------------------------------------------------------------------+
| 0x00D  | Count of rows to process                                          |
+--------+-------------------------------------------------------------------+
| 0x00E  | Count of 4 bit pixels to process per row                          |
+--------+-------------------------------------------------------------------+
| 0x00F  | Pattern for Line & Filler mode & start trigger                    |
+--------+-------------------------------------------------------------------+
| 0x010  | Source Y whole                                                    |
+--------+-------------------------------------------------------------------+
| 0x011  | Source Y fraction                                                 |
+--------+-------------------------------------------------------------------+
| 0x012  | Source Y increment whole                                          |
+--------+-------------------------------------------------------------------+
| 0x013  | Source Y increment fraction                                       |
+--------+-------------------------------------------------------------------+
| 0x014  | Source Y post-add whole                                           |
+--------+-------------------------------------------------------------------+
| 0x015  | Source Y post-add fraction                                        |
+--------+-------------------------------------------------------------------+
| 0x016  | Source X whole                                                    |
+--------+-------------------------------------------------------------------+
| 0x017  | Source X fraction                                                 |
+--------+-------------------------------------------------------------------+
| 0x018  | Source X increment whole                                          |
+--------+-------------------------------------------------------------------+
| 0x019  | Source X increment fraction                                       |
+--------+-------------------------------------------------------------------+
| 0x01A  | Source X post-add whole                                           |
+--------+-------------------------------------------------------------------+
| 0x01B  | Source X post-add fraction                                        |
+--------+-------------------------------------------------------------------+
| 0x01C  | Destination whole                                                 |
+--------+-------------------------------------------------------------------+
| 0x01D  | Destination fraction                                              |
+--------+-------------------------------------------------------------------+
| 0x01E  | Destination increment whole                                       |
+--------+-------------------------------------------------------------------+
| 0x01F  | Destination post-add whole                                        |
+--------+-------------------------------------------------------------------+
