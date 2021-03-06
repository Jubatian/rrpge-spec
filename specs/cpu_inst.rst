
RRPGE CPU instruction set
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------

The RRPGE CPU accesses instruction data in word (16 bit) units.

Most instructions are one word long, however the addressing mode may add an
another word for forming a 16 bit immediate parameter. Function calls also
have trailing parameter lists as extra opcode words.

Most instructions take an addressing mode specified parameter. This is formed
identically for all opcodes on the low 6 bits of the opcode.

Timing of instructions are determined by the opcode, the addressing mode, and
when accessing the memory, the activity of any stall. The CPU also may
implement a simple pipeline, however it's behavior is deterministic as far as
it is relied upon for the design.


Instruction execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An instruction may have multiple effects. These effects if their target
clashes are executed in a rigid order as follows; topmost in the list first,
so bottommost having the largest priority in affecting the target:

- Address generation for Read and Write memory accesses.
- Source read.
- Destination read.
- Post-increment if addressing mode requires it.
- Destination write (both in the case of XCH).
- Carry (C) write.

Note the order of Post-increment and Destination write. This order is in
effect when the destination is the same register which post-increments. This
order has no ill effect on destination writes targeting memory as the address
generation is performed before post-incrementing.


Parameter notations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- rx:    One of the registers: A, B, C, D, X0, X1, X2 or X3.
- rp:    One of the pointers: X0, X1, X2 or X3.
- xmn:   One of the 4bit parts of the XM register.
- xbn:   One of the 4bit parts of the XB register.
- imm4:  4bit immediate ranging from 0 - 15.
- imm16: 16bit immediate.
- adr:   Parameter specified by the addressing mode field.


Timing notations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- ai: 1 additional cycle if 2nd opcode word is used.
- ma: 1 additional cycle if a memory access is performed.
- wc: 1 additional cycle if carry write is necessary.


Operand order
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In two-operand instructions such as MOV or ADD, the destination operand comes
first, that is for example "ADD A, B" is equivalent to "A = A + B".


Memory access stalls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When reading through the PRAM interface, since only every second cycle of it
is available to the CPU, a cycle of stall may happen. Writing however is
performed through a latch if necessary, so no stall happens for these accesses
in normal circumstances. Extra stalls may be incurred by the Audio system if
an audio output DMA read clashes with the access, however these accesses are
infrequent.

For the purposes of acceptable emulation, implementing the PRAM read stall is
sufficient. The simplest implementation may unconditionally add a stall cycle
to any instruction accessing the PRAM (irrespective of whether it accesses the
PRAM for Read only or R-M-W).




Addressing modes
------------------------------------------------------------------------------

The addressing mode if the opcode has such an operand, is specified on the low
6 bits of the opcode. The nine supported addressing modes are as follows:

+---------------------+--------------+---------------------------------------+
| Binary              | Mnemonic     | Notes                                 |
+=====================+==============+=======================================+
| 00 iiii             | imm4         | 4bit immediate.                       |
+---------------------+--------------+---------------------------------------+
| 01 iiii             | [BP + imm4]  | Stack space.                          |
+---------------------+--------------+---------------------------------------+
| 10 00ii / 11ii...ii | imm16        | 16bit immediate (2 opcode words).     |
+---------------------+--------------+---------------------------------------+
| 10 01ii / 11ii...ii | BP + imm16   | BP rel. immediate (2 opcode words).   |
+---------------------+--------------+---------------------------------------+
| 10 10ii / 11ii...ii | [imm16]      | Data space (2 opcode words).          |
+---------------------+--------------+---------------------------------------+
| 10 11ii / 11ii...ii | [BP + imm16] | Stack space (2 opcode words).         |
+---------------------+--------------+---------------------------------------+
| 11 0rrr             | rx           | Registers A, B, C, D, X0, X1, X2, X3. |
+---------------------+--------------+---------------------------------------+
| 11 10rr             | [rp]         | Data space, pointer.                  |
+---------------------+--------------+---------------------------------------+
| 11 11rr             | [BP + rp]    | Stack space, pointer.                 |
+---------------------+--------------+---------------------------------------+

For more information, see "cpu_arch.rst", the "Addressing modes" section.

Immediate values spanning two opcodes have the highest 2 bits (bit 15 and 14)
in the first opcode.

Note that it is not strictly required that the second opcode word's high two
bits are set (bit 15 and bit 14 being one), however otherwise this opcode word
will have effect when executed if a jump or skip targets it (following the
recommendation it is a NOP, so has no effect).




Alphabetical instruction set
------------------------------------------------------------------------------


