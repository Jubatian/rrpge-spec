
RRPGE application binary specification
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Overview of the application binary
------------------------------------------------------------------------------


The application binary contains the application's descriptor, the code, and
arbitrary binary data to be used by the application.

When stored as a file on a file system, it's extension should be ".rpa"
identifying it as an RRPGE application. The byte order within an application
file on byte-oriented systems must be Big Endian, so that text information
occurring it, parsable by the RRPGE CPU using it's 8bit sub-word addressing
mode, is also visible in normal plaintext viewers on the host system.

The binary begins with a text oriented header which describes the application,
and locates it's binary descriptor by which the application's properties, code
area, and initial data may be found. Apart from these, the contents of the
binary are arbitrary, and may be read using the 0x00: Start loading binary
data kernel call (described in "kcall.rst").




Application binary maps
------------------------------------------------------------------------------


The following table defines the layout of the application binary header.
Fields marked "F" are fixed contents, that is they must occur as defined
within the binary for identifying it as an RRPGE application. Fields marked
with "M" are mandatory, that is they must be filled with valid data according
to the constraints defined in the description. Fields marked with "O" are
optional, if enabled, they must be filled according to their description.
Other areas can have arbitrary content. Ranges are specified in 16 bit words.

The character identified "\n" is the Unix newline character of the binary
value 0x0A.

+--------+---+---------------------------------------------------------------+
| Range  | U | Description                                                   |
+========+===+===============================================================+
| 0x0000 |   |                                                               |
| \-     | F | "RPA\n" (File identification characters)                      |
| 0x0001 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0002 |   |                                                               |
| \-     | F | "\nAppAuth: "                                                 |
| 0x0006 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0007 |   | Application author identification. It should be a registered  |
| \-     | M | name (this is not mandatory). It is of exactly 16 characters  |
| 0x000E |   | from the following set: "[a-z][A-Z][0-9]- " (the last is      |
|        |   | 0x20: space). Hosts may check it for various reasons.         |
+--------+---+---------------------------------------------------------------+
| 0x000F |   |                                                               |
| \-     | F | "\nAppName: "                                                 |
| 0x0013 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0014 |   | Application name of 34 characters, padded with 0x20 spaces.   |
| \-     | M | It should be unique for an author. It must only contain 7 bit |
| 0x0024 |   | ASCII excluding control codes (anything below 0x20) and 0x7F. |
+--------+---+---------------------------------------------------------------+
| 0x0025 |   |                                                               |
| \-     | F | "\nVersion: "                                                 |
| 0x0029 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x002A |   | Application version in the form of "hh.mmm.ppp" where "h" is  |
| \-     | M | the major version, "m" is the minor, "p" is the patch. Only   |
| 0x002E |   | numerals ([0-9]) are allowed. Example: "01.14.019".           |
+--------+---+---------------------------------------------------------------+
| 0x002F |   |                                                               |
| \-     | F | "\nEngSpec: "                                                 |
| 0x0033 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0034 |   | Minimal RRPGE specification version the application conforms  |
| \-     | M | to. Uses the same format like the Application version.        |
| 0x0038 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0039 |   |                                                               |
| \-     | F | "\nDescOff: "                                                 |
| 0x003D |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x003E |   | Hexadecimal location of application descriptor. It specifies  |
| \-     | M | the start offset of the descriptor as a word offset within    |
| 0x003F |   | the first 64K words (must be within). Uppercase.              |
+--------+---+---------------------------------------------------------------+
| 0x0040 |   |                                                               |
| \-     | F | "\nLicense: "                                                 |
| 0x0044 |   |                                                               |
+--------+---+---------------------------------------------------------------+
| 0x0045 |   | License or licenses (if multiple alternative licenses are     |
| \-     | M | supported) the application may be used under. Multiple        |
| L.End  |   | license identifications must be separated by commas (","),    |
|        |   | spaces (" ") in-between are allowed. The end of the license   |
|        |   | list is marked by a newline ("\n"). It is not necessarily on  |
|        |   | a word boundary.                                              |
+--------+---+---------------------------------------------------------------+
| L.End  |   | Textual data. This may contain specific fields for various    |
| \-     | M | application types, and language specific information. It may  |
| T.End  |   | contain UTF-8 encoded characters. It is terminated by a       |
|        |   | single null (0x00) character.                                 |
+--------+---+---------------------------------------------------------------+

