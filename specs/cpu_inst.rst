
RRPGE CPU instruction set
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
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
- xhn:   One of the 4bit parts of the XH register.
- imm4:  4bit immediate ranging from 0 - 15.
- imm16: 16bit immediate.
- adr:   Parameter specified by the addressing mode field.


Timing notations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- ai: 1 additional cycle if 2nd opcode word is used.
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
| 1000 101- --aa aaaa | ADD SP, adr        |
+---------------------+--------------------+

Adds the source operand to the destination, and stores the result in the
destination: ::

    ADD dst, src
    dst = dst + src;

If Carry was specified, it will store 1 if the add generated a carry, 0
otherwise.

Timing (cycles): 4 + ai + wc


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

Timing (cycles): 4 + ai + wc


AND
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 011r rraa aaaa | AND rx, adr        |
+---------------------+--------------------+
| 0100 010r rraa aaaa | AND adr, rx        |
+---------------------+--------------------+

Performs binary AND between the source and destination operands, and stores
the result in the destination: ::

    AND dst, src
    dst = dst & src;

Timing (cycles): 4 + ai


ASR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 011r rraa aaaa | ASR rx, adr        |
+---------------------+--------------------+
| 0011 010r rraa aaaa | ASR adr, rx        |
+---------------------+--------------------+
| 0111 011r rraa aaaa | ASR C:rx, adr      |
+---------------------+--------------------+
| 0111 010r rraa aaaa | ASR C:adr, rx      |
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

Timing (cycles): 4 + ai + wc


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

Timing (cycles): 4 + ai


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

Timing (cycles): 4 + ai


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

Timing (cycles): 21 + ai + wc


JFR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 1000 0eaa aaaa | JFR adr {...}      |
+---------------------+--------------------+

Relative function call (subroutine entry). The target address is calculated by
adding the operand to the current PC which points to the JFR instruction.

The stack receives the PC pointing after the function call opcode, then the
current BP, after which the called function's stack frame is established.
For more information, see "Stack Management" in "cpu_arch.rst".

Note that the caller's stack frame is remembered for passing parameters from
the caller's stack.

After the function call opcode (including the additional opcode word if it was
necessary by the addressing mode) up to 16 parameters may follow which are
pushed on the called function's stack. The parameter opcode format is as
follows: ::

    ---- ---- -eaa aaaa
    (Normally first two bits should be 11 for making these NOPs)

The 'e' bit (also found in the function's opcode) marks the last parameter if
if it is set. The 16th parameter ignores the 'e' bit treating it set.

Note that as mentioned above, if stack space addressing is used in the
parameter opcodes, the parameter is taken from the caller's stack (and pushed
onto the new stack frame created for the subroutine).

Also note that the stored PC points after the function call's opcode, so the
parameter list will be executed as normal opcodes after return. They should
be formatted as NOPs to prevent this producing ill effects.

An example call with 3 parameters: ::

    0: JFL func {10, [20], [X0]}

    0: 1000 1000 0010 0000 -- JFL opcode with imm16 address
    1: 1100 ffff ffff ffff -- Address (low 12 bits effective) of function
    2: 1100 0000 0000 1010 -- Parameter 10 decimal as imm4 addressing mode
    3: 1100 0000 0010 1000 -- Parameter [20], first byte
    4: 1100 0000 0001 0100 -- Second byte
    5: 1100 0000 0111 1000 -- Parameter [X0], final parameter ('e' bit set)

    | (...)       |
    +-------------+
    | PC (at 2)   | Saved return address, pointing at first parameter
    +-------------+
    | BP (caller) |
    +-------------+--> End of caller's stack frame
    | 10          | <- BP; first parameter's value
    +-------------+
    | pppp        | Second parameter, value read from [20].
    +-------------+
    | pppp        | Third parameter, value read from [X0].
    +-------------+
    |             | <- SP
    +-------------+
    | (...)       |

Timing (cycles): 9 + ai; 4 + ai / parameter


JFA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 1001 0eaa aaaa | JFA adr {...}      |
+---------------------+--------------------+

