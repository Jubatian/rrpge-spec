
RRPGE state save specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Overview of the state save binary
------------------------------------------------------------------------------


The state save binary contains the full state of the RRPGE system necessary to
restore the application to the point when the state save was archived. Note
that the state save does not contain the application binary, so both the state
save and the application binary (of exactly matching versions) is necessary to
be able restore.

The state save's most important component is the ROPD dump (see "ropddump.rst"
for details) which contains the Application Header with the necessary
identification and version information so the state can be matched to the
application along with the internal state of the RRPGE system.

The total size of the state save is 361 pages or 2.82 Mbytes (2957312 bytes).




State save map
------------------------------------------------------------------------------


The state save simply combines the necessary RRPGE system pages to completely
cover the system's state from the user's point of view (with the limitations
described in "ropddump.rst"). Following the map of pages as they occur in the
state save binary are listed.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+--------+-------------------------------------------------------------------+
| 0x0000 | ROPD dump as defined in "ropddump.rst".                           |
+--------+-------------------------------------------------------------------+
| 0x0001 |                                                                   |
|   -    | Stack memory pages.                                               |
| 0x0008 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0009 |                                                                   |
|   -    | Data memory pages.                                                |
| 0x00E8 |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x00E9 |                                                                   |
|   -    | Video memory pages.                                               |
| 0x0168 |                                                                   |
+--------+-------------------------------------------------------------------+
