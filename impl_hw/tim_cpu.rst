
RRPGE CPU pipeline and timing details
==============================================================================

.. image:: https://cdn.rawgit.com/Jubatian/rrpge-spec/00.013.002/logo_txt.svg
   :align: center
   :width: 100%

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


In this document a coarse overview is provided for the RRPGE CPU's internals,
roughly describing how the timing characteristics outlined in the
specification may be realized.

The RRPGE CPU is described as a simple, weakly pipelined CISC design, with
multiple clock cycles taken for each instruction. This guide assigns tasks to
these clock cycles, also describing parallel operations where such is
appropriate.

The weak pipelining is provided at instruction fetch and decode which
operations may be performed in parallel with other steps in the preceding
instruction's execution.

In the cycle break-downs, the following abbreviations are used to describe
various steps:

- O1: Opcode load (first word)
- O2: Opcode load (second word, if any)
- DC: Opcode decode
- XP: Post-increment (memory address modes, where appropriate)
- XO: Address calculation (memory address modes)
- RM: Memory read
- RR: Register read
- AA: Arithmetic operation
- WM: Memory write
- WR: Register write
- WC: Carry write
- SR: Stack read (as part of a stack pop operation)
- SW: Stack write (as part of a stack push operation)
- PU: Program Counter update calculation (jumps and skips)
- PC: Program Counter overwrite (with pipeline flush)

Operations which access some type of memory are marked with an asterisk ('*').
These can not be performed in parallel. Optional operations are marked with a
tilde ('~'): these, depending on the instruction's format, may not be present.




Common properties
------------------------------------------------------------------------------


Pipeline
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The CPU is assumed to be capable to perform simple pipelining by fetching and
decoding the subsequent instruction while the current one is still executing.
This saves 2 cycles in each instruction's execution time as long as the
pipeline is operable.

For example, in the case of MOV rx, rx, the pipeline operates the following
way: ::

          0      1      2      3      4      5
    0    O1* |  DC  |  RR  |  WR  |      |
    0        |      |      |      |      |
    1        |  O1*------> |  DC  |  RR  |  WR

The effective throughput is 1 instruction every 2 cycles. The second
instruction's opcode fetch is performed first time during the first
instruction's decode step (and might occur again if it is invalidated for some
reason, for example due to loading a second opcode word for the first
instruction). The second instruction's decode step may occur first time in
parallel with the write-back of the first instruction.

The pipelining opcode fetches are of low priority: they happen whenever the
memory bus is free (not used by the currently executing instruction if any),
and an opcode has to be loaded.


Arithmetic (AA) cycles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Arithmetic cycles vary depending on the instruction used. They are as follows:

- 0 cycle: MOV, NOT
- 1 cycle: ADD, ADC, AND, ASR, BTC, BTS, NEG, OR, SBC, SHL, SHR, SLC, SRC,
  SUB, XBC, XBS, XEQ, XNE, XOR, XSG, XUG
- 10 cycles: MUL
- 11 cycles: MAC
- 18 cycles: DIV

When determining an instruction's timing, these values should be used in the
place of 'AA'.


Relation of address calculation and increment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The address calculation (XO) is always carried out before the post-increment
(XP), so the proper memory address is used. Note that the post-increment has
no subsequent effect on the calculated address: it only affect the register it
targets.


Relation of write-backs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To maintain the appropriate priority order, first the effect of XP (pointer
post-increment) has to be written back, then the destination, and finally, the
carry.




Instruction break-downs
------------------------------------------------------------------------------


Register source, Memory target R-M-W
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most operations in the form "OP adr, rx" imply a register source, memory
target read-modify-write operation. If "adr" is not memory, see "Register
source, Register target R-M-W" (if it is an immediate, the same applies).

Note that MOV and similar operations with no sub-word addressing where the
target does not have to be read may execute the same manner: in the RRPGE
system, optimizing out reading the target is not considered as it does not
produce significant advantage.

Bit set (BTS) and bit clear (BTC) instructions may also belong here if their
target is a memory address.

Instruction break-down: ::

          0      1      2      3      4      5      6      7
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  AA  |  WM* | ~WC  |
    0        |      |      |      |  RR  |      |      |      |
    0        |      |      |      | ~XP  |      |      |      |
    0        |      |      |      |      |      |      |      |
    1        |  O1* | ------->O1*-------------> |  DC-------> | ...

These instructions so effectively take "3 + O2 + AA + WC" cycles.


Memory source, Register target R-M-W
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Most operations in the form "OP rx, adr" imply a memory source, register
target read-modify-write operation. If "adr" is not memory, see "Register
source, Register target R-M-W" (if it is an immediate, the same applies).

Note that MOV and similar operations with no sub-word addressing where the
target does not have to be read may execute the same manner: in the RRPGE
system, optimizing out reading the target is not considered as it does not
produce significant advantage.

Instruction break-down: ::

          0      1      2      3      4      5      6      7
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  AA  |  WR  | ~WC  |
    0        |      |      |      |  RR  |      |      |      |
    0        |      |      |      | ~XP  |      |      |      |
    0        |      |      |      |      |      |      |      |
    1        |  O1* | ------->O1*-------------> |  DC-------> | ...

