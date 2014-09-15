
RRPGE state save specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Overview of the state save binary
------------------------------------------------------------------------------


The state save binary contains the full state of the RRPGE system necessary to
restore the application to the point when the state save was archived. Note
that the state save does not contain the application binary, so both the state
save and the application binary (of exactly matching versions) is necessary to
be able restore.

The total size of the state save is 2145 KWords or 4290 KBytes.




State save map
------------------------------------------------------------------------------


The state save simply combines the Application Header and the necessary RRPGE
system memories to completely cover the system's state from the user's point
of view (with the limitations described in "state.rst"). Following the map of
areas as they occur in the state save binary are listed (the addresses are in
16 bit word units).

+----------+-----------------------------------------------------------------+
| Range    | Description                                                     |
+==========+=================================================================+
| 0x000000 | Application header, the "RPA" head replaced to "RPS", otherwise |
| \-       | left as-is. This is used to identify the matching application   |
| 0x00003F | binary.                                                         |
+----------+-----------------------------------------------------------------+
| 0x000040 |                                                                 |
| \-       | Application state. See "state.rst" for details.                 |
| 0x0003FF |                                                                 |
+----------+-----------------------------------------------------------------+
| 0x000400 |                                                                 |
| \-       | Stack memory (32 KWords).                                       |
| 0x0083FF |                                                                 |
+----------+-----------------------------------------------------------------+
| 0x008400 |                                                                 |
| \-       | CPU RAM data (64 KWords, first 64 words unused).                |
| 0x0183FF |                                                                 |
+----------+-----------------------------------------------------------------+
| 0x018400 |                                                                 |
| \-       | Peripheral RAM data (1 MCells or 2 MWords).                     |
| 0x2183FF |                                                                 |
+----------+-----------------------------------------------------------------+
