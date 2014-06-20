
RRPGE CPU architecture
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


The RRPGE CPU is a 16bit CISC design with the following fundamental features:

- Word sized memory accesses, base unit for the CPU is a word (16 bit unit).
  Sub-word accesses however remain possible through special memory addressing
  modes.

- Built-in memory management unit and separate address space for code, data
  read, data write and stack accesses enabling a large address space despite
  the 16 bit pointer size and allowing for fine-grained access control by the
  supervisor.

- No flags in user mode. Carry mechanism is provided through a carry register
  and instruction variants affecting it, other common flags on other
  architectures have no direct equivalent. A set of conditional skip
  instructions are provided to realize similar functionality.

- Extended subroutine entry and return operations complete with parameter
  passing and stack management. Compact one word addressing mode is provided
  for time and code size efficient reach for parameters and limited work area
  on stack.

- Addressing modes for sub-word accesses using read-modify-write logic. Bytes,
  nybbles, 2bit and bit units may be addressed by these modes.




Supervisor mode
------------------------------------------------------------------------------


The RRPGE system specification only defines the user mode of the CPU as this
is the only mode visible for an application. The supervisor mode is only
covered where it interfaces with the user mode, any other extent of it is
left implementation defined.

For hardware implementations requiring the supervisor mode implemented the
documents in the "impl_hw" directory may be consulted.

The interfacing areas are supervisor calls and returns and interrupt handlers.
These are covered in the respective sections.




Register set
------------------------------------------------------------------------------


The following table summarizes the processor registers available in user mode:

+----+------+---------+------------------------------------------------------+
| ID | Size | Group   | Notes                                                |
+====+======+=========+======================================================+
| A  | 16   |         | - For all arithmetic & logic operations.             |
+----+------+ General +------------------------------------------------------+
| B  | 16   | purpose | - For all arithmetic & logic operations.             |
+----+------+         +------------------------------------------------------+
| C  | 16   |         | - Carry register for carry operations.               |
|    |      |         | - For all arithmetic & logic operations.             |
+----+------+         +------------------------------------------------------+
| D  | 16   |         | - For all arithmetic & logic operations.             |
+----+------+---------+------------------------------------------------------+
| X0 | 16   |         | - Pointer in addressing modes.                       |
|    |      | Pointer | - For all arithmetic & logic operations.             |
+----+------+         +------------------------------------------------------+
| X1 | 16   |         | - Pointer in addressing modes.                       |
|    |      |         | - For all arithmetic & logic operations.             |
+----+------+         +------------------------------------------------------+
| X2 | 16   |         | - Pointer in addressing modes.                       |
|    |      |         | - For all arithmetic & logic operations.             |
+----+------+         +------------------------------------------------------+
| X3 | 16   |         | - Pointer in addressing modes.                       |
|    |      |         | - For all arithmetic & logic operations.             |
+----+------+---------+------------------------------------------------------+
| XM | 4*4  |         | - Addressing mode specifier for the pointer          |
|    |      | Pointer |   registers. Lowest bits refer to X0, highest to X3. |
+----+------+ config  +------------------------------------------------------+
| XH | 4*4  |         | - High bits for the pointer registers. Lowest bits   |
|    |      |         |   refer to X0, highest to X3.                        |
+----+------+---------+------------------------------------------------------+
| PC | 16   |         | - Program counter. Not accessible directly.          |
+----+------+---------+------------------------------------------------------+
| SP | 16   |         | - Stack pointer. Some arithmetic are valid for it.   |
+----+------+ Stack   +------------------------------------------------------+
| BP | 16   |         | - Frame pointer. Not accessible directly.            |
+----+------+---------+------------------------------------------------------+

The General purpose and Pointer registers are accessible for all arithmetic
and logic operations as register operands.

The Pointer config registers are only accessible in load and store (MOV)
operations as register operands. The 4bit parts of these registers may be
accessed individually for altering the configuration of a single pointer
without disturbing the rest.

The XM register specifies the mode of the pointer which is used when the
pointer is used to address memory. There are 15 modes as follows:

- 0000b:  8 bit stationary.
- 0001b:  4 bit stationary.
- 0010b:  2 bit stationary.
- 0011b:  1 bit stationary.
- 0100b: 16 bit stationary.
- 0101b: 16 bit stationary (same as above).
- 0110b: 16 bit post-incrementing.
- 0111b: 16 bit pre-decrementing.
- 1000b:  8 bit post-incrementing.
- 1001b:  4 bit post-incrementing.
- 1010b:  2 bit post-incrementing.
- 1011b:  1 bit post-incrementing.
- 1100b:  8 bit pre-decrementing.
- 1101b:  4 bit pre-decrementing.
- 1110b:  2 bit pre-decrementing.
- 1111b:  1 bit pre-decrementing.

The XH register adds high bits to the pointers in sub-word addressing modes,
as many as required to keep addressing the whole 64KWord address space. Bits
in the register above this are not used and are ignored. In pre-incrementing
or post-decrementing modes only the used bits are affected as well.