These instructions so effectively take "3 + O2 + AA + WC" cycles.


Register source, Register target R-M-W
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the address of a normal "OP rx, adr" or "OP adr, rx" instruction is a
register or an immediate, no data memory access is performed, with no
address calculation.

Note that MOV and similar operations where the target does not have to be
read may execute the same manner: in the RRPGE system, optimizing out reading
the target is not considered as it does not produce significant advantage.

Immediate source also belongs to this layout: a second opcode word may be
present only this case in this layout.

Bit set (BTS) and bit clear (BTC) instructions may also belong here if their
target is a register.

Instruction break-down: ::

          0      1      2      3      4      5      6
    0    O1* |  DC  | ~O2* |  RR  |  AA  |  WR  | ~WC  |
    0        |      |      |  RR  |      |      |      |
    0        |      |      |      |      |      |      |
    1        |  O1* | ------->O1*------> |  DC-------> | ...

These instructions so effectively take "2 + O2 + AA + WC" cycles.


XCH instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If one of the operands of XCH is a memory address, the following break-down
may be assumed: ::

          0      1      2      3      4      5      6
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  WM* |  WR  |
    0        |      |      |      |  RR  |      |      |
    0        |      |      |      | ~XP  |      |      |
    0        |      |      |      |      |      |      |
    1        |  O1* | ------->O1*------> |  DC-------> | ...

Otherwise the following applies: ::

          0      1      2      3      4      5
    0    O1* |  DC  | ~O2* |  RR  |  WR  |  WR  |
    0        |      |      |  RR  |      |      |
    0        |      |      |      |      |      |
    1        |  O1* | ------->O1* |  DC-------> | ...

The timing of the XCH instruction is so identical to those of "normal"
instructions where the carry is written, except that there is no AA cycle
(similar to MOV).


NOP instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Normal NOPs (bits 14 and 15 set in the opcode word) effectively take 1 cycle
to execute: they are over with the decode (DC) operation which pipelines with
the next instruction's opcode fetch.


Skip instructions (XBC, XBS, XEQ, XNE, XSG, XUG)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When the skip is not taken, the following break-down may be assumed with a
memory operand: ::

          0      1      2      3      4      5      6
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  AA  |      :
    0        |      |      |      |  RR  |      |      :
    0        |      |      |      | ~XP  |      |      :
    0        |      |      |      |  PU  |      |      :
    0        |      |      |      |      |      |      :
    1        |  O1* | ------->O1*-------------> |  DC  | ...

Note that the pipelining is not perfect since the next instruction's opcode
decode cycle does not complete in parallel with any of the skip's operations.

The instruction (with no skip) so effectively takes "2 + O2 + XO + AA" cycles.

If the skip is taken, the following break-down takes place instead: ::

          0      1      2      3      4      5      6      7      8
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  AA  |  PC  |      :      :
    0        |      |      |      |  RR  |      |      |      :      :
    0        |      |      |      | ~XP  |      |      |      :      :
    0        |      |      |      |  PU  |      |      |      :      :
    0        |      |      |      |      |      |      |      :      :
    1        |  O1* | ------->O1*--------------------> |  O1* |  DC  | ...

Due to flushing the pipeline (after performing the arithmetic operation), two
additional cycles are introduced.

Note that the Program Counter update may be performed in either cycle before
the Program Counter overwrite.


JNZ instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When the jump is not taken, the following break-down may be assumed: ::

          0      1      2      3
    0    O1* |  DC  |  RR  |      :
    0        |      |  PU  |      :
    0        |      |      |      :
    1        |  O1*------> |  DC  | ...

The instruction so effectively takes 2 cycles (there is no AA cycle since for
equality compares this cycle count is zero).

When the jump is taken, the following takes place instead: ::

          0      1      2      3      4      5
    0    O1* |  DC  |  RR  |  PC  |      :      :
    0        |      |  PU  |      |      :      :
    0        |      |      |      |      :      :
    1        |  O1*-------------> |  O1* |  DC  | ...

Note that the calculation of the new PC should be performed parallel with the
register load and compare (similar to skip instructions).


JMS instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The JMS instruction may use the following break-down: ::

          0      1      2      3      4      5
    0    O1* |  DC  |  PU  |  PC  |      :      :
    0        |      |      |      |      :      :
    1        |  O1*-------------> |  O1* |  DC  | ...

Essentially this is the same as a taken JNZ.


Absolute and relative jumps (JMA, JMR)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The absolute and relative jumps may need to store a return address, which is
indicated by an optional "WR" (Register Write) operation here. The break-down
of these instructions may form as follows with a memory address: ::

          0      1      2      3      4      5      6      7      8
    0    O1* |  DC  | ~O2* |  XO  |  RM* | ~PU  |  PC  | ~WR  |      :
    0        |      |      |      | ~XP  |      |      |      |      :
    0        |      |      |      |      |      |      |      |      :
    1        |  O1* | ------->O1*--------------------> |  O1* |  DC  | ...