ADD
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0000 101r rraa aaaa | ADD rx, adr        |
+---------------------+--------------------+
| 0000 100r rraa aaaa | ADD adr, rx        |
+---------------------+--------------------+
| 0100 101r rraa aaaa | ADD C:rx, adr      |
+---------------------+--------------------+
| 0100 100r rraa aaaa | ADD C:adr, rx      |
+---------------------+--------------------+

Adds the source operand to the destination, and stores the result in the
destination: ::

    ADD dst, src
    dst = dst + src;

If Carry was specified, it will store 1 if the add generated a carry, 0
otherwise.

Timing (cycles): 3 + ai + ma + wc


ADC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0001 101r rraa aaaa | ADC rx, adr        |
+---------------------+--------------------+
| 0001 100r rraa aaaa | ADC adr, rx        |
+---------------------+--------------------+
| 0101 101r rraa aaaa | ADC C:rx, adr      |
+---------------------+--------------------+
| 0101 100r rraa aaaa | ADC C:adr, rx      |
+---------------------+--------------------+

Adds the source operand to the destination with carry, and stores the result
in the destination: ::

    ADC dst, src
    dst = dst + src + (C & 1);

If Carry was specified, it will store 1 if the add generated a carry, 0
otherwise.

Timing (cycles): 3 + ai + ma + wc


AND
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 001r rraa aaaa | AND rx, adr        |
+---------------------+--------------------+
| 1011 000r rraa aaaa | AND adr, rx        |
+---------------------+--------------------+

Performs binary AND between the source and destination operands, and stores
the result in the destination: ::

    AND dst, src
    dst = dst & src;

Timing (cycles): 3 + ai + ma


ASR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0001 001r rraa aaaa | ASR rx, adr        |
+---------------------+--------------------+
| 0001 000r rraa aaaa | ASR adr, rx        |
+---------------------+--------------------+
| 0101 001r rraa aaaa | ASR C:rx, adr      |
+---------------------+--------------------+
| 0101 000r rraa aaaa | ASR C:adr, rx      |
+---------------------+--------------------+

Performs arithmetic right shift (replicating the high bit) on the destination
operand using the lower 4 bits of the source as shift count: ::

    ASR dst, src
    dst = dst ASR (src & 0xF);

If Carry was specified, it is set zero, then receives the shifted out bits
from it's high end. ::

                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      dst (before the shift) |1|0|0|1|1|1|0|0|1|1|1|1|0|0|0|1| ASR 12
                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    dst (after the shift)             Carry (after the shift)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |1|1|1|1|1|1|1|1|1|1|1|1|1|0|0|1| |1|1|0|0|1|1|1|1|0|0|0|1|0|0|0|0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Timing (cycles): 3 + ai + ma + wc


BTC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1010 00ii iiaa aaaa | BTC adr, imm4      |
+---------------------+--------------------+

Clears the bit given in the immediate of the operand specified by adr. ::

    BTC dst, imm4
    dst = dst & ~(1 << imm4);

Timing (cycles): 3 + ai + ma


BTS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1010 10ii iiaa aaaa | BTS adr, imm4      |
+---------------------+--------------------+

Sets the bit given in the immediate of the operand specified by adr. ::

    BTS dst, imm4
    dst = dst | (1 << imm4);

Timing (cycles): 3 + ai + ma


DIV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0001 011r rraa aaaa | DIV rx, adr        |
+---------------------+--------------------+
| 0001 010r rraa aaaa | DIV adr, rx        |
+---------------------+--------------------+
| 0101 011r rraa aaaa | DIV C:rx, adr      |
+---------------------+--------------------+
| 0101 010r rraa aaaa | DIV C:adr, rx      |
+---------------------+--------------------+

Divides the (unsigned) destination operand with the (unsigned) source operand,
and produces the whole part of the result in the destination: ::

    DIV dst, src
    dst = dst / src;

If Carry was specified, it receives the remainder.

If the divisor is zero, both the destination and the remainder is zeroed; this
condition does not trigger any supervisor action (trap).

Timing (cycles): 20 + ai + ma + wc


JFR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 0100 0eaa aaaa | JFR adr {...}      |
+---------------------+--------------------+

Relative function call (subroutine entry). The target address is calculated by
adding the operand to the current PC which points to the JFR instruction.

After the function call opcode (including the additional opcode word if it was
necessary by the addressing mode) up to 16 parameters may follow which are
pushed on the called function's stack. The parameter opcode format is normally
formed as follows: ::

    --00 00-- -eaa aaaa
    (Normally first two bits should be 11 for making these NOPs)

The 'e' bit (also found in the function's opcode) marks the last parameter if
if it is set. The 16th parameter ignores the 'e' bit treating it set.

If stack space addressing is used in the parameter opcodes, the parameter is
taken from the caller's stack (and pushed onto the new stack frame created for
the subroutine).

