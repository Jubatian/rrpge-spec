
RRPGE system specification and guides
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction
------------------------------------------------------------------------------


This is the specification of the RRPGE system and associated guides for
creating conforming hardware or software implementations and on the usage of
the system.

RRPGE originally stood for Retro Role Playing Game Engine, later reformed into
Retro Revolution Project Game Engine. The abbreviation should be pronounced as
"Rerpige", with the ending "ge" as in "gear".

The system roughly is a complete 16bit console with specialized CPU and
peripherals aiming for running all genres of retro-style games and other types
of software-art.

It's design goals are as follows:

- Overall system efficiency achieved by specialized hardware.
- Ease of use from assembler.
- Ease and efficiency of emulation.
- Good integration in the host when emulated.
- Possibility of realization in hardware.
- Completely open specification.

The main components of the system are as follows:

- 16bit CISC CPU with user / supervisor mode separation, memory protection,
  with a highly orthogonal instruction set. Notable features are small chunk
  (even bit) level memory accesses, no flags logic, and complex function call
  instructions. The CPU is provided with 1 MWords of data memory (half of
  which is also used for audio) of which 896 KWords are available to the user.

- 32bit Graphic Display & Accelerator unit capable of outputting 50-70Hz
  640x400 4bit or 320x400 8bit graphics. Notable features are up to four
  independent graphic layers and accelerators for various fast blit
  operations. The graphics hardware is accompanied with 512 KWords of Video
  memory.

- 8bit Stereo digital audio output with 24KHz or 48KHz sampling frequency.

- Mixer DMA peripheral capable to assist audio mixing algorithms.

- Various input sources including, but not limited to digital gamepads,
  pointing devices, touch devices, and joysticks.

- Additional features provided by a kernel, including network support.




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

The RRPGE Logo. Note that this logo is under trademark registration, rights
owned by Sandor Zsuga (Jubatian).
