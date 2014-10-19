
RRPGE Overview
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




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


    +---+        +-----------+
    |   |        | 16 bit    |
    |   |<======>| RRPGE CPU |
    |   |        |           |
    |   |        +-----------+
    |   |
    |   |        +-----------+
    |   |        | 16 bit    | User code (64 KWords)
    |   |<======>| CPU RAM   | User data (64 KWords)
    |   |        |           | User stack (32 KWords)
    |   |        +-----------+
    |   |
    |   |    +---+-----------+                     +---+
    | 1 |    | R | Graphics  |===> Video           |   |
    | 6 |<==>| e | Display   |                     |   |
    |   |    | g | Generator |<===================>|   |
    | b |    +---+-----------+                     | 3 |
    | i |                                          | 2 |
    | t |    +---+-----------+                     |   |
    |   |    | R |           |===> Sound           | b |
    | C |<==>| e | Audio     |                     | i |
    | P |    | g |           |<====================| t |
    | U |    +---+-----------+                     |   |
    |   |                                          | p |
    | b |    +---+-----------+                     | e |    +-----------+
    | u |    | R | PRAM      |                     | r |    | 1M x      |
    | s |<==>| e | Interface |<===================>| i |<==>| 32 bit    |
    |   |    | g |           |                     | p |    | PRAM      |
    |   |    +---+-----------+    +-----------+    | h |    +-----------+
    |   |                         |           |    | e |
    |   |    +---+-----------+    | Mixer DMA |<==>| r |
    |   |    | R | Mixer     |===>|           |    | a |
    |   |<==>| e | FIFO      |    +-----------+    | l |
    |   |    | g |           |<===================>|   |
    |   |    +---+-----------+    +-----------+    | b |
    |   |                         | Graphics  |    | u |
    |   |    +---+-----------+    | Accel.    |<==>| s |
    |   |    | R | Graphics  |===>|           |    |   |
    |   |<==>| e | FIFO      |    +-----------+    |   |
    |   |    | g |           |<===================>|   |
    +---+    +---+-----------+                     +---+


The 16 bit CPU bus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CPU is connected to this bus, so hardware on this bus is accessible by
normal memory accesses. From the user's point of view this hardware is the CPU
RAM, and the 64 words of peripheral registers giving access to the peripherals
whose CPU bus accessing side is marked as "Reg".

Only the CPU can initiate accesses to this bus, no other DMA peripherals are
present (at least not from the user's point of view).


The 32 bit Peripheral bus
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This bus is provided for the peripherals primarily for accessing the 1M x 32
bit Peripheral RAM (PRAM). They access it asynchronously to the CPU in a top
to down priority order by the block diagram (So Graphics Display Generator
having the highest priority while the Graphics FIFO having the lowest).

The Graphics Display Generator's accesses are fixed at every second main clock
cycle, any other peripheral can only use the other cycles. The Graphics
Display Generator is also capable to write on the bus, for the Display List
Clear feature (which it can exploit in VBlank).




The operating system (kernel)
------------------------------------------------------------------------------


The RRPGE CPU provides adequate supervisor and user mode separation. By this
the actual operating system code and data remains hidden, so it may be
implemented by native code in emulators.

The total size of the CPU RAM is not defined as no more from it is visible to
the user than noted on the block diagram. The minimal reasonable size to
accommodate these needs is 256 KWords. The extra area may be utilized by the
kernel to store internal data such as parts of it's code, some of the
application state, network buffers, file I/O buffers, and the likes.

There is no access by the kernel to the Peripheral RAM, it is entirely open
for the user. For most part the user accessible peripherals are also
independent of the kernel.