The function call is executed through the following essential steps:

- Current SP is saved for remembering the target offest for the saved PC.
- SP is incremented (skipping the word which will receive PC).
- BP is pushed on the stack.
- Current BP + SP is saved for loading in BP later (for new stack frame).
- Parameters (if any) are processed and pushed on the stack, counting those.
- PC (pointing after the last parameter) is saved to the remembered offset.
- BP is loaded to establish new stack frame.
- SP is set to the parameter count.

Note that the actual implementation may establish the new stack frame before
processing the parameter list, however then it has to remember the old stack
frame to be used when stack addressing modes are used in parameter loads.

An example call with 3 parameters: ::

    0: JFR func {10, [20], [X0]}

    0: 0100 0100 0010 00ff -- JFR opcode with imm16 address
    1: 11ff ffff ffff ffff -- Address (relative) of function
    2: 1100 0000 0000 1010 -- Parameter 10 decimal as imm4 addressing mode
    3: 1100 0000 0010 1000 -- Parameter [20], first byte
    4: 1100 0000 0001 0100 -- Second byte
    5: 1100 0000 0111 1000 -- Parameter [X0], final parameter ('e' bit set)

    | (...)       |
    +-------------+
    | PC (at 6)   | Saved return address, pointing after the last parameter
    +-------------+
    | BP (caller) |
    +-------------+--> End of caller's stack frame
    | 10          | <- BP; first parameter's value
    +-------------+
    | pppp        | Second parameter, value read from [20].
    +-------------+
    | pppp        | Third parameter, value read from [X0].
    +-------------+
    |             | <- BP + SP
    +-------------+
    | (...)       |

The function parameters may encode extended immediates, thus allowing for a
larger range of immediates to be encoded within a single instruction word. The
supported parameter encodings are as follows:

+---------------------+------------------------------------------------------+
| Binary              | Effect                                               |
+=====================+======================================================+
| --00 00-- -eaa aaaa | Normal address parameter                             |
+---------------------+------------------------------------------------------+
| --00 01-j jeii iiii | jjii iiii jjii iiii (Examples: 0x5A5A; 0x4444)       |
+---------------------+------------------------------------------------------+
| --00 1jjj jeii iiii | 1111 11jj jjii iiii (Examples: 0xFC12; 0xFFFD)       |
+---------------------+------------------------------------------------------+
| --01 0jjj jeii iiii | jjjj iiii ii00 0000 (Examples: 0x8000; 0x96C0)       |
+---------------------+------------------------------------------------------+
| --01 1jjj jeii iiii | jjjj iiii ii11 1111 (Examples: 0x803F; 0x96FF)       |
+---------------------+------------------------------------------------------+
| --1j jjjj jeii iiii | 0000 jjjj jjii iiii (Examples: 0x0125; 0x0FED)       |
+---------------------+------------------------------------------------------+

Timing (cycles): 5 + ai + ma; 2 + ai + ma / parameter


JFA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 0101 0eaa aaaa | JFA adr {...}      |
+---------------------+--------------------+

Absolute function call (subroutine entry). The target address is the operand.

See JFL for details.

Timing (cycles): 5 + ai + ma; 2 + ai + ma / parameter


JNZ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 10ir rrii iiii | JNZ rx, simm7      |
+---------------------+--------------------+

Jump if rx is nonzero. The base of the jump is the address of the opcode, so
an immediate of zero will generate an infinite loop. The 7 bit immediate is
2's complement signed ranging from -64 to +63 inclusive. The 6th bit
(determining the sign) of it is bit 6 of the opcode.

This instruction is basically a replacement for the commonly used "XEG rx, 0;
JMS addr" sequence, mostly occurring in loop constructs.

Timing (cycles): 2 (no jump) / 4 (jump)


JMR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 0100 00aa aaaa | JMR adr            |
+---------------------+--------------------+
| 1000 0100 01aa aaaa | JMR B, adr         |
+---------------------+--------------------+
| 1000 0100 10aa aaaa | JMR C, adr         |
+---------------------+--------------------+
| 1000 0100 11aa aaaa | JMR D, adr         |
+---------------------+--------------------+

Relative jump. The target address is calculated by adding the operand to the
current PC which points to the JMR instruction.

If a register (B, C or D) is specified, it receives the value of PC pointing
after the jump opcode: this may be used to implement small subroutines.

Timing (cycles): 5 + ai + ma


JMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 0101 00aa aaaa | JMA adr            |
+---------------------+--------------------+
| 1000 0101 01aa aaaa | JMA B, adr         |
+---------------------+--------------------+
| 1000 0101 10aa aaaa | JMA C, adr         |
+---------------------+--------------------+
| 1000 0101 11aa aaaa | JMA D, adr         |
+---------------------+--------------------+

