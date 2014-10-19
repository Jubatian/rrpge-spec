
RRPGE system specification and guides
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


This is the specification of the RRPGE system and associated guides for
creating conforming hardware or software implementations and on the usage of
the system.

RRPGE originally stood for Retro Role Playing Game Engine, later reformed into
Retro Revolution Project Game Engine. The abbreviation should be pronounced as
"Rerpige", with the ending "ge" as in "gear".

The system roughly is a complete 16bit microcomputer with specialized CPU and
peripherals aiming for running all genres of retro-style games and other types
of software-art.


Temporary license notes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently the project is developed under a temporary GPL compatible license.
The intention for later is to add some permissive exceptions to this license,
allowing for creating derivative works (most importantly, applications) under
other licenses than GPL.

For more information, see http://www.rrpge.org/community/index.php?topic=30.0


Design goals
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In summary meeting the following goals are aimed for when designing this
system:

- Overall system efficiency achieved by specialized hardware.
- Relative ease of use from assembler.
- Ease and efficiency of emulation.
- Good integration in the host when emulated.
- Possibility of (preferably not too complex) realization in hardware.
- Completely open specification.

As always, there are necessary compromises. For example the design of the CPU
is heavily constrained by three of the goals: be easy to program in assembler,
be efficient when emulated, and be possible to be realized as real hardware.
The first two of these goals determined it to be CISC processor.


Main components
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 16bit CISC CPU with user / supervisor mode separation, memory protection,
  with a highly orthogonal instruction set. Notable features are small chunk
  (even bit) level memory accesses, no flags logic, and complex function call
  instructions. The CPU is provided with CPU Data memory of which 64 KWords
  are available to the user as Data, 32 KWords as Stack, while another 64
  KWords contain the application code.

- 32bit Peripheral system on a separate bus, operating in parallel with the
  CPU. 1M * 32bits of Peripheral RAM is provided on this bus, accessible by
  the CPU through a streaming interface.

- Graphic Display Generator & Accelerator unit on the Peripheral bus capable
  of outputting 50-70Hz 640x400 4bit or 320x400 8bit graphics. Notable
  features are display list based line composing, and a FIFO assisted
  Accelerator capable to perform many types of fast blit operations.

- 8bit Stereo digital audio output with up to 48KHz sampling frequency on the
  Peripheral bus.

- FIFO assisted Mixer DMA peripheral capable to assist audio mixing
  algorithms.

- Various input sources including, but not limited to digital gamepads,
  pointing devices, touch devices, and joysticks.

- Additional features provided by a kernel, including network support.


Comparison with computers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE system aims to reproduce something which might have been possible in
the early 90s. In this era already the IBM PC dominated, still running mostly
DOS in 16 bits, gradually pushing Commodore's Amiga behind. By design RRPGE is
closer to an Amiga than an IBM PC with it's extensive graphics system (the IBM
PC mostly only had VGA with very little acceleration used), however it has a
16 bit CPU. Access to larger amounts of RAM than directly adressable by this
processor is provided through an unique streaming interface, mostly resembling
to a concept utilized by a rare type of REU for the Commodore 64.

By CPU power and the main clock frequency of 12.5MHz RRPGE mostly falls behind
the microcomputers of the era, however it provides a graphics subsystem and
audio acceleration features usually not available on those.


Comparison with consoles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By design RRPGE may be considered a 4th generation game console. The system
design is simpler than most consoles, and a notable difference is a rather
large RAM memory compared to those, while no fast direct access is provided
for application binary data (unlike cartridges).




Structure of the project
------------------------------------------------------------------------------


Below the contents of each of the directories and files found in the project
root are summarized.

Note that in every directory there is an "index.rst" file which serves as a
starting point for that part of the project.


Directory "specs"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The definition of the RRPGE system as far as necessary for understanding it at
user level and for writing software emulators. This area only covers the
system itself, and not any additional services related to the system (such as
suggested network interfaces and protocols).


Directory "impl_hw"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additional details for assisting developing hardware RRPGE systems. Most
particularly it includes suggested timing breakdowns and access patterns for
CPU opcodes and the graphics accelerator, and some additional details not
necessary for software implementations (such as the supervisor mode of the
CPU).


Directory "impl_sw"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additional details for assisting developing software RRPGE systems
(emulators). This includes a suggested emulator architecture splitting it in
a platform independent emulator library and a host application. Also included
are low-level techniques for efficiently realizing particular features of the
RRPGE system.


Directory "emu_lib"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Platform independent emulation library interface definitions including
programming language specific headers.


Directory "emu_host"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Guidelines for implementing software emulator hosts, that is applications
which use the emulation library or libraries to present the user an RRPGE
system.


Directory "services"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Additional RRPGE related service specifications mainly including network
protocols to be implemented by emulator hosts by which hosts and servers may
communicate in a standardized way. These components are only suggestions as an
RRPGE system by itself is functional without the implementation of any of
these.


File "LICENSE.RRPGEv1"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE License.


File "LICENSE.GPLv3"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A copy of version 3 of the GNU General Public License license text
(http://www.gnu.org/licenses/gpl-3.0.html).


File "LICENSE.LGPLv3"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A copy of version 3 of the GNU Lesser General Public License license text
(http://www.gnu.org/licenses/lgpl-3.0.html).


File "TRADEMRK"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

List of trademarks incorporated within this project, as required by the RRPGE
License.


File "VERSION"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The version number of the RRPGE specification.


File "index.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file.


File "logo.png"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE Logo. Note that this logo is a registered European trademark, rights
owned by Sandor Zsuga (Jubatian).