Absolute function call (subroutine entry). The target address is the operand.

See JFL for details.

Timing (cycles): 9 + ai; 4 + ai / parameter


JMR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 1100 --aa aaaa | JMR adr            |
+---------------------+--------------------+

Relative jump. The target address is calculated by adding the operand to the
current PC which points to the JMR instruction.

Timing (cycles): 5 + ai


JMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 1101 --aa aaaa | JMA adr            |
+---------------------+--------------------+

Absolute jump. The target address is the operand.

Timing (cycles): 5 + ai


JMS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 01ii iiii iiii | JMS simm10         |
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
| 1000 1000 1e-- ---- | JSV {...}          |
+---------------------+--------------------+

Supervisor call.

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
| 0011 001r rraa aaaa | MAC rx, adr        |
+---------------------+--------------------+
| 0011 000r rraa aaaa | MAC adr, rx        |
+---------------------+--------------------+
| 0111 001r rraa aaaa | MAC C:rx, adr      |
+---------------------+--------------------+
| 0111 000r rraa aaaa | MAC C:adr, rx      |
+---------------------+--------------------+

Multiply and accumulate. Multiplies the destination with the source operand,
then adds carry, and stores the result in the destination: ::

    MAC dst, src
    dst = dst * src + C;

If Carry was specified, it receives the high 16 bits of the result.

Timing (cycles): 14 + ai + wc


MOV
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0000 001r rraa aaaa | MOV rx, adr        |
+---------------------+--------------------+
| 0000 000r rraa aaaa | MOV adr, rx        |
+---------------------+--------------------+
| 0000 0110 nnaa aaaa | MOV xmn, adr       |
+---------------------+--------------------+
| 0000 0100 nnaa aaaa | MOV adr, xmn       |
+---------------------+--------------------+
| 0000 0111 nnaa aaaa | MOV xhn, adr       |
+---------------------+--------------------+
| 0000 0101 nnaa aaaa | MOV adr, xhn       |
+---------------------+--------------------+
| 1000 001- 00aa aaaa | MOV XM, adr        |
+---------------------+--------------------+
| 1000 000- 00aa aaaa | MOV adr, XM        |
+---------------------+--------------------+
| 1000 001- 01aa aaaa | MOV XH, adr        |
+---------------------+--------------------+
| 1000 000- 01aa aaaa | MOV adr, XH        |
+---------------------+--------------------+
| 1000 001- 1-aa aaaa | MOV SP, adr        |
+---------------------+--------------------+
| 1000 000- 1-aa aaaa | MOV adr, SP        |
+---------------------+--------------------+

Moves from source to target.

When the source is a 4bit part of the XM (xmn) or XH (xhn) register, the
destination will receive the value in it's low 4 bits, and it's high 12 bits
are set zero.

When the destination is a 4bit part of the XM (xmn) or XH (xhn) register, the
destination (the appropriate part of XM or XH) will receive the low 4 bits of
the source.

Timing (cycles): 3 + ai


MUL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 001r rraa aaaa | MUL rx, adr        |
+---------------------+--------------------+
| 0010 000r rraa aaaa | MUL adr, rx        |
+---------------------+--------------------+
| 0110 001r rraa aaaa | MUL C:rx, adr      |
+---------------------+--------------------+
| 0110 000r rraa aaaa | MUL C:adr, rx      |
+---------------------+--------------------+

Multiplies the destination with the source operand, and stores the result in
the destination: ::

    MUL dst, src
    dst = dst * src;

If Carry was specified, it receives the high 16 bits of the result.

Timing (cycles): 13 + ai + wc


NEG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0110 011r rraa aaaa | NEG rx, adr        |
+---------------------+--------------------+
| 0110 010r rraa aaaa | NEG adr, rx        |
+---------------------+--------------------+

2's complement negates the source operand, and stores the result in the
destination: ::

    NEG dst, src
    dst = 0 - src;

Timing (cycles): 4 + ai


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
| 0010 011r rraa aaaa | NOT rx, adr        |
+---------------------+--------------------+
| 0010 010r rraa aaaa | NOT adr, rx        |
+---------------------+--------------------+

