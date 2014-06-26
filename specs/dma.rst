
Memory copy & fill DMA peripherals
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


There are two memory DMA peripherals in the RRPGE system: the CPU RAM Copy &
Fill DMA and the CPU <=> VRAM DMA. Both DMA peripherals are capable to work
in the entire user accessible CPU RAM, allowing fixed size operations of 256
words (or 128 cells in the case of the Video RAM).

The peripherals can generate kernel traps (terminating the application) if
they are attempted to be operated on user inaccessible CPU RAM areas, or the
Video RAM is attempted to be accessed during a Graphics FIFO operation.

The CPU is stalled while either DMA is executing.




DMA variants and timing
------------------------------------------------------------------------------


CPU fill DMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Fills 256 words of CPU RAM with a given source word. Needs 272 cycles.


CPU <=> CPU DMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Copies 256 words from CPU RAM to CPU RAM. Needs 528 cycles.


CPU <=> VRAM DMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Copies 256 words between the Video RAM and the CPU RAM in either direction.
Needs 272 cycles (uses the CPU bus and the Video bus in parallel). It is
stalled by Accelerator operations if any is performing, and produces a kernel
trap if the Graphics FIFO is not empty.

(Note: An accelerator operation may only be allowed without falling under an
undefined behavior if it is the last operation from the Graphics FIFO, so the
Graphics FIFO is already empty. Implementations are allowed to report Graphics
FIFO non-empty this case inhibiting this situation)




DMA peripheral memory map
------------------------------------------------------------------------------


The following table lists the memory addresses within the User peripheral page
which relate the DMA peripherals. Note that these repeat every 32 words in the
0xE00 - 0xFFF range within this page.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xE00  | DMA source 256 word area or fill value.                           |
+--------+-------------------------------------------------------------------+
| 0xE01  | CPU fill DMA target 256 word area & trigger. Writing this         |
|        | register starts the CPU fill DMA.                                 |
+--------+-------------------------------------------------------------------+
| 0xE02  | CPU <=> CPU DMA target 256 word area & trigger. Writing this      |
|        | register starts the CPU <=> CPU DMA.                              |
+--------+-------------------------------------------------------------------+
|        | CPU <=> VRAM DMA target 128 cell VRAM area, direction & trigger.  |
| 0xE03  |                                                                   |
|        | bit    15: Direction: 0: CPU RAM => VRAM; 1: VRAM => CPU RAM      |
|        | bit 11-14: Unused                                                 |
|        | bit  0-10: 128 cell area within Video RAM                         |
|        |                                                                   |
|        | Writing this register starts the CPU <=> VRAM DMA.                |
+--------+-------------------------------------------------------------------+

The 256 word area selectors into the CPU RAM provide bits 8 - 23 of the
address which specify the high 4 bits within memory pages, and the low 12 bits
of the memory page. Note that only the user accessible range can be used
(pages 0x4000 - 0x41BF), so only register values 0x0000 - 0x1BFF are valid
(other values cause a kernel trap terminating the application).
