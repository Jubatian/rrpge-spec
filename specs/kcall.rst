
RRPGE kernel calls
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, the role and layout of the kernel calls
------------------------------------------------------------------------------


The kernel calls are the main interface by which an RRPGE application can
communicate with the host system, or perform certain more or less host
specific actions. Moreover kernel calls are in place to support the tight
user mode - supervisor mode separation of the RRPGE CPU, such as to provide
means for switching in and out memory banks from the CPU's address space.

Other important aspects of the kernel are detailed in the "kernel.rst"
documentation.

The kernel call interface is implemented using the "JSV" opcode of the RRPGE
CPU (see "JSV" in "cpu_inst.rst"). The first parameter of the call selects the
kernel function to execute while further parameters may be passed to the
function itself. Note that there is no kernel function which would accept a
variable amount of parameters, so all functions have to be called with the
appropriate count of parameters as specified in their descriptions.

The layout of the kernel functions (by the value of the first parameter
selecting it) are as follows:

- 0x0000-0x00FF: Memory management (empty)
- 0x0100-0x01FF: Storage & File management
- 0x0200-0x02FF: Audio (empty)
- 0x0300-0x03FF: Video
- 0x0400-0x04FF: Input devices
- 0x0500-0x05FF: Delay
- 0x0600-0x06FF: User information management
- 0x0700-0x07FF: Networking
- 0x0800-0x08FF: Kernel task management


Kernel tasks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kernel tasks are asynchronous processes which may be initiated by certain
kernel functions. After starting they progress asynchronously to the
application, using only the time allowed to be taken by the kernel (see
"Kernel timing constraints" in "kernel.rst"). They have no set deadline, and
they neither can be terminated by the application. The application may poll
their status to see whether they finished and what their results were.

Advanced hosts may use this feature to provide means for example for smooth
loading of data either from disk or network depending on how well the
application supports waiting for the termination of the initiated tasks.

At least up to 16 tasks may be started simultaneously. If the application
needs more, it has to discard previous completed tasks by the "Discard task"
function (0x0801) provided in the Kernel task management group.

Kernel tasks typically operate from or into CPU Data memory areas. If the
memory area is a target, it's state is undefined until the completion of the
task. If the memory area is a source and it is modified before the completion
of the task, the result of the task regarding the changed contents, unless
otherwise specified, is undefined.


Unsupported functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When calling an unsupported function (that is the first parameter is some
value not listed in this specification) the kernel is normally required to
terminate the application.

Likewise when calling a supported function with a wrong number of parameters,
the kernel is required to terminate the application.


Invalid parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the parameters passed to a kernel function are invalid in some manner (such
as attempting to access application data above the application binary size)
the kernel typically terminates the application. Exceptions from this rule are
indicated for the appropriate functions.


Notes on extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If necessary for implementing a given host, or to provide extra features for
some specific target applications, the kernel may be expanded with additional
functions. The preferred range for this is 0x8000 or above.

Relying on this extension normally can't break any conforming application as
they would only terminate calling any unspecified functions. However the
feature should be used sparingly and cautiously as a critical function such as
one capable to write arbitrary files on a given host system may open a
back-door for viruses on that particular host.

Any application using a kernel function not listed in this specification must
be regarded nonconforming. These types of applications should only be built
for two specific purposes:

- To provide an user interface on a specific RRPGE host using native RRPGE
  applications, such as supportive parts of an operating system of a hardware
  implementation. The preferred range for this is 0x8000 - 0x8FFF.

- To experiment with new functionality which may later be included in the
  main RRPGE specification. The preferred range for this is 0x9000 or above.

For the first type the recommended style of implementation is to hide all the
extended functionality when running external RRPGE applications (that is
applications not serving as a part of the operating system or it's interface
requiring the extended functions).


Function names (F.name)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The function names are suggested identifiers for the kernel functions in case
the build system would want to support such more developer-friendly labels.


Cycle counts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The cycle counts provided for each kernel function include the complete
overhead of the function including the cycle requirements of executing "JSV",
but excluding traversing the parameter list after returning from the call.


Host requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is indicated whether implementation on the host's side is required to
support a given functionality, or it can normally be contained within a
freestanding RRPGE library.


Default return (N/S)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the host is required to support a given functionality it might choose to
not support it in a sensible way. This case by producing an appropriate
default reaction applications requiring the functionality may still operate in
a limited mean (for example not allowing to load and save user files if the
host lacks the implementation of those).