The first 64 words (0x0000 to 0x003F) also appear in state saves, used to
match the state save with the application binary.

At least one license must be provided. For the supported license
identifications, see the "Licenses" chapter below.

The textual data is although mandatory, may be left empty by providing a
single terminating null (0x00) character (not necessarily on word boundary).
Note that the terminating newline ("\n") of the license field must not be
omitted.

The application descriptor's start offset is provided by the DescOff field
of the header. Beginning at this offset it is mapped as follows:

+--------+---+---------------------------------------------------------------+
| Range  | U | Description                                                   |
+========+===+===============================================================+
| 0x0000 | M | File total size in words, high                                |
+--------+---+---------------------------------------------------------------+
| 0x0001 | M | File total size in words, low                                 |
+--------+---+---------------------------------------------------------------+
| 0x0002 | M | Word offset of code, high                                     |
+--------+---+---------------------------------------------------------------+
| 0x0003 | M | Word offset of code, low                                      |
+--------+---+---------------------------------------------------------------+
| 0x0004 | O | Word offset of data, high (ignored if no data)                |
+--------+---+---------------------------------------------------------------+
| 0x0005 | O | Word offset of data, low (ignored if no data)                 |
+--------+---+---------------------------------------------------------------+
| 0x0006 | M | Count of code words. 0: 64 KWords. Code is loaded beginning   |
|        |   | at address 0 in the Code space, and is started at address 0.  |
+--------+---+---------------------------------------------------------------+
| 0x0007 | M | Count of data words. 0: no data. Data is loaded beginning at  |
|        |   | address 0x0040 in the Data space.                             |
+--------+---+---------------------------------------------------------------+
|        |   | Count of stack words. 0: Use a separate 32 KWords stack       |
| 0x0008 | M | address space (no Data memory is taken for stack, but         |
|        |   | pointers into stack requested with the BP + immediate         |
|        |   | addressing mode can not be used to retrieve data in stack).   |
+--------+---+---------------------------------------------------------------+
|        |   | Start offset of stack. The stack must not overlap with the    |
| 0x0009 | O | User Peripheral Area, and can not exceed the data space. Only |
|        |   | used if Count of stack words is set nonzero.                  |
+--------+---+---------------------------------------------------------------+
|        |   | Input controller types the application may see. Each bit      |
| 0x000A | M | refers to one of the controller types (bit 0 corresponding to |
|        |   | controller type 0). See "inputdev.rst" for details.           |
+--------+---+---------------------------------------------------------------+
|        |   | Application flags.                                            |
| 0x000B | M |                                                               |
|        |   | - bit 14-15: Suggested data caching scheme                    |
|        |   | - bit    13: Has media total length if set                    |
|        |   | - bit    12: Has seek entry point and location data if set    |
|        |   | - bit    11: Has important audio output if set                |
|        |   | - bit    10: Has important video output if set                |
|        |   | - bit  8- 9: Icon resource type and availability              |
|        |   | - bit     7: Has alternate icon for inverted theme            |
|        |   | - bit  3- 6: Must be 0                                        |
|        |   | - bit  1- 2: Required File I/O support level                  |
|        |   | - bit     0: Requires network if set                          |
|        |   |                                                               |
|        |   | Icon resource type and availability:                          |
|        |   |                                                               |
|        |   | - 0: No icon (bit 7 also must be 0 this case)                 |
|        |   | - 1: 1 bit 64 x 64 icon (256 words)                           |
|        |   | - 2: 2 bit 64 x 64 icon (512 words)                           |
|        |   | - 3: 4 bit 64 x 64 icon (1024 words)                          |
|        |   |                                                               |
|        |   | File I/O support levels:                                      |
|        |   |                                                               |
|        |   | - 0: No File I/O support is required.                         |
|        |   | - 1: File I/O for "default.rbb" is required.                  |
|        |   | - 2: File I/O for ".rb*" and ".rf*" files is required.        |
|        |   | - 3: Generic File I/O is required.                            |
+--------+---+---------------------------------------------------------------+

The application descriptor from this point provides data for the optional
features, as many elements as many optional features in the Application flags
are enabled and require such. The following additional 16 bit words may be
included in the given order:

+-----------------+----------------------------------------------------------+
| Flag state      | Description                                              |
+=================+==========================================================+
| bit 13 set      | Media total length high 16 bits, in 187.5Hz ticks        |
+-----------------+----------------------------------------------------------+
| bit 13 set      | Media total length low 16 bits, in 187.5Hz ticks         |
+-----------------+----------------------------------------------------------+
| bit 12 set      | Seek function entry point in code space                  |
+-----------------+----------------------------------------------------------+
| bit 12 set      | Seek location data in data space                         |
+-----------------+----------------------------------------------------------+
| bit 8-9 nonzero | Word offset of icon, high                                |
+-----------------+----------------------------------------------------------+
| bit 8-9 nonzero | Word offset of icon, low                                 |
+-----------------+----------------------------------------------------------+
| bit 7 set       | Word offset of alternate icon, high                      |
+-----------------+----------------------------------------------------------+
| bit 7 set       | Word offset of alternate icon, low                       |
+-----------------+----------------------------------------------------------+

If either offset or the associated data length addresses out of the file's
total size, the application binary may be considered having an error, and
should be rejected.




Version information
------------------------------------------------------------------------------


There are two version information at 0x002A and 0x0034, one specifying the
application version, the other the specification's version the application
conforms to. The specification's version suggests the host whether it may or
may not load and run the application.

For the versions the following compatibility rules shall be followed:

- If major versions differ, it means complete incompatibility. The host
  implementing one major version of the specification should not attempt to
  load an application conforming to a different major version.

- Minor versions are upwards compatible. A host may load and run an
  application designed for a specification whose major version matches and the
  minor is less or equal.

- Patch versions are compatible either way.

- Exception: Versions of the specification having a major version of 0 may be
  incompatible with each other, and might be upwards compatible with major
  version 1. The major version number of 0 is intended to be used through the
  initial drafting process.




Licenses
------------------------------------------------------------------------------


The License field is meant to identify the license of the application using a
common acronym. The following acronyms are available:

- RRPGEvt: Temporary version of the RRPGE License.
- GPLv3: Version 3 of GNU General Public License.
- GPLv3+: Version 3 or any later version of GNU General Public License.
- GPLv2: Version 2 of GNU General Public License.
- GPLv2+: Version 2 or any later version of GNU General Public License.
- Other: ...: May be used for other licenses not having a defined acronym.

License compatibility chart: ::

    RRPGEvt ----> GPLv2+ -----> GPLv2
       |            |
       |            |
       |            V
       +--------> GPLv3+ -----> GPLv3

For example for the development of an application licensed under GPLv3, and
RRPGE Licensed component may be used.

Other acronyms may be added later.




Data caching schemes (bit 14 - 15 in Application flags)
------------------------------------------------------------------------------


Selecting an appropriate data caching scheme can improve loading times for an
application if it's binary is served over a slow connection (such as directly
from a network as streaming media).

The following schemes are available:

- 0: Random access. There is no suggested access pattern, only a generic
  caching algorithm may be used by the host.

- 1: Incremental access. The application normally will try to load areas
  incrementally from a starting point, while it may reload areas already
  loaded, and might access multiple locations incrementally at once.

- 2: Single streaming access. The application normally accesses areas
  sequentially, not reloading any area already used.

- 3: Multi streaming access. The application normally accesses it's areas
  sequentially, not reloading any area already used. However it accesses
  multiple such streams in it's data simultaneously (such as loading a
  separate audio stream along playing a primary stream).

Hosts aware of this feature should first load the application's descriptor and
the defined code and data areas, then access and pre-fetch data as suggested
by the caching scheme to achieve optimal performance.

If memory is low, and the application is streaming (either single or multi
streaming access) areas already used by the application may be discarded
favoring areas not yet loaded.




Media related properties (bit 10 - 13 in Application flags)
------------------------------------------------------------------------------


The media related properties suggests the application's usability by RRPGE
emulation capable media players in a sensible way.

If there is no seek entry point and data (bit 12 is clear) provided, but there
is a media total length (bit 13 is set) provided, it indicates the entire
application may be used as a playable media, which media may be treated as
audio or video according to the appropriate fields (bits 10 - 11). It may have
a playlist in addition, but this case it is only informative since there is no
way to seek onto the particular tracks.

If seek entry point and data is provided (bit 12 is set), players must use
this to start the media content. The normal entry point this case may boot
into an interactive application.