Sub-word addressing modes take the higher bits for the lower address (that is
the system is Big Endian).

The PC (Program Counter) increments before executing the instruction;
instructions referring to it (subroutine entries and skips) will see it
pointing at the following instruction (or the parameter list of the subroutine
entry). Note that some instructions also require the address of the opcode,
that is the original value of the PC (local jumps and calls and relative
jumps).

The Stack registers are described in the Stack management section.




Address spaces and Memory management unit
------------------------------------------------------------------------------


The RRPGE CPU core accesses memory as 16bit words by 16 address lines in four
distinct address spaces:

- Code space. Opcode fetches happen from this address space.

- Stack space. Stack addressing modes access in this address space for both
  reads and writes.

- Data read space. Data read accesses happen from this address space.

- Data write space. Data write accesses happen into this address space.

Each of the address spaces are divided into 16 pages of 4096 words each.

The memory management unit maps pages into the address spaces of the CPU core
from a common pool of 64K pages (giving a total of 256MWords of address space
or 28 address lines).

Note that the memory management unit itself is only accessible to the
supervisor mode, so the possible paging configurations are limited by the
kernel to the pages which are supposed to be available for the user mode.

For the user mode the kernel only exposes a possibility to page in and out
from or to the Data read and Data write spaces. However on a real hardware
implementation the Stack and Code spaces are also paged which is exploited by
supervisor - user mode switches.

The CPU core can not use the address spaces simultaneously (like a Harvard
architecture would be able) as these are supposed to map from a common address
space with a single set of address lines at hardware level.




Addressing stalls
------------------------------------------------------------------------------


The processor has at least two distinct stall lines referring to the low and
high halves of the common page pool (low half are pages 0x0000 - 0x7FFF; high
half are pages 0x8000 - 0xFFFF). These stall lines are activated by the Mixer
DMA and the audio output DMA on the lower half, and by the Graphics Display &
Accelerator unit on the higher half.

If a stall line is active and the CPU addresses into the respective half, it
must wait until the peripherals holding the stall lines release them.

The stalls incurred by the audio output DMA are minimal and left unspecified
in this system specification. They should be calculated in the limitations
imposed on the cycles consumed by internal kernel tasks (see "kernel.rst" for
further details).




Memory accessing
------------------------------------------------------------------------------


The processor is capable to access memory in two ways:

- Read: A read access is performed to fetch the 16bit data from a given 16 bit
  word address.

- Read-Modify-Write: A read access is performed to fetch the 16bit data from a
  given 16 bit word address, an operation is performed (not necessarily
  actually using the read data), then the result is written to the given
  address.

Note that any writes so are accompanied with a read from the same 16 bit
address.

Stack push operations are also affected, but the read data is always
discarded. Implementations are allowed to omit these reads as by the
specification these reads can never have side effects (the stack is always
located in ordinary data memory).

In the case of accessing data the read access always happens from the read
page, and the write access always happens to the write page. If the two does
not match, the result may be unexpected but defined. The implementation must
behave accordingly this case.




Stack management
------------------------------------------------------------------------------


Stack memory is implemented using a distinct address space, the SP and BP
registers, and two supervisor mode registers specifying user mode stack
bounds.

The stack grows upwards, post-incrementing.

The BP register is the frame pointer which points at the bottom of the current
subroutine's stack frame. Addressing modes use BP to reach subroutine local
data in the stack.

The SP register specifies the frame size of the subroutine. Code may alter
this register for manipulating this frame size. When new subroutines are
called, after pushing the return address and the current BP, the newly
called subroutine's BP will be set equal the current one with SP added. ::

    | (...)      |
    +------------+
    | PC         | <- SP(caller) when the call is made
    +------------+
    | BP(caller) |
    +------------+
    |            | <- SP after pushes; BP(sub) = SP
    +------------+
    | (...)      |

When returning from subroutines, the current SP is simply discarded (it is not
necessary to restore it), and the previous function's BP and SP are restored
based on the current BP and the BP pushed on the stack at entry. ::

    | (...)      |
    +------------+
    | PC         | <- SP(caller) = BP(sub) - 2
    +------------+
    | BP(caller) |
    +------------+
    |            | <- BP(sub) before return
    +------------+
    | (...)      |

In user mode there are no direct push and pop operations, however subroutine
entries, returns and parameter passings realize identical mechanisms. The
primary use of the stack is providing an efficient parameter and local
variable storage for subroutines supporting reentrancy.

Two additional supervisor mode registers are provided specifying user mode
stack top and stack bottom. When stack accesses are generated outside these
bounds, it is trapped (through an identical mechanism to interrupts) by the
kernel which terminates the offending application.

The only exception is executing a subroutine return with having BP equal the
stack bottom. This operation passes control back to the supervisor mode.

In the RRPGE system the stack top is fixed at 0x8000 (32768). The stack bottom
is zero for the main program, for user mode program execution in interrupts,
see the Interrupts section.