Performs a binary NOT on the source operand, and stores the result in the
destination: ::

    NOT dst, src
    dst = src ^ 0xFFFF;

Timing (cycles): 3 + ai


OR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0100 001r rraa aaaa | OR rx, adr         |
+---------------------+--------------------+
| 0100 000r rraa aaaa | OR adr, rx         |
+---------------------+--------------------+

Performs binary OR between the source and destination operands, and stores the
result in the destination: ::

    OR dst, src
    dst = dst | src;

Timing (cycles): 4 + ai


RFN
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1000 1001 1--- ---- | RFN                |
+---------------------+--------------------+

Returns from function or subroutine.

For the associated mechanisms, check the JFL opcode and the "Stack Management"
section in "cpu_arch.rst".

Timing (cycles): 6


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

Timing (cycles): 4 + ai + wc


SHL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 101r rraa aaaa | SHL rx, adr        |
+---------------------+--------------------+
| 0010 100r rraa aaaa | SHL adr, rx        |
+---------------------+--------------------+
| 0110 101r rraa aaaa | SHL C:rx, adr      |
+---------------------+--------------------+
| 0110 100r rraa aaaa | SHL C:adr, rx      |
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

Timing (cycles): 4 + ai + wc


SHR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 111r rraa aaaa | SHR rx, adr        |
+---------------------+--------------------+
| 0010 110r rraa aaaa | SHR adr, rx        |
+---------------------+--------------------+
| 0110 111r rraa aaaa | SHR C:rx, adr      |
+---------------------+--------------------+
| 0110 110r rraa aaaa | SHR C:adr, rx      |
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

Timing (cycles): 4 + ai + wc


SLC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 101r rraa aaaa | SLC rx, adr        |
+---------------------+--------------------+
| 0011 100r rraa aaaa | SLC adr, rx        |
+---------------------+--------------------+
| 0111 101r rraa aaaa | SLC C:rx, adr      |
+---------------------+--------------------+
| 0111 100r rraa aaaa | SLC C:adr, rx      |
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

Timing (cycles): 4 + ai + wc


SRC
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0011 111r rraa aaaa | SRC rx, adr        |
+---------------------+--------------------+
| 0011 110r rraa aaaa | SRC adr, rx        |
+---------------------+--------------------+
| 0111 111r rraa aaaa | SRC C:rx, adr      |
+---------------------+--------------------+
| 0111 110r rraa aaaa | SRC C:adr, rx      |
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

Timing (cycles): 4 + ai + wc


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
| 1000 111- --aa aaaa | SUB SP, adr        |
+---------------------+--------------------+

Subtracts the source operand from the destination, and stores the result in
the destination: ::

    SUB dst, src
    dst = dst - src;

If Carry was specified, it will store 0xFFFF if the subtraction generated a
borrow, 0 otherwise.

Timing (cycles): 4 + ai + wc


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

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)


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

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)


XCH
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0010 001r rraa aaaa | XCH rx, adr        |
+---------------------+--------------------+
| 0010 000r rraa aaaa | XCH adr, rx        |
+---------------------+--------------------+

Exchanges the value of it's operands. This happens as first loading both
operand values in temporary registers, then writing those back swapped. For
the exact operation order in conflicting cases, check "Instruction execution".

Note that by definition if the operand provided by an addressing mode is an
immediate, the XCH executes like an appropriate MOV.

Timing (cycles): 4 + ai


XEQ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 001r rraa aaaa | XEQ rx, adr        |
+---------------------+--------------------+
| 1011 000r rraa aaaa | XEQ adr, rx        |
+---------------------+--------------------+

Skips the next instruction if the value of the operands are equal.

For more information on the skip mechanism, check XBC.

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)


XNE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 101r rraa aaaa | XNE rx, adr        |
+---------------------+--------------------+
| 1011 100r rraa aaaa | XNE adr, rx        |
+---------------------+--------------------+

Skips the next instruction if the value of the operands are not equal.

For more information on the skip mechanism, check XBC.

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)