Absolute jump. The target address is the operand.

If a register (B, C or D) is specified, it receives the value of PC pointing
after the jump opcode: this may be used to implement small subroutines.

Timing (cycles): 4 + ai + ma


JMS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 11ii iiii iiii | JMS simm10         |
+---------------------+--------------------+

Short relative jump. The base of the jump is the address of the opcode, so an
immediate of zero will generate an infinite loop. The 10bit immediate is 2's
complement signed ranging from -512 to +511 inclusive.

Timing (cycles): 4


JSV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 0100 1eii iiii | JSV imm6 {...}     |
+---------------------+--------------------+

Supervisor call. The imm6 operand selects the supervisor service to call.

Works in a similar manner to JFL including the parameter list, however
parameters are pushed on the supervisor stack, and the PC and BP of the caller
is not pushed onto any stack (they are preserved by the user - supervisor mode
switch).

Note that this call is accompanied by some kind of return from supervisor
mechanism (a supervisor - user mode switch might do using the context saved or
swapped out on entry), however it's exact mechanism is left implementation
defined.

The supervisor stack is protected from overflowing by the 16 parameter high
limit. Kernel code may prepare the supervisor stack so 16 parameters are
guaranteed to fit in case the user mode performs a supervisor call.

Timing of this opcode including it's parameter passings is implementation
defined (included in the kernel operation time limits, see the kernel's
documentation "kernel.rst" for details).


MAC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 011r rraa aaaa | MAC rx, adr        |
+---------------------+--------------------+
| 0011 010r rraa aaaa | MAC adr, rx        |
+---------------------+--------------------+
| 0111 011r rraa aaaa | MAC C:rx, adr      |
+---------------------+--------------------+
| 0111 010r rraa aaaa | MAC C:adr, rx      |
+---------------------+--------------------+

Multiply and accumulate. Multiplies the destination with the source operand,
then adds carry, and stores the result in the destination: ::

    MAC dst, src
    dst = dst * src + C;

If Carry was specified, it receives the high 16 bits of the result.

Timing (cycles): 13 + ai + ma + wc


MOV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0000 001r rraa aaaa | MOV rx, adr        |
+---------------------+--------------------+
| 0000 000r rraa aaaa | MOV adr, rx        |
+---------------------+--------------------+
| 0000 011r rrxx xxxx | MOV rx, imx        |
+---------------------+--------------------+
| 0100 0010 nnaa aaaa | MOV xmn, adr       |
+---------------------+--------------------+
| 0100 0000 nnaa aaaa | MOV adr, xmn       |
+---------------------+--------------------+
| 0100 0011 nnaa aaaa | MOV xbn, adr       |
+---------------------+--------------------+
| 0100 0001 nnaa aaaa | MOV adr, xbn       |
+---------------------+--------------------+
| 0100 011r rrxx xxxx | MOV rx, imx        |
+---------------------+--------------------+
| 1000 0010 00aa aaaa | MOV XM, adr        |
+---------------------+--------------------+
| 1000 0000 00aa aaaa | MOV adr, XM        |
+---------------------+--------------------+
| 1000 0010 01aa aaaa | MOV XB, adr        |
+---------------------+--------------------+
| 1000 0000 01aa aaaa | MOV adr, XB        |
+---------------------+--------------------+
| 1000 0010 10aa aaaa | MOV SP, adr        |
+---------------------+--------------------+
| 1000 0000 10aa aaaa | MOV adr, SP        |
+---------------------+--------------------+
| 1000 0011 1iii i000 | MOV SP, imx        |
+---------------------+--------------------+
| 1000 0001 1iii i000 | MOV imx, SP (NOP)  |
+---------------------+--------------------+
| 1100 011r rrxx xxxx | MOV rx, imx        |
+---------------------+--------------------+

Moves from source to target.

When the source is a 4bit part of the XM (xmn) or XB (xbn) register, the
destination will receive the value in it's low 4 bits, and it's high 12 bits
are set zero.

When the destination is a 4bit part of the XM (xmn) or XB (xbn) register, the
destination (the appropriate part of XM or XB) will receive the low 4 bits of
the source.

The "MOV SP, imx" variant can be used to load SP with values from 16 to 31
(the provided immediate is used for bits 0 - 3, bit 4 is set).

The "MOV rx, imx" variants are used to load special immediate values into a
register in one instruction word. These instructions lay out as follows:

+---------------------+------------------------------------------------------+
| Binary              | Effect                                               |
+=====================+======================================================+
| 0000 011r rr00 iiii | MOV rx, table0[i] (Loads from a table of immediates) |
+---------------------+------------------------------------------------------+
| 0000 011r rr01 iiii | MOV rx, 0x001i (Loads 16 - 31)                       |
+---------------------+------------------------------------------------------+
| 0000 011r rr10 iiii | MOV rx, 0x002i (Loads 32 - 47)                       |
+---------------------+------------------------------------------------------+
| 0000 011r rr11 iiii | MOV rx, 0x003i (Loads 48 - 63)                       |
+---------------------+------------------------------------------------------+
| 0100 011r rrii iiii | MOV rx, 0000 0000 01ii iiii (Loads 64 - 127)         |
+---------------------+------------------------------------------------------+
| 1100 011r rrii iiii | MOV rx, table1[i] (Loads from a table of immediates) |
+---------------------+------------------------------------------------------+

The values in table0 (16 words) are as follows: ::

    0x0280U, 0xFF0FU, 0xF0FFU, 0x0180U, 0x0300U, 0x01C0U, 0x0F00U, 0x0118U,
    0x0140U, 0x0168U, 0x0190U, 0x01B8U, 0x01E0U, 0x0208U, 0x0230U, 0x0258U

The values in table1 (64 words) are as follows: ::

    0x0080U, 0x0088U, 0x0090U, 0x0098U, 0x0010U, 0x0020U, 0x0040U, 0x0080U,
    0x0100U, 0x0200U, 0x0400U, 0x0800U, 0x1000U, 0x2000U, 0x4000U, 0x8000U,
    0x00A0U, 0x00A8U, 0x00B0U, 0x00B8U, 0xFFEFU, 0xFFDFU, 0xFFBFU, 0xFF7FU,
    0xFEFFU, 0xFDFFU, 0xFBFFU, 0xF7FFU, 0xEFFFU, 0xDFFFU, 0xBFFFU, 0x7FFFU,
    0x00C0U, 0x00C8U, 0x00D0U, 0x00D8U, 0xFFE0U, 0xFFC0U, 0xFF80U, 0xFF00U,
    0xFE00U, 0xFC00U, 0xF800U, 0xF000U, 0xE000U, 0xC000U, 0x8000U, 0x0000U,
    0x00E0U, 0x00E8U, 0x00F0U, 0x00F8U, 0x001FU, 0x003FU, 0x007FU, 0x00FFU,
    0x01FFU, 0x03FFU, 0x07FFU, 0x0FFFU, 0x1FFFU, 0x3FFFU, 0x7FFFU, 0xFFFFU

Timing (cycles): 2 + ai + ma


MUL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 011r rraa aaaa | MUL rx, adr        |
+---------------------+--------------------+
| 0010 010r rraa aaaa | MUL adr, rx        |
+---------------------+--------------------+
| 0110 011r rraa aaaa | MUL C:rx, adr      |
+---------------------+--------------------+
| 0110 010r rraa aaaa | MUL C:adr, rx      |
+---------------------+--------------------+

Multiplies the destination with the source operand, and stores the result in
the destination: ::

    MUL dst, src
    dst = dst * src;

If Carry was specified, it receives the high 16 bits of the result.

Timing (cycles): 12 + ai + ma + wc


NEG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0110 001r rraa aaaa | NEG rx, adr        |
+---------------------+--------------------+
| 0110 000r rraa aaaa | NEG adr, rx        |
+---------------------+--------------------+

2's complement negates the source operand, and stores the result in the
destination: ::

    NEG dst, src
    dst = 0 - src;

Timing (cycles): 3 + ai + ma


NOP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 11-- ---- ---- ---- | NOP                |
+---------------------+--------------------+

No operation.

Timing (cycles): 1


NOT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 001r rraa aaaa | NOT rx, adr        |
+---------------------+--------------------+
| 0010 000r rraa aaaa | NOT adr, rx        |
+---------------------+--------------------+

Performs a binary NOT on the source operand, and stores the result in the
destination: ::

    NOT dst, src
    dst = src ^ 0xFFFF;

Timing (cycles): 2 + ai + ma


OR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 001r rraa aaaa | OR rx, adr         |
+---------------------+--------------------+
| 0011 000r rraa aaaa | OR adr, rx         |
+---------------------+--------------------+

Performs binary OR between the source and destination operands, and stores the
result in the destination: ::

    OR dst, src
    dst = dst | src;

Timing (cycles): 3 + ai + ma


POP
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 0011 rrrr rrrr | POP rx [,...]      |
+---------------------+--------------------+

Pops off registers from the stack. For each register, the Stack Pointer (SP)
decrements first, then the register is loaded. The eight 'r' bits specify
which registers to load (if set), which are as follows in bit7 to bit0
order: ::

    XM, XB, A, B, D, X0, X1, X2

