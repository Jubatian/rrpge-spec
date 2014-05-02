
RRPGE nonvolatile save binary specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Overview of the nonvolatile save binary
------------------------------------------------------------------------------


The nonvolatile save binary is a binary coupled to the application, used for
the purpose of saving user made content from the application without user
noticeable safety barriers.

It has a 48 word header, including and following that an arbitrary number of
4096 word pages follow with arbitrary application specific binary data.




Nonvolatile save header
------------------------------------------------------------------------------


The nonvolatile save header is a copy of the first 48 words of the Application
Header (see "bin_rpa.rst") with a single byte change in the first, file
identification characters. The first bytes of the nonvolatile save so read as
"RPN\n".

Note that word 0x02F, the last word copied from the Application Header reads
as "\nE" (has an 'E' on the low byte coming from the "EngSpec" field of the
Application Header). This also must be replicated as-is.

For the loading and saving of Nonvolatile saves check the appropriate
functions of the kernel in "kcall.rst" (functions 0x0110 "Task: Start loading
nonvolatile save" and 0x0111 "Task: Start saving nonvolatile save").

For version compatibility (which should be enforced by the kernel or host)
across saves and applications check the "Version information" chapter of
"bin_rpa.rst".
