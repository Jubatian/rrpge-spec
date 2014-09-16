
RRPGE system specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction to the System
------------------------------------------------------------------------------


The RRPGE system is composed of the following fundamental functional
components:

- 16bit CISC CPU for running the application and kernel code. It is coupled
  with CPU RAM from which the user is served with 64 KWords of Data memory and
  8 KWords of stack in addition to the Code memory.

- 32bit asynchronous Peripheral system coupled with 2 MWords (1M x 32bit) of
  Peripheral RAM. Graphics and Audio operates in this domain.

- Kernel providing interface for certain hardware (or in emulated case, host)
  features, particularly the input system.

- Host system managing the kernel provided hardware (such as input devices and
  networking), and the file system (if any).


Overall system architecture
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the "overview.rst" part for a block diagram displaying the connections
of the user available hardware components. The "mem_map.rst" document provides
the overall memory map of the system.


Timing details
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An RRPGE application is realized mostly as code to be executed on the CPU,
accompanying program data, and some support information. Alongside the
specifications of the operations possible by the system (CPU instructions,
peripheral and kernel functions) it is also necessary to provide performance
related information by which the applications may profile themselves.

The RRPGE system specification solves this problem by providing minimal timing
requirements for each of the performance related aspects of the system. This
means that it defines the maximal time particular operations may take to
perform, and where necessary, minimal time for certain states the system might
be in.

An implementation of the system must meet these maximum and minimum limits,
however beyond that it may provide arbitrary timing still conforming with this
specification (simplified this means an implementation may realize a faster
system than specified).


Clocking requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The system is clocked as required by the display generation, typically
(assuming VGA or NTSC output) by a 25.175MHz base frequency. Most of the
components including the memories are clocked with half of this frequency.

Implementations must so meet the followings:

- At least 25MHz base clock for video display output (required for VGA
  compatible non-interlaced 640 pixel wide modes).

- At least 12.5MHz main clock for running the system's components. Operation
  timing requirements further are indicated in units by this clock, however
  those times should be treated in real time units (so if an implementation
  decides to use a faster main clock, it may choose to run operations in more
  clocks than required by the specification as long as by real time the
  requirement is met).


The host system
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The behavior of the host system, that is how the system behaves outside the
scope of running the RRPGE application is not part of this specification.

The basic features of the host system would be for example mapping the native
input devices to those supported by the RRPGE kernel, providing some sort of
networking to serve the related kernel features, and giving some file system
abstraction for the user to manage RRPGE applications and associated data.

For more information regarding the host, check the "/services" directory of
the specification which provides suggestions for this matter.




Description of component documents
------------------------------------------------------------------------------


File "acc_arch.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The architecture of the Graphics Accelerator providing blit, fill and line
drawing functions.


File "bin_rpa.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Application Binary format defining the application and it's data.


File "bin_rps.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The State Save binary format defining the format of application state binaries
which may be used to take snapshots of the application for continuing later.


File "boot.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bootup (reset) state of the RRPGE System as seen by the application.


File "cpu_arch.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The architecture of the 16 bit CISC CPU. Details like processor registers,
memory management, stack management, and user - supervisor mode separation.


File "cpu_inst.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Processor instruction set including bytecodes, suggested opcode mnemonics,
timing information, and detailed behaviors of instructions.


File "data.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Constant data provided for the application, and the layout of those.


File "file_io.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usage of the file I/O interface provided by the kernel.


File "fifo.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The operation of FIFO's which run the Graphics Accelerator and the Mixer DMA.


File "index.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file.


File "inputdev.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Input device related information. It should be read alongside with the input
device related sections of "kcall.rst" and "bin_rpa.rst".


File "kcall.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

List and details of each kernel call and related information.


File "kernel.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The description of the kernel, particularly the interrupt (event) system and
timing details related to the kernel.


File "mem_map.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Memory map of the complete system as seen by the user. The "Address spaces and
Memory management unit" of "cpu_arch.rst" should be read before this.


File "mix_arch.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The architecture of the Audio Mixer (Mixer DMA) used for accelerated digital
audio sample mixing.


File "names.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Conventions for the interpretation of User ID values.


File "overview.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The overall hardware architecture of the system as seen by the user.


File "pointer.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Peripheral RAM interface description, which provides pointers for accessing
the Peripheral RAM in a sequential manner.


File "snd_arch.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The sound system of RRPGE.


File "state.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The structure and the mapping of the Application State containing necessary
internal state information to make state saves possible.


File "vid_arch.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Graphics Display Generator component of RRPGE and the architecture of
display generation.