The order of registers on the stack from lower address to higher is the same:
for example if all 'r' bits are set, first 'X2' will be popped off of the
stack, and last, 'XM'.

There are some invalid combinations which are used to encode different
instructions:

- 1000 0011 01xx xxxx: Popping XB without XM is not available.
- 1000 0011 1xxx x000: Popping XM without a pointer register is not available.

Timing (cycles): 2 + 1 / register


PSH
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 0001 rrrr rrrr | PSH rx [,...]      |
+---------------------+--------------------+

Pushes registers on the stack. For each register, the register is saved, then
the Stack Pointer (SP) increments. The eight 'r' bits specify which registers
to save (if set), which are as follows in bit7 to bit0 order: ::

    XM, XB, A, B, D, X0, X1, X2

The order of registers on the stack from lower address to higher is the same:
for example if all 'r' bits are set, first 'XM' will be pushed on the stack,
and last, 'X2'.

There are some invalid combinations which are used to encode different
instructions:

- 1000 0001 01xx xxxx: Pushing XB without XM is not available.
- 1000 0001 1xxx x000: Pushing XM without a pointer register is not available.

Timing (cycles): 2 + 1 / register


RFN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 0101 10aa aaaa | RFN x3, adr        |
+---------------------+--------------------+
| 0100 0101 11aa aaaa | RFN c:x3, adr      |
+---------------------+--------------------+

Returns from function or subroutine. Loads a return value in x3, optionally
also clearing (to zero) carry. To omit returning a value, x3 may be used as
adr. Note: adr is evaulated before performing the return, so stack relative
sources use the appropriate stack frame.

For the associated mechanisms, check the JFR opcode and the "Stack Management"
section in "cpu_arch.rst".

Timing (cycles): 6 + ai + ma


SBC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0001 111r rraa aaaa | SBC rx, adr        |
+---------------------+--------------------+
| 0001 110r rraa aaaa | SBC adr, rx        |
+---------------------+--------------------+
| 0101 111r rraa aaaa | SBC C:rx, adr      |
+---------------------+--------------------+
| 0101 110r rraa aaaa | SBC C:adr, rx      |
+---------------------+--------------------+

Subtracts the source operand from the destination with borrow, and stores the
result in the destination: ::

    SBC dst, src
    dst = dst - src - (C & 1);

If Carry was specified, it will store 0xFFFF if the subtraction generated a
borrow, 0 otherwise.

Timing (cycles): 3 + ai + ma + wc


SHL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 111r rraa aaaa | SHL rx, adr        |
+---------------------+--------------------+
| 0010 110r rraa aaaa | SHL adr, rx        |
+---------------------+--------------------+
| 0110 111r rraa aaaa | SHL C:rx, adr      |
+---------------------+--------------------+
| 0110 110r rraa aaaa | SHL C:adr, rx      |
+---------------------+--------------------+

Left shifts the destination operand using the lower 4 bits of the source as
shift count: ::

    SHL dst, src
    dst = dst << (src & 0xF);

If Carry was specified, it is set zero, then receives the shifted out bits
from it's low end. ::

                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      dst (before the shift) |1|0|0|1|1|1|0|0|1|1|1|1|0|0|0|1| << 4
                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Carry (after the shift)           dst (after the shift)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|0|0|0|0|0|0|0|0|0|0|0|1|0|0|1| |1|1|0|0|1|1|1|1|0|0|0|1|0|0|0|0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Timing (cycles): 3 + ai + ma + wc


SHR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 101r rraa aaaa | SHR rx, adr        |
+---------------------+--------------------+
| 0010 100r rraa aaaa | SHR adr, rx        |
+---------------------+--------------------+
| 0110 101r rraa aaaa | SHR C:rx, adr      |
+---------------------+--------------------+
| 0110 100r rraa aaaa | SHR C:adr, rx      |
+---------------------+--------------------+

Right shifts the destination operand using the lower 4 bits of the source as
shift count: ::

    SHR dst, src
    dst = dst >> (src & 0xF);

If Carry was specified, it is set zero, then receives the shifted out bits
from it's high end. ::

                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      dst (before the shift) |1|0|0|1|1|1|0|0|1|1|1|1|0|0|0|1| >> 12
                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    dst (after the shift)             Carry (after the shift)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|0|0|0|0|0|0|0|0|0|0|0|1|0|0|1| |1|1|0|0|1|1|1|1|0|0|0|1|0|0|0|0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Timing (cycles): 3 + ai + ma + wc


