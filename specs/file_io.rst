
RRPGE file I/O interface
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction, the basic file system interface concepts
------------------------------------------------------------------------------


The RRPGE kernel supports file I/O through related kernel calls (see "Kernel
functions, Storage & File management (0x0100 - 0x01FF)" in "kcall.rst" for
details). This document specifies the backend which should be provided behind
this functionality.

The host must provide an abstract file system interpretation to the RRPGE
Application with it's root being the area where the application should
naturally place it's files. A directory concept may or may not be supported.

UTF-8 file naming with extension based types is supported so the RRPGE
Application may interface with the host properly through common file formats
as needed. The host however may restrict the range of valid file names and
extensions for the purposes of safety.

Roughly the following levels of support may be provided by the host:


No support
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The host does not support file I/O at all. This case all four related kernel
calls return failure, indicating the feature is unsupported.


Single file (default.rbb)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The host provides a single file, presented to the application by the name
"default.rbb". The kernel calls should behave accordingly. The 0x0112 "Task:
Find next file" call may return "default.rbb" (if it exists) if called with an
empty string, and with an empty string in any other case.

Note that the physical file or binary data on the host need not be named
"default.rbb", it just needs to be provided for the RRPGE Application by this
name.


Flat filesystem
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The host provides an arbitrary amount of files with various restrictions on
the naming of those, but no directory concept. Physically the files may be
stored alongside the application, or in a directory defined by an environment
variable the host interprets.


Fixed directory support
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Directories are provided, but with the inability to create or remove these.
This type of support may be used if there are multiple, but fixed places where
an application may access files.


Full support
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The creating and the removal of directories is also supported alongside the
work with files.




File naming restrictions
------------------------------------------------------------------------------


The host may put restrictions on the file names for integrating into it's
file system and for safety concerns.

The file names if unrestricted should be interpreted as UTF-8. They can be
at most 256 bytes long including a terminating zero.

Simple hosts may restrict the character set of files to the following set:

- 'a' - 'z' (26 characters, codes 0x61 - 0x7A)
- '0' - '9' (codes 0x30 - 0x39)
- '_' (code 0x5F)

Moreover exactly one '.' (0x2E) is allowed to separate an extension from the
body of the file name. Restrictions in the length of file name components may
also be set, but these should allow at least 8 ASCII characters base and 3
ASCII characters extension.

When accessing the file system hosts may convert file names between upper and
lower case, and may silently accept file names of improper case. Other
conversions should not be attempted.

If directories are supported, the '/' (0x2F) character must be used as
directory separator (the host may convert this to a native separator if
necessary).




Extension restrictions
------------------------------------------------------------------------------


The host may restrict the application from loading or saving (writing) files
of particular extensions for the purposes of safety. It may apply different
restrictions on loading and saving if desired (more relaxed for loading).

For proper support minimally the following extensions should be allowed both
for loading and saving:

- 'rf?' where '?' is a character in the range [0-9][a-z]. This extension group
  is reserved for RRPGE managed "official" file formats.

- 'rb?' where '?' is a character in the range [0-9][a-z]. This extension group
  is reserved for application specific file formats.




File ordering and file searching
------------------------------------------------------------------------------


The exact ordering of files (for the purposes of kernel call 0x0112 "Task:
Find next file") is implementation defined, however some key characteristics
of it are specified so it can be useful:

- The ordering must be deterministic for the host.

- The character code 0x00 (terminator) at a position is always the first on
  that position.

- The character code 0xFF at a position is always the last on that position.
  This code is also invalid in file names, so a valid file never matches it.

- Searching always includes the entire tree, so traverses directories and
  returns files by their full path (if directories are supported).

These properties allow for useful searches, such as skipping a group of files
(for example ones within a directory) by appropriately formatting a next
search start point using the 0x00 or 0xFF codes.




Directories
------------------------------------------------------------------------------


If supported, directories need to operate by the following principles:

- They are empty (zero content) files.

- Their file name is always formed with a '/' as it's last character.
  Referring a directory without the '/' refers to a different normal file
  (usually invalid on most hosts).

- When creating a directory with a file save, the source data size must be
  0 bytes. Otherwise the save must fail.

- Deleting a directory must fail if there are files within.

- Directories can only be created explicitly. If the directories in a file
  name do not exist (except for the last if the name refers to that
  directory), the file name is invalid.




Error codes
------------------------------------------------------------------------------


The File I/O related kernel calls may return error codes on the termination of
their tasks. These are as follows:


+--------+-------------------------------------------------------------------+
|  Code  | Description                                                       |
+========+===================================================================+
| 0x0000 | Unsupported feature                                               |
+--------+-------------------------------------------------------------------+
| 0x0001 | Improper / Invalid file name                                      |
+--------+-------------------------------------------------------------------+
| 0x0002 | Source file does not exist                                        |
+--------+-------------------------------------------------------------------+
| 0x0003 | Target file exists (for file moves)                               |
+--------+-------------------------------------------------------------------+
| 0x0004 | Target file exists, but is not writable (for example it is a      |
|        | directory, or has no write permissions)                           |
+--------+-------------------------------------------------------------------+
| 0x0005 | Source file exists, but can not be renamed or deleted (for        |
|        | example it is a non-empty directory selected for removal)         |
+--------+-------------------------------------------------------------------+
| 0x0006 | Can not create the file (for moves and saves; also if attempting  |
|        | to create a directory with nonzero data)                          |
+--------+-------------------------------------------------------------------+
|        | Can not write more data into the file (for example if the storage |
| 0x0007 | is full, a quota for the application is exceed, or attempting to  |
|        | fill a file with gaps)                                            |
+--------+-------------------------------------------------------------------+