Function return values
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Kernel functions either has no return, return in the register "X3", or in
"C:X3" depending on the amount of direct return information they produce. No
other registers are affected. If the kernel function requires to produce more
return data, it takes pointers to memory areas it should fill.




Kernel functions, Memory management (0x0000 - 0x00FF)
------------------------------------------------------------------------------


(No kernel function in this group)




Kernel functions, Storage & File management (0x0100 - 0x01FF)
------------------------------------------------------------------------------


0x0100: Task: Start loading binary data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_loadbin
- Cycles: 800
- Host:   Required.
- N/S:    This function must be supported.
- Param1: Target offset in CPU Data memory.
- Param2: Number of words to load.
- Param3: Source application binary offset high word.
- Param4: Source application binary offset low word.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Loads an area of the Application binary into the CPU Data memory. The kernel
terminates the application if either parameter is invalid:

- The target must not be the User Peripheral Area, it must neither wrap around
  to it, and must not have zero size.

- The source must be within the Application binary entirely.

The task always returns 0x8000 on completion.


0x0110: Task: Start loading from file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_load
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful load.
- Param1: Target (word) offset in CPU Data memory.
- Param2: Number of bytes (!) to load, up to 16383.
- Param3: Byte offset to start loading from the file, high word.
- Param4: Byte offset to start loading from the file, low word.
- Param5: File name offset in CPU Data memory.
- Param6: File name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Loads bytes from a file. The bytes are loaded in Big Endian order (so first
loaded byte of the file will be the high byte of the first word of the
target).

The file name is excepted to be a zero terminated UTF-8 string.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task has bit 14 clear if the load was successful,
bits 0 - 13 indicating the number of bytes successfully loaded (0 - 16383).
This may be less than the requested number of bytes (maybe even zero) if the
file was too small. Bit 14 set in the return value indicates failure, bits
0 - 13 providing a fault code.

See "file_io.rst" for further details including fault codes.


0x0111: Task: Start saving into file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_save
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful save.
- Param1: Source (word) offset in CPU Data memory.
- Param2: Number of bytes (!) to save, up to 16383.
- Param3: Byte offset to start at in the file, high word.
- Param4: Byte offset to start at the file, low word.
- Param5: File name offset in CPU Data memory.
- Param6: File name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Saves bytes into the target file. The bytes are saved in Big Endian order (so
first saved byte of the file will be from the high byte of the first word in
the source area).

Note that the host should fail if the file is not sufficiently large already
so the new data can be added without gaps.

The file name is excepted to be a zero terminated UTF-8 string.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task has bit 14 clear if the save was successful,
bits 0 - 13 indicating the number of bytes successfully saved (0 - 16383).
This equals to the requested number of bytes to save. Bit 14 set in the return
value indicates failure, bits 0 - 13 providing a fault code.

See "file_io.rst" for further details including fault codes.


0x0112: Task: Find next file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_next
- Cycles: 800
- Host:   Required.
- N/S:    The target area may always be zeroed to indicate no files.
- Param1: File name offset in CPU Data memory.
- Param2: File name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Finds and fills in the next valid file after the one passed. The passed file
name does not need to be valid (zero terminated UTF-8 string). If there are no
files after the given name, fills in a zero (empty string indicated by
terminator).

Zero (terminator) at a character position is always the first entry for that
position. 0xFF (which is invalid in a file name) is always the last entry.
Otherwise the ordering is implementation defined. The file name need not be
formatted properly (it may even lack a terminator).

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task on completion is always 0x8000.

See "file_io.rst" for further details.


0x0113: Task: Move a file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_move
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful move.
- Param1: Target file name offset in CPU Data memory.
- Param2: Target file name size limit in words.
- Param3: Source file name offset in CPU Data memory.
- Param4: Source file name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Moves (renames) a file, or deletes it. Deleting can be performed by setting
the target name an empty string.

The file names are excepted to be zero terminated UTF-8 strings.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task is 0x8000 if the move succeed. Otherwise bit 14
is set, and bits 0 - 13 provides a fault code.

See "file_io.rst" for further details including fault codes.




Kernel functions, Audio (0x0200 - 0x02FF)
------------------------------------------------------------------------------


(No kernel function in this group)




