
Graphics & Mixer FIFO
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The Graphics and Mixer FIFO components operate in an identical manner, one
providing access to the Graphics Accelerator ("acc_arch.rst"), the other to
the Mixer DMA ("mix_arch.rst"). They use a FIFO structure in the Peripheral
RAM in which operations (register writes) may be queued which they execute in
sequence on the appropriate peripheral asynchronously to the RRPGE CPU.




FIFO operation basics
------------------------------------------------------------------------------


When the CPU writes new data into either FIFO through it's interface in the
User Peripheral Area, normally the FIFO forwards this data into the
appropriate FIFO buffer in the Peripheral RAM to be submitted to the
peripheral later, as soon as it is capable to receive it.

The FIFO buffer fills share the priority of the PRAM interface for accessing
the peripheral bus, ensuring the RRPGE CPU is normally not stalled for
accessing the filling registers. These fill registers have latches, so even if
the peripheral bus is inaccessible on the cycle the CPU attempts the access,
they don't stall. Note that it is not possible to read back PRAM data through
the FIFO interface, so no read stalls happen either.

The asynchronous reads of the FIFO have the priority just below the peripheral
they are commanding (Accelerator or Mixer), so they always wait for the
peripheral to finish before providing it with more data.

The FIFOs use internal 15 bit incrementing read and write pointers to perform
their task, of which 8 - 15 bits are used for generating the FIFO buffer
address depending on the selected FIFO size. Pointer equality signals FIFO
empty condition. There is no overflow check on the FIFOs (so the overflow
essentially empties the FIFO as the pointers become equal).

If the commanded peripheral is idle when a FIFO write access is performed by
the RRPGE CPU, the FIFO should bypass and write directly the target
peripheral, so the data does not appear in the buffer, and neither consumes
peripheral bus cycles. Otherwise the FIFO generates one write cycle when
filling, and one read cycle when reading the buffer.




Timing and DMA accesses
------------------------------------------------------------------------------


Store trigger
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When hitting the store trigger (writing the FIFO's data register), normally
the FIFO does not stall the RRPGE CPU. A stall may only happen if a previous
store didn't finish yet, which is unlikely (may only happen for a tight
sequence of stores interleaving with an Audio output access), and not needs to
be emulated. If the FIFO needs to access the peripheral bus (can not bypass),
any running Mixer or Accelerator task is stalled by 2 main clock cycles.


FIFO processing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Reading an entry of the FIFO buffer requires one peripheral bus access cycle
(not affecting the operation of the RRPGE CPU). In the case of the Mixer FIFO
this stalls the Accelerator (or it's FIFO) by 2 main clock cycles due to their
priority order.

Unless the commanded peripheral stalls it's FIFO, the FIFO is capable to
process one buffer entry each 2 main clocks (so is capable to use each
available peripheral bus cycle).


Emulation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the FIFO is non-empty, it can be assumed that until it can exhaust it's
buffer, it (along with the connected peripheral) is capable to fully utilize
the peripheral bus, so blocking any lower priority accesses.




FIFO registers in the user peripheral area
------------------------------------------------------------------------------


The Mixer FIFO resides in the 0x0008 - 0x000B area, and the Graphics FIFO in
the 0x000C - 0x000F area. Both FIFO's operate in an identical manner, so it is
sufficient to list the registers of one of them. The Graphics FIFO is used for
this purpose: the Mixer FIFO's registers are laid out identically.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | FIFO location & size.                                             |
| 0x000C |                                                                   |
|        | - bit    15: Unused, reads zero                                   |
|        | - bit 12-14: Size of FIFO                                         |
|        | - bit  0-11: Position in Peripheral RAM in 256 cell units         |
|        |                                                                   |
|        | Size of FIFO:                                                     |
|        |                                                                   |
|        | - 0: 256 cells, all position bits are used                        |
|        | - 1: 512 cells, high 11 bits of position are used                 |
|        | - 2: 1K cells, high 10 bits of position are used                  |
|        | - 3: 2K cells, high 9 bits of position are used                   |
|        | - 4: 4K cells, high 8 bits of position are used                   |
|        | - 5: 8K cells, high 7 bits of position are used                   |
|        | - 6: 16K cells, high 6 bits of position are used                  |
|        | - 7: 32K cells, high 5 bits of position are used                  |
+--------+-------------------------------------------------------------------+
|        | FIFO status flags.                                                |
| 0x000D |                                                                   |
|        | - bit  2-15: Unused, reads zero                                   |
|        | - bit     1: FIFO is suspended if set                             |
|        | - bit     0: FIFO is not empty or peripheral is working if set    |
|        |                                                                   |
|        | The FIFO is suspended flag can only be active in the case of the  |
|        | Graphics FIFO, for the Mixer FIFO it is always zero. When the     |
|        | FIFO is suspended, it won't perform buffer reads, and will never  |
|        | bypass on fills, so it won't access the peripheral. See "Double   |
|        | buffering assistance" in "vid_arch.rst" for further details.      |
+--------+-------------------------------------------------------------------+
|        | FIFO address word latch. Always reads zero.                       |
| 0x000E |                                                                   |
|        | - bit    15: If set, increments latch, ignoring bits 0-14         |
|        | - bit  0-14: Address to latch                                     |
+--------+-------------------------------------------------------------------+
|        | FIFO data word & store trigger. Always reads zero. Writing        |
| 0x000F | triggers a store into the FIFO buffer (or a bypass), and after    |
|        | the store / bypass, increments the latched address word.          |
+--------+-------------------------------------------------------------------+

In the FIFO buffer one 32 bit cell forms one entry, and it is laid out the
following way:

- bit    31: Always zero
- bit 16-30: Address
- bit  0-15: Data
