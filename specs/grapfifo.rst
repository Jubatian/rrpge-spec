
Graphics FIFO DMA
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


The Graphics FIFO component provides access to the graphics hardware using a
FIFO data structure located in the CPU RAM. This enables filling in graphics
operations (typically operating the Graphics Accelerator) asynchronously to
their execution, increasing the performance of the system.

The CPU can not access the graphics hardware registers by other means than the
Graphics FIFO. Addresses specified in the Accelerator's specification
(acc_arch.rst) and the Graphics Display Generator (vid_arch.rst) thus mean
addresses as used by the Graphics FIFO.

In RRPGE the size of the graphics FIFO is 8 pages, which can store up to 16K
operations (2 words per operation). This CPU RAM memory area is not directly
accessible to the user.




Graphics FIFO operation basics
------------------------------------------------------------------------------


When the CPU writes new operations to the Graphics FIFO, a write pointer to
the FIFO is incremented. When the Graphics FIFO executes an operation, a read
pointer to the FIFO is incremented.

The FIFO is empty if the two pointers (read and write) are equal.

If the FIFO is non-empty, any access to Video other than by the FIFO produces
undefined results (may normally cause a kernel trap on a capable system). This
covers accesses by the CPU <=> VRAM DMA and the 16 <=> 32 VRAM interface. The
empty state of the FIFO can be queried.

The non-empty state may be broader (such as not only indicating FIFO not empty
but also being set as long as anything started by the FIFO is processing), but
the non-empty flag must always be consistent with the internal state (the
allowance of accessing graphics).

Overflowing the FIFO produces undefined results (may normally cause a kernel
trap on a capable system).

When writing an operation to the Graphics FIFO registers, normally only a
store DMA is executed to forward the operation to the FIFO memory area (2
words: command and data). To start processing, a start trigger needs to be
written. The FIFO however starts automatically when writing an operation to
fill the Accelerator's start trigger register.

When started, the FIFO operates autonomously, waiting for Accelerator
operations to finish if needed, until draining the FIFO. When the FIFO becomes
empty, the started state resets, so it may be started over later.




Timing and DMA accesses
------------------------------------------------------------------------------


Store trigger
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When hitting the store trigger (writing the Graphics FIFO's data register),
the Graphics FIFO stalls the CPU until finishing the store operation. This
adds 4 cycles to the execution time of the writing instruction (of which 2
cycles are CPU RAM writes outputting the command and the data words into the
FIFO area).


FIFO processing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The FIFO needs 2 DMA accesses to fetch an operation from the FIFO area. These
accesses are performed 2 cycles apart, leaving one unused cycle between, which
may be taken by other peripherals (typically the CPU) on the bus. Unless a
graphics accelerator operation is started or a beam condition is waited for,
the FIFO is capable to perform one operation every 4 cycles.


Emulation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The FIFO processing cycles are not necessary to be emulated on the CPU bus.
Their effect may be simulated by increasing the stall cycle count of store
trigger hits from 4 to 5 cycles. The processing cycles must however be present
on the Video bus.

Time taken by the FIFO once started may not be emulated in a cycle exact
manner. A line based emulation is sufficient.




Graphics FIFO registers in the user peripheral area
------------------------------------------------------------------------------


The following table lists the graphics FIFO registers accessible to the CPU,
found in the user peripheral area. Note that these registers repeat every 32
words in the 0xE00 - 0xFFF range.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xE04  | Unused, always reads zero                                         |
+--------+-------------------------------------------------------------------+
|        | Graphics FIFO non-empty flag & start trigger                      |
| 0xE05  |                                                                   |
|        | bit  1-15: 0 (always reads zero)                                  |
|        | bit     0: If set, the Graphics FIFO is non-empty, Video should   |
|        |            not be accessed.                                       |
|        |                                                                   |
|        | When writing this register, the written value is ignored, while   |
|        | it starts the Graphics FIFO. If the FIFO is empty, it has no      |
|        | effect.                                                           |
+--------+-------------------------------------------------------------------+
| 0xE06  | Graphics FIFO command word latch. Always reads zero. Writing      |
|        | updates the command word to the value written.                    |
+--------+-------------------------------------------------------------------+
|        | Graphics FIFO data word & store trigger. Always reads zero.       |
| 0xE07  | Writing triggers a store into the FIFO area, and after the store, |
|        | increments bits 0-8 of the latched command word. This increment   |
|        | only happens if the command word is a Graphics register write,    |
|        | and it does not address the Accelerator's start trigger.          |
+--------+-------------------------------------------------------------------+




Graphics FIFO command word format
------------------------------------------------------------------------------


There are two types of commands: Graphics register write and Beam wait
condition. If bit 15 of the command word is set, the command is a Graphics
register write, otherwise it is a Beam wait condition.

The graphics registers are described in the appropriate sections of the
Accelerator's specification (acc_arch.rst) and the Graphics Display
Generator's specification (vid_arch.rst).


Graphics register write
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bits 9-14 of the Command word are unused. Bits 0-8 specify the graphics
register to fill in.

The Data word provides the data to be written into the specified graphics
register.

Note that graphics registers may be filled in sequentially by the CPU by only
writing the Graphics FIFO data word & store trigger since it auto-increments
the register address in the command word automatically.

If the register written is the Accelerator's start trigger (matched as binary
0xxx01111, 'x' indicating don't care bits), the Graphics FIFO also autostarts,
the same way like if the Start trigger (0xE05) register was written.

The latched Command word's bit 0-8 auto-increments after a store if it was a
Graphics register write. The increment neither happens if it addresses the
Accelerator's start trigger (like above, matched as binary 0xxx01111, 'x'
indicating don't care bits).


Beam wait condition
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bits 10-14 of the Command word are unused. Bits 0-9 specify the beam position
to be Already reached, in 2's complement (-512 to 511).

Bits 10-15 of the Data word are unused. Bits 0-9 specify the beam position to
be Not yet reached, in 2's complement (-512 to 511).

When the Graphics FIFO encounters this operation while processing (reading)
the FIFO, if necessary, it waits for the specified beam condition to be met
before continuing processing. The conditions are formed as follows:

| If already_reached <= not_yet_reached then continue if:
| (beam_current >= already_reached) AND (beam_current < not_yet_reached)
| else continue if:
| (beam_current >= already reached) OR  (beam_current < not_yet_reached)

There are 400 display lines, indexed from 0 to 399 inclusive. The Vertical
blank lines have negative numbers, -1 being the line before the first display
line.

Note that there are at least 449 lines (400 displayed, 49 vertical blanking,
giving a range from -49 to 399 inclusive), depending on display hardware,
there may be more (such as 625 on PAL). It is possible to specify wait
conditions which are never met, essentially locking out the user application
from accessing the display any further.