Kernel functions, Video (0x0300 - 0x03FF)
------------------------------------------------------------------------------


0x0300: Set palette entry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_setpal
- Cycles: 100
- Host:   Required.
- N/S:    This function must be supported if the host produces display.
- Param1: Palette index (only low 8 bits used).
- Param2: Color in 4-4-4 RGB format (only low 12 bits used in this layout).

Changes an entry in the video palette. There are 256 palette entries even in
4 bit mode (although this case the upper 240 entries don't contribute to
display).

Irrespective of whether the host actually produces display or not the palette
data in the Application State (see "state.rst") is updated according the set
colors immediately.

For more on the color representation, see "Palette" in "vid_arch.rst".

The change of a color may only affect display data produced after the call: a
conforming implementation must strictly follow this rule (it may be an issue
on true palettized display modes not in sync with the emulator). The actual
palette updates may delay by multiple frames.


0x0330: Change video mode
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_mode
- Cycles: - (up to one frame or more)
- Host:   Required.
- N/S:    This function must be supported if the host produces display.
- Param1: Requested video mode.

Changes the video mode. The action may include extra stalls to meet
implementation-specific timing requirements during the video mode change.

The contents of the Video RAM, the configuration of the Graphics Display
Generator or the Accelerator, and the palette is not changed by this action.

The following video modes are available:

- 0: 640x400; 4 bit (16 colors).
- 1: 320x400; 8 bit (256 colors).
- 2: 640x200; 4 bit (16 colors), double scanned.
- 3: 320x200; 8 bit (256 colors), double scanned.

Other values passed in Param1 set mode 0 (640x400; 4 bit).


0x0340: Set stereoscopic 3D
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_setst3d
- Cycles: 2400
- Host:   Required.
- N/S:    This function may be ignored (apart from altering the app. state).
- Param1: Stereoscopic 3D output parameters.

Informs the host about current use of stereoscopic 3D. Upon initialization,
this is disabled. The parameter is formatted as follows:

- bit  1- 2: Vertical used pixels (only used if bit 0 is set).
- bit     0: 1 if stereoscopic 3D output is generated, 0 otherwise.

Other bits of the parameter are ignored.

If the application sets stereoscopic 3D, it should continue to render the
image for the left eye on the right half of the display, and the image for
the right eye on the left half (cross-eyed format). If the host supports 3D
devices, it may combine the two halves appropriately for the device by the
information provided through this function.

The Vertical used pixels may be used if the application does not utilize the
entire height of the half-image. The following values are possible:

- 0: 400 pixels (full height used).
- 1: 320 pixels (320x320 rectangular 3D content).
- 2: 240 pixels (4:3 aspect ratio for the 3D content).
- 3: 200 pixels (16:10 aspect ratio for the 3D content).

The application must vertically center the output (start it 0 / 40 / 80 / 100
pixels from the top respectively), and should leave the top and bottom unused
areas showing the darkest color of the current palette. In double scanned
mode all these pixel counts are halved.




Kernel functions, Input devices (0x0400 - 0x04FF)
------------------------------------------------------------------------------


0x0410: Get device properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getprops
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the device is not available.
- Param1: Device to query (only low 4 bits used).
- Ret.X3: Device properties.

The return value provides the properties of the device queried. It is composed
of the following fields:

- bit 12-15: Input device type.
- bit    11: Nonzero indicating the device is available.
- bit  5-10: 0
- bit     4: Set if bits 0-3 contain a valid device ID.
- bit  0- 3: Device ID which this device maps to.

If the device is not available, the return value is zero.

Only device types allowed in the Application Header (see "bin_rpa.rst") may be
returned.

If bit 4 is set, it indicates that the device maps to the same physical device
as an another, and that another device is a more accurate representation (for
example a device type of text input may map to a keyboard).

Before first calling this function, the given device ID behaves like there is
no device behind (all functions returning according to N/S). By calling, the
application notifies the kernel (and by it, the host) that it might want to
use the device, so the device (if any) may come live. The kernel the same time
updates the application state (0x070 - 0x07F, see "state.rst") according to
the return.

For more on the behavior and handling of input devices, see "inputdev.rst".


0x0411: Drop device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_dropdev
- Cycles: 800
- Host:   Required.
- N/S:    May ignore it if this functionality is not necessary for the host.
- Param1: Device to drop (only low 4 bits used).

