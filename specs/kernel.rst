
RRPGE kernel overall specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




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

The other feature exposed to the user is the provision of two event handlers,
detailed later below. These are called upon specific interrupts.

On a real hardware implementation the kernel also has several internal tasks
to perform in order to be able to support it's interface, timing constraints
are defined for these to make it possible for them to conform with this
specification.




Supported events
------------------------------------------------------------------------------


In the RRPGE CPU interrupts are served by supervisor code (see "Interrupts" in
"cpu_arch.rst"). This supervisor code (the kernel) after dealing with any
necessary internal aspects, can call back user code to act on these events.

There are two such interrupts for which user events may be provided: The audio
and the video raster interrupts. These are described below. The audio
interrupt has higher priority than the video raster interrupt, so the video
event handler might be interrupted by an audio handler.

The overhead imposed by the kernel for both event types is up to 400 cycles
total. This overhead only exist if the particular event is enabled and
triggers. It is not defined how large portion of this overhead takes part
before or after the execution of the user event handler.

User event handlers might also be interrupted by the kernel as long as these
are kept within the constraints for internal kernel tasks.

While one type of interrupt or related user event is processed, even one of
the same type may queue, triggering as soon as the currently executing handler
returns. Only one interrupt may queue this way, any further events are
discarded.

Kernel calls and internal kernel tasks are not necessarily interruptible, so
they may also delay the execution of an event handler.


Audio half-buffer exhausted
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This interrupt triggers, so the event is called when the audio output DMA
passes half buffer boundary. The event has two parameters prepared by the
kernel:

- Left or Mono target sample pointer in sample (byte) units
- Right target sample pointer in sample (byte) units

For the possible values of these, check "snd_arch.rst", the "DMA buffers"
chapter. The parameters provide pointers to the buffer half just finished by
the audio DMA, so from the rise of the event there is a whole audio tick for
the call to happen and to fill these in.

The right target sample pointer equals the left in Mono mode.

The audio events come at a fixed rate, called the audio tick. This rate is
consistent across any realization of the RRPGE system, only depending on the
configuration of the application. The following rates are possible:

- 23.4375 Hz (24 KHz sample rate, 1024 sample half buffer).
- 46.8750 Hz (48 KHz / 1024 samples or 24KHz / 512 samples half buffer).
- 93.7500 Hz (48 KHz sample rate, 512 sample half buffer).

This should be used as the time base of the application.


Video raster passed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This interrupt triggers, so the event is called when the video display enters
a set raster line. It has no parameters.

Note that as the "vid_arch.rst" part states, the display's update frequency
may range from 50Hz to 70Hz (or probably even more) arbitrarily, so the rate
of this event is not fixed across implementations. The frequency may even
change during the execution of the application such as if it's state was saved
and restored on a different host.

Moreover it is not possible to specify lines within the vertical blanking
period; it is only possible to select the beginning of the vertical blank area
to generate an event at.





Kernel timing constraints
------------------------------------------------------------------------------


The kernel on a real hardware implementation may require CPU time for
performing internal tasks (such as managing network packets or communication
with input peripherals). It is important to constrain the processing time
taken away by these tasks so application developers may rely on a common base
designing timing critical components (such as audio output or certain types of
graphics engines).

The constraints described here do not involve the cycle requirements of
processing particular kernel calls, neither the (up to 400 cycle) overhead of
interrupt entries and returns described above. They however include any other
time spent by the kernel, including time required to serve user initiated
kernel tasks (such as asynchronously loading a binary page).

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
from the (longer) vertical blanking. This relaxing is only allowed as long as
it is guaranteed to not affect any audio code assuming conformance to these
constraints (that is with the audio interrupt triggering at any time the
event handler must be guaranteed to have at least the time a strictly
conforming kernel would provide within an audio tick).

The constraints in overall may be relaxed if the implementation is faster than
required by the specification as long as equivalent processing power is
guaranteed to be provided for the application in an appropriate granularity as
these constraints would enforce on a minimal performance implementation.

Note that due to these constraints an application can assume only at most 10
Mcycles worth of processing power per second.




Video peripheral use
------------------------------------------------------------------------------


The kernel does not use the video peripheral for any of it's internal tasks,
so it does not stall on user initiated video operations.

Some video related kernel calls are exceptions, these are mentioned at the
appropriate calls in the "kcall.rst" documentation.




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

