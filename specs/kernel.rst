
RRPGE kernel overall specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction, the role of the kernel
------------------------------------------------------------------------------


The kernel provides simple means for accessing certain functionality of the
RRPGE system, while also realizing strict protection and information hiding.

The primary role of the kernel is hiding details which vary widely across
hosts, for example the nature of possible input devices. This allows for
integrating the RRPGE system seamlessly into most types of hosts.

All details on how the kernel is implemented is fully hidden from the RRPGE
applications, so emulators can realize it with native code without breaking
conformance with this specification.

The most prominent feature of the kernel is it's interface exposed through
supervisor calls (the "JSV" opcode). These are detailed in the "kcall.rst"
part of the specification.

On a real hardware implementation the kernel also has several internal tasks
to perform in order to be able to support it's interface, timing constraints
are defined for these to make it possible for them to conform with this
specification.




Kernel timing constraints
------------------------------------------------------------------------------


The kernel on a real hardware implementation may require CPU time for
performing internal tasks (such as managing network packets or communication
with input peripherals). It is important to constrain the processing time
taken away by these tasks so application developers may rely on a common base
designing timing critical components (such as audio output or certain types of
graphics engines).

The constraints described here do not involve the cycle requirements of
processing particular kernel calls. They however include any other time spent
by the kernel, including time required to serve user initiated kernel tasks
(such as asynchronously loading a binary page).

The constraints:

- Up to 20% of the processing time may be taken of a second interpreted as a
  sliding window (so sampling the past one second any time no more than 20% of
  the processing time must have been taken by internal kernel tasks).

- Within a display frame at least 143680 cycles must be left for the user
  program. (This comes from taking at most 20% with a 70 Hz display which is
  the minimal frame time of 179600 cycles, as from 400 cycles per line with
  449 lines)

- Within any 48 display lines only up to 16 display lines worth of time may be
  consumed by the kernel for internal tasks (this normally means up to 6400
  cycles out of any 19200 cycles).

The second and third constraints may be relaxed if the particular
implementation realizes a slower display (such as 60Hz) by taking more time
from the (longer) vertical blanking.

The constraints in overall may be relaxed if the implementation is faster than
required by the specification as long as equivalent processing power is
guaranteed to be provided for the application in an appropriate granularity as
these constraints would enforce on a minimal performance implementation.

Note that due to these constraints an application can assume only at most 10
Mcycles worth of processing power per second.




Peripheral use
------------------------------------------------------------------------------


The kernel does not use the Peripheral bus and any Peripheral on it for any of
it's internal tasks.

Some video related kernel calls are exceptions, these are mentioned at the
appropriate calls in the "kcall.rst" documentation. These are allowed to
terminate the user application if they are called in an inappropriate
situation which the application should check for before calling.




Traps, detecting application faults
------------------------------------------------------------------------------


If the user program does something inappropriate, the result is that the host
terminates it with the assistance of the RRPGE kernel.

In the case of kernel calls this is simply the matter of checking the
parameters for validity, and only allowing the call to proceed and finish if
the parameters are right (there are however some kernel calls which can
purposefully return failure, indicated in their documentation).

Otherwise there are two sources of termination:

- Attempting to execute an instruction from the supervisor area (see the
  "Instruction Matrix" table in "cpu_inst.rst").

- Malformed stack accesses (addressing outside the allowed stack area, see
  "Stack Management" in "cpu_arch.rst").

These in real hardware are realized by trap mechanisms. The result is the same
like for inappropriately formatted kernel calls: the kernel terminates the
application.