XOR
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 0110 001r rraa aaaa | XOR rx, adr        |
+---------------------+--------------------+
| 0110 000r rraa aaaa | XOR adr, rx        |
+---------------------+--------------------+

Performs binary exclusive OR between the source and destination operands, and
stores the result in the destination: ::

    XOR dst, src
    dst = dst ^ src;

Timing (cycles): 4 + ai


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

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)


XUG
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+---------------------+--------------------+
| Binary              | Mnemonic           |
+=====================+====================+
| 1011 111r rraa aaaa | XUG rx, adr        |
+---------------------+--------------------+
| 1011 110r rraa aaaa | XUG adr, rx        |
+---------------------+--------------------+

Skips the next instruction if the value of the first operand is unsigned
greater than the second. A complementing operation (unsigned less than) may be
provided by swapping the operand order, for which an XUL mnemonic may be
supported.

For more information on the skip mechanism, check XBC.

Timing (cycles): 4 + ai (no skip) / 5 + ai (skip)




Instruction matrix
------------------------------------------------------------------------------


The following instruction matrix sorts instructions by the high 7 bits. Note
that the ordering is somewhat mixed to respect the structure of the opcode
layout. The columns group by the highest two bits (bit 15 and bit 14) and bit
9. The rows group by bits 13, 12, 11 and 10.

+----+---------+---------+----------+----------+---------+---------+---------+
|    | 00....0 | 00....1 | 01....0  | 01....1  | 10....0 | 10....1 | 11..... |
+====+=========+=========+==========+==========+=========+=========+=========+
|    || MOV    || MOV    || OR      || OR      || MOV    || MOV    |         |
|0000|| adr, rx|| rx, adr|| adr, rx || rx, adr || adr, XM|| XM, adr|   NOP   |
|    |         |         |          |          || adr, XH|| XH, adr|         |
|    |         |         |          |          || adr, SP|| SP, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || MOV    || MOV    || AND     || AND     |                   |         |
|0001|| adr,xmn|| xmn,adr|| adr, rx || rx, adr |    JMS simm10     |         |
|    || adr,xhn|| xhn,adr|          |          |                   |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || ADD    || ADD    || ADD     || ADD     || JFR    || ADD    |         |
|0010|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| JFA    || SP, adr|         |
|    |         |         |          |          || JSV    |         |         |
|    |         |         |          |          || RFN    |         |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || SUB    || SUB    || SUB     || SUB     || JMR    || SUB    |         |
|0011|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| JMA    || SP, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || XCH    || XCH    || XOR     || XOR     |                   |         |
|0100|| adr, rx|| rx, adr|| adr, rx || rx, adr |     Supervisor    |         |
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
|    || MUL    || MUL    || MUL     || MUL     |                   |         |
|1000|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   BTC adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || NOT    || NOT    || NEG     || NEG     |                   |         |
|1001|| adr, rx|| rx, adr|| adr, rx || rx, adr |   XBC adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || SHL    || SHL    || SHL     || SHL     |                   |         |
|1010|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   BTS adr, imm4   |         |
+----+---------+---------+----------+----------+-------------------+         |
|    || SHR    || SHR    || SHR     || SHR     |                   |         |
|1011|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|   XBS adr, imm4   |         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || MAC    || MAC    || MAC     || MAC     || XEQ    || XEQ    |         |
|1100|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || ASR    || ASR    || ASR     || ASR     || XSG    || XSG    |         |
|1101|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || SLC    || SLC    || SLC     || SLC     || XNE    || XNE    |         |
|1110|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+         |
|    || SRC    || SRC    || SRC     || SRC     || XUG    || XUG    |         |
|1111|| adr, rx|| rx, adr|| C:adr,rx|| C:rx,adr|| adr, rx|| rx, adr|         |
+----+---------+---------+----------+----------+---------+---------+---------+

The area marked as "Supervisor" is reserved for instructions for the
supervisor mode only. If the user mode attempts to execute any of them, the
result is a supervisor trap (interrupt). By the RRPGE system specification
this means the application is terminated.