User - Supervisor mode switches
------------------------------------------------------------------------------


When switching from supervisor to user mode and vice-versa, certain hardware
supported page flips and register replacements necessarily happen. The minimal
set of resources necessary to be automatically handled:

- One defined supervisor mode Code space page which is banked in as Code page
  0 when returning to supervisor mode through a subroutine return at stack
  bottom, or entering a supervisor call or interrupt (these may be different
  offsets within this page, the first fetching a PC from the supervisor stack,
  the second and third a fixed address).

- One defined supervisor mode Stack space page which is banked in as Stack
  page 0 when returning to supervisor mode through a subroutine return at
  stack bottom, or entering a supervisor call (this receives supervisor call
  parameters then) or interrupt.

- One defined user mode Code space page to bank into Code page 0 when entering
  or returning to user mode (the rest may be handled by kernel code as
  necessary).

- One defined user mode Stack space page to bank into Stack page 0 when
  entering or returning to user mode (the rest may be handled by kernel code
  as necessary).

- Similar to the above shadow user and supervisor mode SP and BP registers
  have to be provided.

- To allow the supervisor mode reasonably saving the state of the user mode
  register set, a similarly shadowed general purpose register (preferably A)
  has to be provided.

These requirements are guidelines only, a software emulator not necessarily
needs these information for realizing a conforming RRPGE system
implementation.




Interrupts
------------------------------------------------------------------------------


The RRPGE system does not provide user mode interrupts, so the followings are
optional design guidelines only.

Interrupts always enter into supervisor mode; where necessary, the supervisor
mode may pass control back to user mode for running an user level handler.

If the interrupt entry condition raises while the processor is running in user
mode, an user-supervisor mode switch is performed first before starting the
handler. This condition is necessarily remembered, and a matching supervisor-
user mode switch is performed on exiting the interrupt.

The entry-return logic automatically pushes (entry) and pops (return) the
necessary minimal state on the supervisor stack.

If the supervisor mode program will be executing an user mode handler, it
should save the user mode state of the main line of the user mode program
before entry, and restores that state after return.

Before entering an user mode interrupt handler, the stack bottom should be set
up to the current (user mode) top of the stack (BP + SP) in the mainline (or
lower level interrupt), so the user mode handler may properly return with a
return from function operation.




Addressing modes
------------------------------------------------------------------------------


The RRPGE CPU's instruction set contains a single unified method of addressing
encoded on the low six bits of any opcode using an address. Additionally only
the addressing mode may pull in an additional opcode word for 16 bit
immediates.

Instructions may include only up to one operand specified by an addressing
mode, the other operand (if any) is always a general purpose register (or some
special registers in some cases).

The following eight addressing modes are implemented:

- General purpose register. One of A, B, C, D, X0, X1, X2 or X3.

- 4 bit immediate. This specifies an immediate value of 0 - 15.

- 16 bit immediate. Specifies an immediate value in the full 16 bit range, but
  needs an extra instruction word (and one additional cycle to decode).

- Stack: BP + 4 bit immediate. Accesses a 16 bit unit from the Stack address
  space. This addressing mode is suitable for accessing the parameters of a
  subroutine.

- Stack: BP + 16 bit immediate. Accesses a 16 bit unit from the Stack address
  space reaching the full address space, but needs an extra instruction word
  (and one additional cycle to decode).

- Stack: BP + Pointer. Accesses an unit from the Stack address space as
  specified by the given pointer register's mode, post-incrementing or pre-
  decrementing the pointer register if such mode was set.

- Data: 16 bit immediate. Accesses a 16 bit unit from the Data address space
  reaching it's full range. Needs an extra instruction word.

- Data: Pointer. Accesses an unit from the Data address space as specified by
  the given pointer register's mode, post-incrementing or pre-decrementing the
  pointer register if such mode was set.

Note that the 4 bit immediate and 16 bit immediate modes may also be used as
destinations. Doing so realizes an essential NOP, although any side effect of
the operation is still carried out (such as writing the Carry register for
operations affecting it).

In Pointer modes the high 16 of the used pointer bits are used to address the
16 bit cells. An example shows this concept with the following parameters:

- X0 = 0x1002
- XM = 0x...1 (4 bit stationary mode)
- XH = 0x...2 ::

    0x83FF  |    (((XH0 & 0x3) << 16) + X0) >> 2 = 0x8400   |  0x8401
    --------+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--------
            |15|14|13|12|11|10| 9| 8| 7| 6| 5| 4| 3| 2| 1| 0|
            | 0| 1| 1| 0| 0| 0| 1| 1| 0| 1| 1| 0| 0| 1| 1| 0|
    --------+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--------
            |     0     |     1     |     2     |     3     |
            |           |           |  X0 & 0x3 |           |

When writing data to sub-word addressing mode accessed cells, the Read -
Modify - Write logic of the data writes realizes the effect of only altering
the appropriate sub-unit of the word.