The seek entry point should be called like normal application reset, however
with the desired seek position (high word first) placed onto the stack, and
SP set to 2 indicating 2 parameters are on the stack.

The seek data is a 2 word location in the Data space of the application where
it should maintain a seek position (so reading it the host may know the
playback position).

Seek positions are expressed in 187.5Hz ticks.

If a playlist is provided, the playlist may provide whether particular tracks
may be used as audio only or they should be treated as audiovisual experiences
instead of the information provided in bits 10 - 11. The playlist is described
in the "Textual data" section. This case the media total length information
may be ignored (it might be present for hosts which do not support playlists).

The seek entry point not necessarily has to be 187.5Hz tick level accurate. It
should seek to or below the position requested. Media players so should not
assume a set position is absolute: they should read the seek data some
(emulated) time after (re)starting the application by this entry point.

From the application's point this is an entry point. The host should call it
by first resetting the application, then before starting the emulation,
setting up the program counter and the stack according to the requirements of
the seek entry point.




Icon resources
------------------------------------------------------------------------------

One or two 64 x 64 monochrome icon resources may be provided by the
application. In these resources, index zero should represent the background
color, and the highest index the foreground color (their actual value
depending on the host).

If two icon resources are provided, the first should be used if the user's
theme is dark foreground (text) over bright background, and the alternate if
it is bright foreground (text) over dark background. If there is no alternate
icon, always the primary icon should be used regardless of the theme.




Input related properties (0x0008 in the Application descriptor)
------------------------------------------------------------------------------


For more information on the supported input devices, and the overall
architecture of processing user input, see "inputdev.rst".

Note that these values do not require the host to actually have a given
hardware device, they only suggest that the application wishes to use one or
more devices in the role provided here. This way hosts may select the most
appropriate mapping to it's physical input capabilities.




Textual data
------------------------------------------------------------------------------


The area after the License field may contain UTF-8 text information describing
the application. Elements like supported languages, short application
description, extended application name, playlists and such may be provided
here in multiple languages.

All fields to be interpreted by the hosts begin with ":FieldName:" or
":FieldName [lang]:" on the beginning of a line. If the language designation
is omitted, the content is assumed to be multilingual, shown in case none of
the fields with language specification match the user's preferences. If there
is no such field, the user will not receive the given content in this case.

The fields end with an ":End:" marker on the beginning of a line.

Note that the field specifiers are all case-sensitive. Only the "\n" (0x0A)
new line character is recognized as a new line, the "\r" (0x0D) character
should not be used.


\:Language:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The languages provided by the application, separated with white characters
(spaces, tabs or newlines). The languages in this list should be identical to
those the application actually recognizes reading the user preferred language.

This field must not have a language designation.


\:AppName:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The application's name as shown to the user. This field may be used to
reformat the name to use UTF-8 characters, or to provide different names for
different languages (by adding a language designation to the field name).


\:AppAuth:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The application author as shown to the user. This field may be used to
reformat the name to use UTF-8 characters, or to provide different names for
different languages (by adding a language designation to the field name). Note
that hosts may ignore this field even if present if they choose to retrieve
the author's UTF-8 name from a network database.


\:HomePage:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A valid URL for more information on the application (home page). Different
homes may be provided for different languages by adding language designation.


\:Short:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Short application description, preferably up to about 300 characters.


\:Long:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Long application description.


\:PlayList:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Primary playlist information, specifying media type and lengths. Only one must
occur from this field with no language designation. To provide language
specific variants, use the ":PListExt:" field.

The format is as follows:

- "A:" or "V:" specifying if the entry has only important audio data or has
  both audio and video.

- Arbitrary UTF-8 entry name, whitespaces from the front and back of it are
  removed when processing.

- "{hh:mm:ss.ff}" specifying the length of the entry in hours, minutes,
  seconds and 1/100th seconds.

- "\n" new line ends the entry.

Empty lines in the playlist are allowed and are not processed.

The length information can be used to calculate the entry point (seek) of the
entry. They should be specified so calculating the entry in 187.5Hz ticks by
rounding down to nearest, passed to the seek entry point, would seek to the
proper track.


\:PListExt:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Extra playlist track names in additional languages. This field must have a
language designation (since ":PlayList:" already specifies the multilingual
interpretation).

Every non-empty text line in this field corresponds to a track in the playlist
whose name it replaces for the targeted language.
