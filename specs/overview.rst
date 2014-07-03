
RRPGE Overview
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


The RRPGE system mimics a microcomputer design oriented for gaming roughly
covering the early 90's era, something which might have been possible if IBM
PC did not take over this domain to that time. Meanwhile it aims for decent
emulation performance even by relatively simple interpreting emulators.

For the second purpose the system is composed of a relatively weak CPU (for an
early 90's era machine) combined with a larger amount of additional hardware
to achieve specific goals in the graphics and audio domains.

In addition to the user accessible hardware components, an operating system is
also provided, whose interface may be accessed for specific functions such as
file I/O and input peripheral readings.




User accessible hardware
------------------------------------------------------------------------------


The user accessible hardware components connect together by the following
block diagram: ::


    +---+    +-----------+
    |   |    | 16 bit    |
    |   |<==>| RRPGE CPU |
    |   |    |           |
    |   |    +-----------+
    |   |
    |   |    +-----------+
    | 1 |    | 2M x      |
    | 6 |<==>| 16 bit    |
    |   |    | CPU RAM   |
    | b |    +-----------+
    | i |
    | t |    +-----------+
    |   |    | CPU RAM   |
    | C |<==>| Copy &    |
    | P |    | Fill DMA  |
    | U |    +-----------+             +-----------+
    |   |                              | Audio     |
    | b |<============================>| output    |============> Sound
    | u |                              | DMA       |
    | s |    +-----------+             +-----------+
    |   |    |           |
    |   |<==>| Mixer DMA |
    |   |    |           |
    |   |    +-----------+
    |   |                              +-----------+
    |   |<============================>| Graphics  |
    |   |                     +---+    | Display   |============> Video
    |   |    +-----------+    | 3 |===>| Generator |
    |   |    | CPU <=>   |    | 2 |    +-----------+
    |   |<==>| VRAM      |<==>|   |
    |   |    | DMA       |    | b |    +-----------+
    |   |    +-----------+    | i |    | 256K x    |
    |   |                     | t |<==>| 32 bit    |
    |   |    +-----------+    |   |    | VRAM      |
    |   |    | 16 <=> 32 |    | V |    +-----------+
    |   |<==>| VRAM      |<==>| i |
    |   |    | interface |    | d.|    +-----------+    +---+
    |   |    +-----------+    |   |    | Graphics  |    | F |
    |   |                     | b |<==>| Accel.    |<===| i |
    |   |                     | u |    |           |    | f |
    |   |                     | s |    +-----------+    | o |
    |   |    +-----------+    +---+                     |   |
    |   |    | Graphics  |                              | b |
    |   |<==>| FIFO DMA  |=============================>| u |
    |   |    |           |                              | s |
    +---+    +-----------+                              +---+


The 16 bit CPU bus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CPU is connected to this bus, so hardware on this bus is accessible by
normal memory accesses. The CPU RAM and the 16 <=> 32 VRAM interface are
accessible as memory pages (see "mem_map.rst" for details). The other DMA
hardware components are configurable and triggerable as memory mapped
registers on the user peripheral page.

The bus access priorities are as follows, the higher priority on the top:

- Audio output DMA
- Graphics FIFO DMA
- Other (synchronous) peripherals
- 16 bit RRPGE CPU

The "other (synchronous) peripherals are the CPU RAM Copy & Fill DMA, the
Mixer DMA, and the CPU <=> VRAM DMA, which are all started by a trigger from
the CPU (a write to a specific register on the user peripheral page).

When accessing the graphics hardware, stalls on the Video bus are propagated.
This applies to the CPU <=> VRAM DMA and the 16 <=> 32 VRAM interface.


The 32 bit Video bus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This bus is provided for working with the 256K x 32 bit Video RAM. Other
hardware connected to this bus only accesses the VRAM for reading or writing.
The priorities on the bus are as follows:

- Graphics Display Generator (fixed, every second cycle)
- Graphics Accelerator
- CPU <=> VRAM DMA
- 16 <=> 32 VRAM interface

The Graphics Display Generator accesses the VRAM for producing the video
output in real time. The bus is wired so it gets an access slot every second
cycle irrespective of whether it will use it, and the other hardware on the
bus get every other cycle.


The 16 bit FIFO bus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Graphics FIFO uses this bus to feed data into the Accelerator's registers.
This bus is unidirectional: the Graphics FIFO is the source, targeting the
appropriate registers of the Graphics Accelerator.

Note that the registers accessed by the Graphics FIFO are not directly
accessible to the RRPGE CPU.




Parallel hardware operation
------------------------------------------------------------------------------


The graphics system using the Graphics FIFO operates in parallel to the
thread ran by the RRPGE CPU. The RRPGE CPU can write the Graphics FIFO to
queue graphics operations which it will execute asynchronously. Since it
primarily feeds the Graphics Accelerator which occupies the Video bus for DMA,
the CPU bus remains free, so the processor can keep executing other tasks
while the queued graphics renders.

Note that the Graphics FIFO's FIFO is allocated within the CPU RAM, so the
operating Graphics FIFO still incurs some stalls on the CPU to read the
commands from it.




The operating system (kernel)
------------------------------------------------------------------------------


The RRPGE CPU provides adequate kernel and user mode separation. By this the
actual operating system code and data remains hidden, so it may be implemented
by native code in emulators.

The operating system uses an area of the CPU RAM to store it's internal data
along with data required by the user program, but not directly accessible to
it as memory. Large areas of the latter are the followings:

- User program code (16 pages)
- User program's stack (8 pages)
- Graphics FIFO (8 pages)

In addition to these, 32 pages are reserved for other non-specific kernel
internal data, so in total 448 pages (1792 Kwords) of data memory is available
for the user.

Note that the Mixer DMA and the Audio Output DMA can only access the first 1
Mwords. The CPU RAM Copy & Fill DMA and the CPU <=> VRAM DMA are restricted at
kernel level to only be able to work with the user accessible 448 pages of CPU
RAM.