Note that absolute jumps don't need a "PU" cycle (they write their source
directly into the Program Counter).

The instructions so take effectively "4 + O2 + XO + PU" cycles.


Function call instructions (JFA, JFR, JSV)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A function call needs to perform several operations along with it's regular
tasks of loading the operand (call address). These are marked with '#n',
explained after the break-downs.

Function call with no parameter, register or immediate operand (the latter
is the most common in usual calls): ::

          0      1      2      3      4      5      6      7
    0    O1* |  DC  | ~O2* |  RR  | ~PU  |  PC  |      :      :
    0        |      |      |  #1  |  #2* |  #3* |      :      :
    0        |      |      |      |      |      |      :      :
    1        |  O1* | ------->O1*-------------> |  O1* |  DC  | ...

Function call with no parameter, memory operand: ::

          0      1      2      3      4      5      6      7      8
    0    O1* |  DC  | ~O2* |  XO  |  RM* | ~PU  |  PC  |      :      :
    0        |      |      |      | ~XP  |      |      |      :      :
    0        |      |      |      |  #1  |  #2* |  #3* |      :      :
    0        |      |      |      |      |      |      |      :      :
    1        |  O1* | ------->O1*--------------------> |  O1* |  DC  | ...

Function call with parameters, register or immediate operand (the latter
is the most common in usual calls): ::

          0      1      2      3      4      5
    0    O1* |  DC  | ~O2* |  RR  | ~PU  |      |
    0        |      |      |  #1  |  #2* |      |
    0        |      |      |      |      |      |
    1        |  O1* | ------->O1*------> |  DC  | ...

The next instruction is the first parameter. It pipelines cleanly with the
function call instruction's execution.

A non-terminating parameter load from memory: ::

          0      1      2      3      4      5
    0    O1* |  DC  | ~O2* |  XO  |  RM* |  SW* |
    0        |      |      |      | ~XP  |      |
    0        |      |      |      |      |      |
    1        |  O1* | ------->O1*------> |  DC  | ...

A parameter so takes effectively "2 + O2 + XO" cycles. (In case of reading
from register, the next opcode fetch would execute in parallel with the "RR"
operation)

A terminating parameter load from a register or immediate: ::

          0      1      2      3      4      5      6      7
    0    O1* |  DC  | ~O2* |  RR  |  SW* |  PC  |      :      :
    0        |      |      |      |      |  #3* |      :      :
    0        |      |      |      |      |      |      :      :
    0        |      |      |      |      |      |      :      :
    1        |  O1* | --------O1*-------------> |  O1* |  DC  | ...

The pipeline flush so essentially always adds 3 cycles to the function call's
execution time, irrespective of whether it happens after parameter loads or
the call itself (no parameters).

Function calls so take effectively "5 + O2 + XO" cycles without parameters.

The operations special to function calls:

- #1: Store location for saving return address, increment SP.
- #2: Push BP on the stack.
- #3: Write return address on the stack.

Establishing the new stack frame also has to be performed at some point,
however this should be possible within these timing constraints, in parallel
with other operations.


RFN instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

With function returns the primary operations to perform are reverting the
stack frame, loading the return address, and producing the return value in
C:X3 as requested in the instruction.

The operation break-down may look like as follows with a register or immediate
source: ::

          0      1      2      3      4      5      6      7      8
    0    O1* |  DC  | ~O2* |  RR  |  WR  | ~WC  |  PC  |      :      :
    0        |      |      |      |  SR* |  SR* |      |      :      :
    0        |      |      |      |      |      |      |      :      :
    1        |  O1* | ------->O1*--------------------> |  O1* |  DC  | ...

The stack reads are for popping BP to restore caller's stack frame, and the
return address. The "WR" and "WC" operations write X3 and C, the latter only
if requested by the instruction.

Returns so effectively take "6 + O2 + XO" cycles.

Note that depending on how the pipeline is realized, a premature "DC" of the
next instruction may happen in parallel with "WR". This is irrelevant.


PSH instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The instruction's break-down may look like as follows for pushing three
registers: ::

          0      1      2      3      4      5      6
    0    O1* |  DC  |  #1  |  RR  |  RR  |  RR  |  SW* |
    0        |      |      |      |  SW* |  SW* |      |
    0        |      |      |      |      |      |      |
    1        |  O1* --------------------------> |  DC  | ...

The "#1" marked operation is meant to decode the register list. The push
effectively takes 2 cycles, plus one for each register stored.


POP instruction
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The instruction's break-down may look like as follows for popping three
registers: ::

          0      1      2      3      4      5      6
    0    O1* |  DC  |  #1  |  SR* |  WR  |  WR  |  WR  |
    0        |      |      |      |  SR* |  SR* |      |
    0        |      |      |      |      |      |      |
    1        |  O1* --------------------------> |  DC  | ...

The "#1" marked operation is meant to decode the register list. The pop
effectively takes 2 cycles, plus one for each register loaded.
