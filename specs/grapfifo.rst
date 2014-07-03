
Graphics FIFO DMA
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


The Graphics FIFO component provides access to the graphics hardware using a
FIFO data structure located in the CPU RAM. This enables filling in Graphics
Accelerator operations asynchronously to their execution, increasing the
performance of the system.

The CPU can not access the Accelerator registers by other means than the
Graphics FIFO. Addresses specified in the Accelerator's specification
(acc_arch.rst) thus mean addresses as used by the Graphics FIFO.

In RRPGE the size of the graphics FIFO is 8 pages, which can store up to 16K
operations (2 words per operation). This CPU RAM memory area is not directly
accessible to the user.




Graphics FIFO operation basics
------------------------------------------------------------------------------


When the CPU writes new operations to the Graphics FIFO, a write pointer to
the FIFO is incremented. When the Graphics FIFO executes an operation, a read
pointer to the FIFO is incremented.

The FIFO is empty if the two pointers (read and write) are equal.

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

After the Graphics FIFO is started, until the last Accelerator operation
finishes, the Video bus is fully occupied. This condition is signalled by the
FIFO busy flag. During this any access to the Video RAM (either through the
CPU <=> VRAM DMA or the 16 <=> 32 VRAM interface) may cause a kernel trap.




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
graphics accelerator operation is started, the FIFO is capable to perform one
operation every 4 cycles.


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
found in the User peripheral area. Note that these registers repeat every 16
words in the 0xE00 - 0xEFF range.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xE00  | Unused, always reads zero                                         |
+--------+-------------------------------------------------------------------+
|        | Graphics FIFO busy flag & start trigger                           |
| 0xE01  |                                                                   |
|        | - bit  1-15: 0 (always reads zero)                                |
|        | - bit     0: If set, the Graphics FIFO is busy.                   |
|        |                                                                   |
|        | When writing this register, the written value is ignored, while   |
|        | it starts the Graphics FIFO. If the FIFO is empty, it has no      |
|        | effect.                                                           |
+--------+-------------------------------------------------------------------+
| 0xE02  | Graphics FIFO command word latch. Always reads zero. Writing      |
|        | updates the command word to the value written.                    |
+--------+-------------------------------------------------------------------+
|        | Graphics FIFO data word & store trigger. Always reads zero.       |
| 0xE03  | Writing triggers a store into the FIFO area, and after the store, |
|        | increments bits 0-8 of the latched command word.                  |
+--------+-------------------------------------------------------------------+




Graphics FIFO command word format
------------------------------------------------------------------------------


The Graphics FIFO command word is simply a FIFO bus (Accelerator) address at
which the data should be written when processing the FIFO operation.

The graphics registers are described in the appropriate sections of the
Accelerator's specification (acc_arch.rst) and the Graphics Display
Generator's specification (vid_arch.rst).

Bits 9-15 of the Command word are unused. Bits 0-8 specify the graphics
register to fill in.

The Data word provides the data to be written into the specified graphics
register.

Note that graphics registers may be filled in sequentially by the CPU by only
writing the Graphics FIFO data word & store trigger since it auto-increments
the register address in the command word automatically.

If the register written is the Accelerator's start trigger (matched as binary
0xxx01111, 'x' indicating don't care bits), the Graphics FIFO also autostarts,
the same way like if the Start trigger (0xE01) register was written.