SLC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 111r rraa aaaa | SLC rx, adr        |
+---------------------+--------------------+
| 0011 110r rraa aaaa | SLC adr, rx        |
+---------------------+--------------------+
| 0111 111r rraa aaaa | SLC C:rx, adr      |
+---------------------+--------------------+
| 0111 110r rraa aaaa | SLC C:adr, rx      |
+---------------------+--------------------+

Left shifts the destination operand using the lower 4 bits of the source as
shift count, then ORs the carry with the result: ::

    SLC dst, src
    dst = (dst << (src & 0xF)) | C;

If Carry was specified, it is set zero, then receives the shifted out bits
from it's low end. ::

                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      dst (before the shift) |1|0|0|1|1|1|0|0|1|1|1|1|0|0|0|1| << 4
                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

                                     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            Carry (before the shift) |0|0|0|0|0|0|0|0|0|0|0|0|1|0|1|1|
                                     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    Carry (after the shift)           dst (after the shift)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|0|0|0|0|0|0|0|0|0|0|0|1|0|0|1| |1|1|0|0|1|1|1|1|0|0|0|1|1|0|1|1|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Timing (cycles): 3 + ai + ma + wc


SRC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 101r rraa aaaa | SRC rx, adr        |
+---------------------+--------------------+
| 0011 100r rraa aaaa | SRC adr, rx        |
+---------------------+--------------------+
| 0111 101r rraa aaaa | SRC C:rx, adr      |
+---------------------+--------------------+
| 0111 100r rraa aaaa | SRC C:adr, rx      |
+---------------------+--------------------+

Right shifts the destination operand using the lower 4 bits of the source as
shift count, then ORs the carry with the result: ::

    SRC dst, src
    dst = (dst << (src & 0xF)) | C;

If Carry was specified, it is set zero, then receives the shifted out bits
from it's high end. ::

                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      dst (before the shift) |1|0|0|1|1|1|0|0|1|1|1|1|0|0|0|1| >> 12
                             +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
     |0|1|0|1|0|0|1|1|0|0|0|1|0|0|0|0| Carry (before the shift)
     +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

    dst (after the shift)             Carry (after the shift)
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |0|1|0|1|0|0|1|1|0|0|0|1|1|0|0|1| |1|1|0|0|1|1|1|1|0|0|0|1|0|0|0|0|
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Timing (cycles): 3 + ai + ma + wc


SUB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0000 111r rraa aaaa | SUB rx, adr        |
+---------------------+--------------------+
| 0000 110r rraa aaaa | SUB adr, rx        |
+---------------------+--------------------+
| 0100 111r rraa aaaa | SUB C:rx, adr      |
+---------------------+--------------------+
| 0100 110r rraa aaaa | SUB C:adr, rx      |
+---------------------+--------------------+

Subtracts the source operand from the destination, and stores the result in
the destination: ::

    SUB dst, src
    dst = dst - src;

If Carry was specified, it will store 0xFFFF if the subtraction generated a
borrow, 0 otherwise.

Timing (cycles): 3 + ai + ma + wc


XBC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1010 01ii iiaa aaaa | XBC adr, imm4      |
+---------------------+--------------------+

Skips the next instruction if the bit specified by the immediate operand is
clear. The skip takes place after the end of the whole skip instruction (so it
works proper even if an addressing mode needing a second opcode word is used).
Skips a single opcode only, so if the skipped instruction has more words, the
tail of it is executed (normally these are NOPs).

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)


XBS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1010 11ii iiaa aaaa | XBS adr, imm4      |
+---------------------+--------------------+

Skips the next instruction if the bit specified by the immediate operand is
set.

For more information on the skip mechanism, check XBC.

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)


XCH
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0000 010r rraa aaaa | XCH adr, rx        |
+---------------------+--------------------+

Exchanges the value of it's operands. This happens as first loading both
operand values in temporary registers, then writing those back swapped. For
the exact operation order in conflicting cases, check "Instruction execution".

Note that by definition if the operand provided by an addressing mode is an
immediate, the XCH executes like an appropriate MOV.

Timing (cycles): 3 + ai + ma


XEQ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 100r rraa aaaa | XEQ adr, rx        |
+---------------------+--------------------+
| 1000 0001 01aa aaaa | XEQ adr, SP        |
+---------------------+--------------------+

Skips the next instruction if the value of the operands are equal.

For more information on the skip mechanism, check XBC.

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)


XNE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 101r rraa aaaa | XNE rx, adr        |
+---------------------+--------------------+
| 1000 0011 01aa aaaa | XNE SP, adr        |
+---------------------+--------------------+

Skips the next instruction if the value of the operands are not equal.

For more information on the skip mechanism, check XBC.

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)