Notifies the kernel that the application does not need the given device any
more. When encountering this call, the kernel discards the device from the
application state, resetting it's field to zero (0x070 - 0x07F, see
"state.rst"). Furthermore the given device will behave as non-existent (all
functions returning according to N/S).

For more on the behavior and handling of input devices, see "inputdev.rst".


0x0412: Get digital input description symbols
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getdidesc
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the input does not exist.
- Param1: Device to query (only low 4 bits used).
- Param2: Input group to query.
- Param3: Input to query within the group (only low 4 bits used).
- Ret. C: High 16 bits of UTF-32 character.
- Ret.X3: Low 16 bits of UTF-32 character.

Returns a descriptive symbol for the given input point of the given device, or
the information that the input is not available. This function may assist
users using their physical controllers within the application by providing
information by which they may identify the appropriate controls on their
hardware.

The highest bit of the 32bit return value (bit 15 of C) if set identifies
special codes for specific (non-keyboard, or special keys on a keyboard)
devices.

A zero return indicates that the input does not exist.

See "inputdev.rst" for the usage and the complete mapping of this return
value.


0x0422: Get digital inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getdi
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating none of the inputs are active.
- Param1: Device to query (only low 4 bits used).
- Param2: Input group to query.
- Ret.X3: Digital inputs.

The exact role and layout of the directions and buttons vary by device type.
For more information see "inputdev.rst".


0x0423: Get analog inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getai
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the device is centered / idle.
- Param1: Device to query (only low 4 bits used).
- Param2: Analog input to query.
- Ret.X3: 2's complement input value.

The exact role an layout of the analog inputs vary by device type. For more
information see "inputdev.rst".


0x0424: Pop text input FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_popchar
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the FIFO is empty.
- Param1: Device to query (only low 4 bits used).
- Ret. C: High 16 bits of UTF-32 character.
- Ret.X3: Low 16 bits of UTF-32 character.

Note that the text input also returns some text-related control codes which
may be used to assist editing the text. For more information, see
"inputdev.rst".


0x0425: Return area activity
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_checkarea
- Cycles: 1200
- Host:   Required.
- N/S:    May always return 0 indicating the area is inactive.
- Param1: Device to query (only low 4 bits used).
- Param2: Upper left corner, X (0 - 639).
- Param3: Upper left corner, Y (0 - 399).
- Param4: Width.
- Param5: Height.
- Ret.X3: Area activity flags.

The kernel truncates the rectangle to fit on the display treating the upper
left corners as 2's complement values. Note that valid X positions range from
0 - 639 even on 8bit (320 pixels wide) display mode, 639 specifying the
rightmost valid location. A width or height of zero turns off the touch
sensitive area.

The return value may provide the following information:

- bit 0: Set if the area is activated (mouse clicked, touched).
- bit 1: Set if the pointer hovers over the area.

Hover might not be available, so it is possible that an activation is present
without hover (bit 0 set while bit 1 is clear).

For more information, see "inputdev.rst".




Kernel functions, Delay (0x0500 - 0x05FF)
------------------------------------------------------------------------------


0x0500: Delay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_dly_delay
- Cycles: 200 - 65535
- Host:   Not required.
- Param1: Number of cycles to delay.

Passes back control to the kernel while waiting for some event. It will wait
at most the given amount of cycles (consuming up to 200 cycles is allowed
irrespective of the request in the parameter), but might terminate sooner for
implementation specific reasons.

Applications should use this function to "burn" cycles while synchronizing to
absolute time (by audio ticks): by this they strain less a properly designed
emulator.

On real hardware implementations when the kernel receives this call it may use
the provided cycles to perform internal tasks, such as accelerating running
kernel tasks where possible or reducing the time otherwise taken from the
application.




Kernel functions, User information management (0x0600 - 0x06FF)
------------------------------------------------------------------------------


0x0600: Get local users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getlocal
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide User ID information returning all zeros.
- Param1: Target offset in CPU Data memory to load the data into (32 words).

The target area is 32 words long for 4 User ID's (one ID is 8 words long). If
the ID is all zeroes, the user is not available. If the first user is not
available, then all the rest are zeroes.

The application may use this for one part to identify users if they are
available, for an other to determine if multiple users want to use the
application simultaneously (such as local multiplayer games).

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

For the layout of User ID's, see "names.rst".


