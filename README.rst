
RRPGE system specification and guides
==============================================================================

.. image:: https://cdn.rawgit.com/Jubatian/rrpge-spec/00.013.002/logo_txt.svg
   :align: center
   :width: 100%

:Author:    Sandor Zsuga (Jubatian)
:License:   2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
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


Related projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- RRPGE home: http://www.rrpge.org
- RRPGE Assembler: https://www.github.com/Jubatian/rrpge-asm
- RRPGE Emulator & Library: https://www.github.com/Jubatian/rrpge-libminimal
- RRPGE User Library: https://www.github.com/Jubatian/rrpge-userlib
- Example programs: https://www.github.com/Jubatian/rrpge-examples


Temporary license notes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently the project is developed under a temporary GPL compatible license.
The intention for later is to add some permissive exceptions to this license,
allowing for creating derivative works (most importantly, applications) under
other licenses than GPL.

For more information, see http://www.rrpge.org/community/index.php?topic=30.0




Design goals
-----------------------------------------------------------------------------


The primary goals of this system are roughly as follows:

- Long term binary stability (once the first stable version is produced).
- Capability to integrate seamlessly in host systems (emulation).
- Simple design allowing for realizations by as little resources as possible.
- Realizing an early 90's era microcomputer or game console.

Below each of these four goals are described, what is aimed for exactly, why,
along with some related design notes.


Long term binary stability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nowadays when information technology advances with a frightening pace, it gets
more and more troublesome for lone developers to release something, and then,
to maintain it to keep it being usable. Especially with retro style games,
supposed to be possible with decades old hardware, this is getting somewhat
ridiculous: If you knew you hadn't got the resources to actively maintain it,
you would have to pick an existing old system (such as a Commodore 64 or IBM
PC with DOS) as target, so, thanks to emulation, your game won't just bit-rot
away in a few years.

RRPGE in this term aims to provide a system similar to those old ones which
are emulated today: with a stable specification, it aims to achieve long-term
binary stability, so games and other software produced for it keep being
usable without the burden of maintenance on the part of the developer. Just
like any other old system with decent emulators today.


Capability to integrate seamlessly in hosts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Designing a new system it is possible to settle for an emulation-centric
viewpoint: aiming to design it such a manner it can integrate without the
need of awkward solutions at the user. For example the need for virtual disks,
complex system configurations, inadequate controller setups can be avoided
fairly well.

From the developer's point this is advantageous since an easy to use system
(at the user's end) is more likely to be adapted.

While it is true that it is possible to hide complex configuration in
pre-packaged installers, that solution is far from ideal, and demands the
developer's attention to design and maintain those. With RRPGE, the problem
is shifted to the system's design, lifting this problem from the developer's
shoulders, and allowing for a much cleaner system on the user's end.


Simple design for little resources
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RRPGE is designed in a manner anticipating scarce resources in both host
systems and integrators (who would adapt RRPGE to a particular host).

The microcomputer is constructed in such a manner it can be largely
self-contained in a completely freestanding library (under development), with
as little interface to the host as reasonably possible, provided in a manner
that it can be developed incrementally.

In addition to this, several features of the system is rather contained in a
so-called User Library, which is a software package designed for the (virtual)
hardware of RRPGE. This approach reduces the implementation needs in case the
RRPGE hardware has to be reproduced.

The RRPGE CPU from the user's point of view is a Harvard architecture, with an
instruction set which is rather trivial to disassemble: this, if such approach
is necessary, allows for simple recompilation, increasing performance.
Otherwise the CPU's characteristics is chosen so it can be emulated fairly
well even with simple cross-platform interpreting.


An early 90's microcomputer or game console
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The target era for RRPGE is chosen to produce a fairly capable system which
still represents an age when lone or small group software development was
still common and viable.

The 80's is quite well covered with well-known and properly emulated systems
such as the Commodore 64, developing an another 8 bit machine would have been
rather pointless. The 90's with higher end 2D, larger RAM allowing for more
complex games and software however is rather unfriendly from today's
perspective. Systems in this era (both computers and consoles) are already
quite complex, largely with no open specifications, and mostly even with legal
constraints (such as the case of Kickstart ROMs for Amigas). Practically
probably the only "clean" system to have from this era is the IBM PC with all
it's known problems.

RRPGE aims to fill in this gap by providing a system with a definite, simpler
specification than several of those from this era, with clear licensing
status.

From microcomputer perspective RRPGE might be at about as capable as a 12 MHz
80286 by it's CPU, however it provides graphics and audio hardware uncommon
for the IBM PC, which can be utilized to produce complex visuals and sound
which wasn't possible until much later with the PC.

From console perspective RRPGE may be somewhere between the 4th and 5th
generations. It provides rich 2D features similar to the capabilities of 4th
generation consoles, while allowing for venturing in the world of 3D as well.




Main components
------------------------------------------------------------------------------


Below a short summary of the main RRPGE components and their features is
provided. For more extensive details, check "specs/overview.rst".

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

- 16bit Stereo digital audio output with up to 48KHz sampling frequency on the
  Peripheral bus, also providing a 187.5Hz clock.

- FIFO assisted Mixer DMA peripheral capable to assist audio mixing
  algorithms.

- Various input sources including, but not limited to digital gamepads,
  pointing devices, touch devices, and joysticks.

- Additional features provided by a kernel, including network support.

- Additional convenience routines and features provided by an User Library
  such as a sprite system and text output facilities.




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


File "CHANGELOG"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Timeline of versions and the changes introduced in them.


File "LICENSE.RRPGEvt"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE License.


File "LICENSE.GPLv3"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A copy of version 3 of the GNU General Public License license text
(http://www.gnu.org/licenses/gpl-3.0.html).


File "TRADEMRK"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

List of trademarks incorporated within this project, as required by the RRPGE
License.


File "VERSION"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The version number of the RRPGE specification.


File "README.rst"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This file.


File "logo.svg"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE Logo. Note that this logo is a registered European trademark, rights
owned by Sandor Zsuga (Jubatian).


File "logo_txt.svg"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The RRPGE Logo with "Retro Revolution Project Game Engine" text, used in the
title. Note that this logo is a registered European trademark, rights owned by
Sandor Zsuga (Jubatian).
