
Peripheral RAM interface
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


The Peripheral RAM interface provides 4 independent pointers for the RRPGE
CPU by which it can access the Peripheral RAM in a sequential manner. This
interface has higher priority than both the Mixer DMA and the Graphics
Accelerator, so it is always accessible without major stalls.




Pointer operation basics
------------------------------------------------------------------------------


The Peripheral RAM interface pointers all operate independently of each other.

They provide a configurable data unit size from 1 bit to 16 bits in power of
2 increments, corresponding with the sub-word addressing scheme used by the
RRPGE CPU.

For realizing this in the write direction, two approaches may be taken.

- If the processor is implemented so it always performs a Read-Modify-Write
  sequence for data writes, the Read access may be used to latch the 32 bit
  PRAM cell for combining it when writing.

- If the processor may optimize out Read accesses, the combining logic has to
  be implemented over the write access if it lacked a corresponding Read,
  without stalling the access. This case the Write access is latched while the
  PRAM is read, then the read PRAM cell is combined with the latch, and
  written back.

To prevent post-incrementing the pointer on the Read access of a
Read-Modify-Write sequence, the signal line defined in "Memory accessing" in
the processor architecture documentation ("cpu_arch.rst") is used.

In read direction, the sub-word data is returned aligned to the lower bits of
the interface register, and in write direction the register accepts the data
to be written on the same lowermost bits. This corresponds with how the RRPGE
CPU realizes sub-word accessing.

The pointers operate according to a Big Endian data unit ordering scheme
(higher data units first), corresponding with the CPU.




Timing
------------------------------------------------------------------------------


Accessing the Peripheral bus is possible only in every second cycle, so read
accesses depending on their location may suffer a cycle of stall. Moreover the
higher priority audio output may also "steal" cycles occasionally.

However, for the point of emulation it is sufficient to add one stall cycle to
any instruction performing a peripheral bus access (irrespective of whether it
is a read or write), as described at "Memory access stalls" in "cpu_inst.rst".




Pointer register memory map
------------------------------------------------------------------------------


The pointers are accessible in the User peripheral area. There are four
pointers in the address ranges 0x0020 - 0x0027, 0x0028 - 0x002F, 0x0030 -
0x0037, and 0x0038 - 0x003F. Only the first (Pointer 0) is described from
these, the rest are formatted in an identical manner.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
|        | Pointer 0 Address, high.                                          |
| 0x0020 |                                                                   |
|        | - bit  9-15: Unused, reads zero                                   |
|        | - bit  0- 8: PRAM address bits 11 - 19                            |
+--------+-------------------------------------------------------------------+
|        | Pointer 0 Address, low.                                           |
| 0x0021 |                                                                   |
|        | - bit  5-15: PRAM address bits 0 - 10                             |
|        | - bit  0- 4: Sub-cell address bits                                |
|        |                                                                   |
|        | Bit 4 of the sub-cell address is always used. Other bits are used |
|        | depending on the selected data unit size, however they are always |
|        | writable, and the post-increment always affects them.             |
+--------+-------------------------------------------------------------------+
| 0x0022 | Pointer 0 Post-increment, high (layout matches Address high's).   |
+--------+-------------------------------------------------------------------+
| 0x0023 | Pointer 0 Post-increment, low (layout matches Address low's).     |
+--------+-------------------------------------------------------------------+
|        | Pointer 0 Data unit size.                                         |
| 0x0024 |                                                                   |
|        | - bit  3-15: Unused, reads zero                                   |
|        | - bit     3: If set, only post-increments after a R-M-W sequence  |
|        | - bit     2: If set, data unit size is 16 bits                    |
|        | - bit  0- 1: Data unit size                                       |
|        |                                                                   |
|        | Data unit size:                                                   |
|        |                                                                   |
|        | - 0: 1 bit. All sub-cell address bits are effective.              |
|        | - 1: 2 bits. Only bits 1 - 4 of sub-cell address are effective.   |
|        | - 2: 4 bits. Only bits 2 - 4 of sub-cell address are effective.   |
|        | - 3: 8 bits. Only bits 3 - 4 of sub-cell address are effective.   |
+--------+-------------------------------------------------------------------+
| 0x0025 | Unused, reads zero.                                               |
+--------+-------------------------------------------------------------------+
| 0x0026 | Pointer 0 Read / Write without post-increment.                    |
+--------+-------------------------------------------------------------------+
| 0x0027 | Pointer 0 Read / Write with post-increment.                       |
+--------+-------------------------------------------------------------------+

Setting bit 3 of the Data unit size register has a similar effect to the
processor's post-increment after write only mode: an increment is only
performed if the access includes a write. Note that the Read access of an
R-M-W sequence never generates a post-increment (realized by the R-M-W signal
from the processor as described in "Memory accessing" in "cpu_arch.rst").