0x0601: Task: Get UTF-8 representation of User ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getutf
- Cycles: 1200
- Host:   Required.
- N/S:    May not provide this information returning zero strings.
- Param1: Target offset in CPU Data memory to load the main part into.
- Param2: Size limit for the main part in words.
- Param3: Target offset in CPU Data memory to load the extended part into.
- Param4: Size limit for the extended part in words.
- Param5: Offset of 8 word User ID to get the UTF-8 representation of.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

This call can request an UTF-8 representation for any name. The host may
consult a network database to provide this feature.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

For the layout of User ID's, see "names.rst".


0x0610: Get user preferred language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getlang
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide this information returning zero.
- Param1: Language number (0: most preferred, 1: second, etc).
- Ret. C: Preferred language, first bytes.
- Ret.X3: Preferred language, last bytes.

An up to 4 character language code is returned in C:X3, aligned towards the
high bytes, padded with zeros. A zero returns indicates no language
information is present for this and any subsequent language numbers.


0x0611: Get user preferred colors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getcolors
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide this information returning zero.
- Ret. C: Preferred foreground color in (4-)4-4-4 (0)RGB.
- Ret.X3: Preferred background color in (4-)4-4-4 (0)RGB.

Returns the preferred color set of the user if any. If the two colors match
the user has no such preference provided.

For more on the color representation, see "Palette" in "vid_arch.rst".


0x0612: Get user stereoscopic 3D preference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getst3d
- Cycles: 2400
- Host:   Required.
- N/S:    May return zero.
- Ret.X3: 1 if stereoscopic 3D may be used, 0 otherwise.

Returns whether the user is willing to accept stereoscopic 3D or not (see
"0x340: Set stereoscopic 3D" for more). If this function returns zero, the
application should not provide such content, otherwise it may.




Kernel functions, Networking (0x0700 - 0x07FF)
------------------------------------------------------------------------------


0x0700: Task: Send data to user
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_send
- Cycles: 2400
- Host:   Required.
- N/S:    May discard the passed data not sending it out on any network.
- Param1: Source offset in CPU Data memory.
- Param2: Number of words to send.
- Param3: Offset of 8 word User ID to send the packet to.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Sends out a packet from the given source data targeting the given user. The
host manages all the framing guaranteeing that if the packet arrives to the
destination it is correct. Arrival and packet order is not guaranteed.

When sending, local users are ignored, so connecting two RRPGE systems running
the same application with the same User ID's (or no User ID's) set, they
should properly communicate with each other.

The sender User ID is the primary user's User ID (the first user returned by
0x0600: Get local users).

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The kernel task's return value is always 0x8000 on completion.


0x0701: Poll for packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_recv
- Cycles: 2400 + 10/word acquiring packet data.
- Host:   Required.
- N/S:    May always report zero (0), indicating there are no packets ready.
- Param1: Target CPU Data memory offset for raw data.
- Param2: Maximal number of words to receive.
- Param3: Target CPU Data memory offset for User ID of sender (8 words).
- Ret.X3: Count of received data words, 0 indicating no packet is ready.

If there is a packet in the receive buffer, it is popped off and copied to the
target area. Correctness of packages are guaranteed, but not delivery and
neither packet order.

Incoming packets from the network are dropped if they don't fit in the
kernel's receive buffer. This buffer must be able to hold at least 4095 words
of packet data. At least up to 63 distinct packets must be bufferable.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.


0x0710: Task: List accessible users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_listusers
- Cycles: 2400
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating no users are found.
- Param1: Target CPU Data memory offset for the list.
- Param2: Maximal number of User ID's to receive (8 words / ID).
- Param3: Start User ID offset in CPU Data memory (8 words).
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Collects and list users available for the application on the network. Only
users running the same application and having their network availability set
are listed.