XOR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0111 001r rraa aaaa | XOR rx, adr        |
+---------------------+--------------------+
| 0111 000r rraa aaaa | XOR adr, rx        |
+---------------------+--------------------+

Performs binary exclusive OR between the source and destination operands, and
stores the result in the destination: ::

    XOR dst, src
    dst = dst ^ src;

Timing (cycles): 3 + ai + ma


XSG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 011r rraa aaaa | XSG rx, adr        |
+---------------------+--------------------+
| 1011 010r rraa aaaa | XSG adr, rx        |
+---------------------+--------------------+

Skips the next instruction if the value of the first operand is 2's complement
signed greater than the second. A complementing operation (signed less than)
may be provided by swapping the operand order, for which an XSL mnemonic may
be supported.

For more information on the skip mechanism, check XBC.

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)


XUG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 111r rraa aaaa | XUG rx, adr        |
+---------------------+--------------------+
| 1011 110r rraa aaaa | XUG adr, rx        |
+---------------------+--------------------+
| 1000 0010 11aa aaaa | XUG SP, adr        |
+---------------------+--------------------+
| 1000 0000 11aa aaaa | XUG adr, SP        |
+---------------------+--------------------+

Skips the next instruction if the value of the first operand is unsigned
greater than the second. A complementing operation (unsigned less than) may be
provided by swapping the operand order, for which an XUL mnemonic may be
supported.

For more information on the skip mechanism, check XBC.

Timing (cycles): 3 + ai + ma (no skip) / 5 + ai + ma (skip)




Instruction matrix
------------------------------------------------------------------------------


The following instruction matrix sorts instructions by the high 7 bits. Note
that the ordering is somewhat mixed to respect the structure of the opcode
layout. The columns group by the highest two bits (bit 15 and bit 14) and bit
9. The rows group by bits 13, 12, 11 and 10.

+----+---------+---------+----------+----------+---------+---------+---------+
|    | 00....0 | 00....1 | 01....0  | 01....1  | 10....0 | 10....1 | 11..... |
+====+=========+=========+==========+==========+=========+=========+=========+
|    || MOV    || MOV    || MOV     || MOV     || MOV    || MOV    |         |
|0000|| adr, rx|| rx, adr|| adr, xmn|| xmn, adr|| adr, XM|| XM, adr|   NOP   |
|    |         |         || adr, xbn|| xbn, adr|| adr, XB|| XB, adr|         |
|    |         |         |          |          || SP ops || SP ops |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || XCH    || MOV    || JFR     || MOV     || JMR    || MOV    |         |
|0001|| adr, rx|| rx, imx|| JFA     || rx, imx || JMA    || rx, imx|         |
|    |         |         || JSV     |          |         |         |         |
|    |         |         || RFN     |          |         |         |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || ADD    || ADD    || ADD     || ADD     |   JNZ rx, simm7   |         |
|0010|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|                   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || SUB    || SUB    || SUB     || SUB     |     JMS simm10    |         |
|0011|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|                   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || ASR    || ASR    || ASR     || ASR     |                   |         |
|0100|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|     Supervisor    |         |
+----+---------+---------+----------+----------+                   |         |
|    || DIV    || DIV    || DIV     || DIV     |                   |         |
|0101|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|                   |         |
+----+---------+---------+----------+----------+                   |         |
|    || ADC    || ADC    || ADC     || ADC     |                   |         |
|0110|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|                   |         |
+----+---------+---------+----------+----------+                   |         |
|    || SBC    || SBC    || SBC     || SBC     |                   |         |
|0111|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|                   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || NOT    || NOT    || NEG     || NEG     |                   |         |
|1000|| adr, rx|| rx, adr|| adr, rx || rx, adr |   BTC adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || MUL    || MUL    || MUL     || MUL     |                   |         |
|1001|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   XBC adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || SHR    || SHR    || SHR     || SHR     |                   |         |
|1010|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   BTS adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || SHL    || SHL    || SHL     || SHL     |                   |         |
|1011|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   XBS adr, imm4   |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || OR     || OR     || XOR     || XOR     || AND    || AND    |         |
|1100|| adr, rx|| rx, adr|| adr, rx || rx, adr || adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || MAC    || MAC    || MAC     || MAC     || XSG    || XSG    |         |
|1101|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || SRC    || SRC    || SRC     || SRC     || XEQ    || XNE    |         |
|1110|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || SLC    || SLC    || SLC     || SLC     || XUG    || XUG    |         |
|1111|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+---------+

The area marked as "Supervisor" is reserved for instructions for the
supervisor mode only. If the user mode attempts to execute any of them, the
result is a supervisor trap (interrupt). By the RRPGE system specification
this means the application is terminated.