The users are listed in incremental order starting from (inclusive if the
user exists) the passed User ID. The list does not contain local users (but
may contain the same User ID's if they reoccur on the network).

The return of the kernel task has bit 15 set (indicating the task is
finished), and on the lower bits the number of users found.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.


0x0720: Set network availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_setavail
- Cycles: 400
- Host:   Required.
- N/S:    This function may be ignored (apart from altering the app. state).
- Param1: 0: Not available, Nonzero: Available.

Indicates whether the user should be available for other users running the
same application on the network or not. This only affects the 0x0710: Task:
List accessible users function (for the other parties on the network).

If networking is not supported by the host, this function may only change the
availability bit at 0x05F of the Application state.


0x0721: Query network availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_getavail
- Cycles: 400
- Host:   Not required.
- Ret.X3: 0: Not available, Nonzero: Available.

Returns the current network availability state (as last set by function
0x0720: Set network availability). This data comes from 0x05F in the
Application State.




Kernel functions, Kernel task management (0x0800 - 0x08FF)
------------------------------------------------------------------------------


0x0800: Query task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_tsk_query
- Cycles: 400
- Host:   Not required.
- Param1: Task index to query.
- Ret.X3: Task status.

Returns information on the given kernel task. The status codes are as follows:

- 0x0000: Empty, next kernel task may take this index.
- 0x0001: Busy, the kernel task was started, and waits for completion.
- 0x8000 - 0xFFFE: Completed, bits 0-14 are completion codes.
- 0xFFFF: Nonexistent index.

The completion codes are described at each kernel function starting a task.


0x0801: Discard task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_tsk_discard
- Cycles: 100
- Host:   Not required.
- Param1: Task index to discard.

Attempts to discard a task. This can only succeed on completed tasks (status
is 0x8000 or above), otherwise it has no effect. If the discard was
successful, the task's status becomes 0x0000 (empty).




Kernel function summary
------------------------------------------------------------------------------


Following a table is provided briefly listing all kernel functions. The
abbreviations used in the table are:

- T:  Whether the function starts a kernel task ('X' if so).
- H:  Host requirement: 'M': Mandatory, 'O': Optional, empty: No host.
- P:  Count of parameters.
- R:  Return value registers used.
- C:  Copy cycles (only for 0x0701: kc_net_recv).

+--------+--------+---+---+---+------+---------------------------------------+
| Fun.ID | Cycles | T | H | P |   R  | Function name                         |
+========+========+===+===+===+======+=======================================+
| 0x0100 |    800 | X | M | 4 |  X3  | kc_sfi_loadbin                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0110 |    800 | X | O | 6 |  X3  | kc_sfi_load                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0111 |    800 | X | O | 6 |  X3  | kc_sfi_save                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0112 |    800 | X | O | 2 |  X3  | kc_sfi_next                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0113 |    800 | X | O | 4 |  X3  | kc_sfi_move                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0300 |    100 |   | M | 2 |      | kc_vid_setpal                         |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0330 |     \- |   | M | 1 |      | kc_vid_mode                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0340 |   2400 |   | O | 1 |      | kc_vid_setst3d                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0410 |    800 |   | O | 1 |  X3  | kc_inp_getprops                       |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0411 |    800 |   | O | 1 |      | kc_inp_dropdev                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0412 |    800 |   | O | 3 | C:X3 | kc_inp_getdidesc                      |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0422 |    800 |   | O | 2 |  X3  | kc_inp_getdi                          |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0423 |    800 |   | O | 1 |  X3  | kc_inp_getai                          |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0424 |    800 |   | O | 1 | C:X3 | kc_inp_popchar                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0425 |   1200 |   | O | 5 |      | kc_inp_checkarea                      |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0500 |  Param |   |   | 1 |      | kc_dly_delay                          |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0600 |   2400 |   | O | 1 |      | kc_usr_getlocal                       |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0601 |   2400 | X | O | 5 |  X3  | kc_usr_getutf                         |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0610 |   2400 |   | O | 1 | C:X3 | kc_usr_getlang                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0611 |   2400 |   | O | 0 | C:X3 | kc_usr_getcolors                      |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0612 |   2400 |   | O | 0 |  X3  | kc_usr_getst3d                        |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0700 |   2400 | X | O | 3 |  X3  | kc_net_send                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0701 | C+2400 |   | O | 3 |  X3  | kc_net_recv                           |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0710 |   2400 | X | O | 3 |  X3  | kc_net_listusers                      |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0720 |    400 |   | O | 1 |      | kc_net_setavail                       |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0721 |    400 |   |   | 0 |      | kc_net_getavail                       |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0800 |    400 |   |   | 1 |  X3  | kc_tsk_query                          |
+--------+--------+---+---+---+------+---------------------------------------+
| 0x0801 |    100 |   |   | 1 |      | kc_tsk_discard                        |
+--------+--------+---+---+---+------+---------------------------------------+
