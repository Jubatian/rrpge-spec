
RRPGE kernel calls
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, the role and layout of the kernel calls
------------------------------------------------------------------------------


The kernel calls are the main interface by which an RRPGE application can
communicate with the host system, or perform certain more or less host
specific actions.

Other important aspects of the kernel are detailed in the "kernel.rst"
documentation.

The kernel call interface is implemented using the "JSV" opcode of the RRPGE
CPU (see "JSV" in "cpu_inst.rst"). The imm6 operand of the call selects the
kernel function to execute. Note that there is no kernel function which would
accept a variable amount of parameters, so all functions have to be called
with the appropriate count of parameters as specified in their descriptions.

The layout of the kernel functions (by the imm6 operand of JSV) are as
follows:

- 0x00 - 0x07: Storage & File management
- 0x08 - 0x0F: Video
- 0x10 - 0x1E: Input devices
- 0x1F - 0x1F: Delay
- 0x20 - 0x27: User information management
- 0x28 - 0x2D: Networking
- 0x2E - 0x2F: Kernel task management


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
function (0x2F) provided in the Kernel task management group.

Kernel tasks typically operate from or into CPU Data memory areas. If the
memory area is a target, it's state is undefined until the completion of the
task. If the memory area is a source and it is modified before the completion
of the task, the result of the task regarding the changed contents, unless
otherwise specified, is undefined.


Unsupported functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When calling an unsupported function (that is the imm6 operand of JSV is some
value not listed in this specification) the kernel is normally required to
terminate the application.

Likewise when calling a supported function with a wrong number of parameters,
the kernel is required to terminate the application.


Invalid parameters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the parameters passed to a kernel function are invalid in some manner (such
as attempting to access application data above the application binary size)
the kernel typically terminates the application. Expections from this rule are
indicated for the appropriate functions.


Notes on extensions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If necessary for implementing a given host, or to provide extra features for
some specific target applications, the kernel may be expanded with additional
functions. The preferred range for this is 0x30 - 0x3F.

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
  implementation.

- To experiment with new functionality which may later be included in the
  main RRPGE specification.

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




Kernel functions, Storage & File management (0x00 - 0x07)
------------------------------------------------------------------------------


0x00: Task: Start loading binary data
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_loadbin
- Cycles: 800
- Host:   Required.
- N/S:    This function must be supported.
- Param0: Target offset in CPU Data memory.
- Param1: Number of words to load.
- Param2: Source application binary offset high word.
- Param3: Source application binary offset low word.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Loads an area of the Application binary into the CPU Data memory. The kernel
terminates the application if either parameter is invalid:

- The target must not be the User Peripheral Area, it must neither wrap around
  to it, and must not have zero size.

- The source must be within the Application binary entirely.

The task always returns 0x8000 on completion.


0x01: Task: Start loading from file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_load
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful load.
- Param0: Target (word) offset in CPU Data memory.
- Param1: Number of bytes (!) to load, up to 16383.
- Param2: Byte offset to start loading from the file, high word.
- Param3: Byte offset to start loading from the file, low word.
- Param4: File name offset in CPU Data memory.
- Param5: File name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Loads bytes from a file. The bytes are loaded in Big Endian order (so first
loaded byte of the file will be the high byte of the first word of the
target).

The file name is expected to be a zero terminated UTF-8 string.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task has bit 14 clear if the load was successful,
bits 0 - 13 indicating the number of bytes successfully loaded (0 - 16383).
This may be less than the requested number of bytes (maybe even zero) if the
file was too small. Bit 14 set in the return value indicates failure, bits
0 - 13 providing a fault code.

See "file_io.rst" for further details including fault codes.


0x02: Task: Start saving into file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_save
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful save.
- Param0: Source (word) offset in CPU Data memory.
- Param1: Number of bytes (!) to save, up to 16383.
- Param2: Byte offset to start at in the file, high word.
- Param3: Byte offset to start at the file, low word.
- Param4: File name offset in CPU Data memory.
- Param5: File name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Saves bytes into the target file. The bytes are saved in Big Endian order (so
first saved byte of the file will be from the high byte of the first word in
the source area).

Note that the host should fail if the file is not sufficiently large already
so the new data can be added without gaps.

The file name is expected to be a zero terminated UTF-8 string.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task has bit 14 clear if the save was successful,
bits 0 - 13 indicating the number of bytes successfully saved (0 - 16383).
This equals to the requested number of bytes to save. Bit 14 set in the return
value indicates failure, bits 0 - 13 providing a fault code.

See "file_io.rst" for further details including fault codes.


0x03: Task: Find next file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_next
- Cycles: 800
- Host:   Required.
- N/S:    The target area may always be zeroed to indicate no files.
- Param0: File name offset in CPU Data memory.
- Param1: File name size limit in words.
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


0x04: Task: Move a file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_move
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0xC000 indicating unsuccessful move.
- Param0: Target file name offset in CPU Data memory.
- Param1: Target file name size limit in words.
- Param2: Source file name offset in CPU Data memory.
- Param3: Source file name size limit in words.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Moves (renames) a file, or deletes it. Deleting can be performed by setting
the target name an empty string.

The file names are expected to be zero terminated UTF-8 strings.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The return of the kernel task is 0x8000 if the move succeed. Otherwise bit 14
is set, and bits 0 - 13 provides a fault code.

See "file_io.rst" for further details including fault codes.




Kernel functions, Video (0x08 - 0x0F)
------------------------------------------------------------------------------


0x08: Set palette entry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_setpal
- Cycles: 100
- Host:   Required.
- N/S:    This function must be supported if the host produces display.
- Param0: Palette index (only low 4 bits used).
- Param1: Color in 4-4-4 RGB format (only low 12 bits used in this layout).

Changes an entry in the video palette. There are 16 palette entries.

Irrespective of whether the host actually produces display or not the palette
data in the Application State (see "state.rst") is updated according the set
colors immediately.

For more on the color representation, see "Palette" in "vid_arch.rst".

The change of a color may only affect display data produced after the call: a
conforming implementation must strictly follow this rule (it may be an issue
on true palettized display modes not in sync with the emulator). The actual
palette updates may delay by multiple frames.


0x0A: Set stereoscopic 3D
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_setst3d
- Cycles: 2400
- Host:   Required.
- N/S:    This function may be ignored (apart from altering the app. state).
- Param0: Stereoscopic 3D output parameters.

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
areas showing the darkest color of the current palette.




Kernel functions, Input devices (0x10 - 0x1E)
------------------------------------------------------------------------------


0x10: Request device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_reqdev
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 if the host doesn't support input.
- Param0: Device ID to request at (only low 4 bits used).
- Param1: Device type to request (only low 4 bits used).
- Ret.X3: Zero if no suitable device exists, one otherwise.

Requests the usage of a device, binding it to the given Device ID. Upon this
call the host should allocate an appropriate device for the application's use,
and subsequently return events for it.

If multiple devices of a given type exist, their order of allocation should be
consistent, that is for example the same (preferably the most suitable) device
should be returned first.

Requesting a device at an already occupied ID drops the previously allocated
device on that ID as if the kc_inp_dropdev function was called on it.

For more information, see "inputdev.rst".


0x11: Drop device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_dropdev
- Cycles: 800
- Host:   Required.
- N/S:    May be ignored if the host doesn't support input.
- Param0: Device ID to drop (only low 4 bits used).

Ceases using the passed device. The host should stop sending events relating
to it. The event queue is flushed.

This function should be used to indicate that a device is no longer used, so
it may be available for further Request Device calls. This is important for
devices supporting multiple types since one physical device can only serve one
type at once.

For more information, see "inputdev.rst".


0x12: Pop input event queue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_pop
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 if the host doesn't support input.
- Ret. C: High 16 bits of event. Zero if the queue is empty.
- Ret.X3: Low 16 bits of event.

Pops next event off the input event queue. This function will always return
zero unless a device is successfully requested using kc_inp_reqdev.

The high 16 bits are structured as follows:

- bit 12-15: Source device ID
- bit    11: Set if Data index is zero, clear otherwise
- bit  8-10: Data index
- bit  4- 7: Device type
- bit  0- 3: Event message type

The Device type and the Event message type together determine the content.

The Data index is provided for supporting more than 16 event bits by sending
it in up to 8 distinct events (long event message). This index indicates which
part of the event message is received.

Bit 11 may be used to detect the beginning of a new event after silence or a
long event message.

A long event message is always delivered in an uninterrupted burst (so no new
event will arrive until all its components are delivered in correct order).
The delivery however may fail skipping some trailing events.

For more information, the types of events supported, see "inputdev.rst".


0x13: Peek input event queue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_peek
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 if the host doesn't support input.
- Ret. C: High 16 bits of event. Zero if the queue is empty.
- Ret.X3: Low 16 bits of event.

Peeks the event queue, checking the next event coming off from it. It doesn't
remove this event from the queue.

Once a peek is made, it is guaranteed that subsequent peeks will return the
same event until a call to kc_inp_pop removes it from the queue, or a call to
kc_inp_dropdev or kc_inp_flush flushing the entire queue.


0x14: Flush input event queue
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_flush
- Cycles: 800
- Host:   Required.
- N/S:    May be ignored if the host doesn't support input.

Flushes the event queue removing all pending events from it.




Kernel functions, Delay (0x1F - 0x1F)
------------------------------------------------------------------------------


0x1F: Delay
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_dly_delay
- Cycles: 200 - 65535
- Host:   Not required.
- Param0: Number of cycles to delay.

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




Kernel functions, User information management (0x20 - 0x27)
------------------------------------------------------------------------------


0x20: Get local users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getlocal
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide User ID information returning all zeros.
- Param0: Target offset in CPU Data memory to load the data into (32 words).

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


0x21: Task: Get UTF-8 representation of User ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getutf
- Cycles: 1200
- Host:   Required.
- N/S:    May not provide this information returning zero strings.
- Param0: Target offset in CPU Data memory to load the main part into.
- Param1: Size limit for the main part in words.
- Param2: Target offset in CPU Data memory to load the extended part into.
- Param3: Size limit for the extended part in words.
- Param4: Offset of 8 word User ID to get the UTF-8 representation of.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

This call can request an UTF-8 representation for any name. The host may
consult a network database to provide this feature.

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

For the layout of User ID's, see "names.rst".


0x22: Get user preferred language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getlang
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide this information returning zero.
- Param0: Language number (0: most preferred, 1: second, etc).
- Ret. C: Preferred language, first bytes.
- Ret.X3: Preferred language, last bytes.

An up to 4 character language code is returned in C:X3, aligned towards the
high bytes, padded with zeros. A zero returns indicates no language
information is present for this and any subsequent language numbers.


0x23: Get user preferred colors
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


0x24: Get user stereoscopic 3D preference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getst3d
- Cycles: 2400
- Host:   Required.
- N/S:    May return zero.
- Ret.X3: 1 if stereoscopic 3D may be used, 0 otherwise.

Returns whether the user is willing to accept stereoscopic 3D or not (see
"0x0A: Set stereoscopic 3D" for more). If this function returns zero, the
application should not provide such content, otherwise it may.




Kernel functions, Networking (0x28 - 0x2D)
------------------------------------------------------------------------------


0x28: Task: Send data to user
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_send
- Cycles: 2400
- Host:   Required.
- N/S:    May discard the passed data not sending it out on any network.
- Param0: Source offset in CPU Data memory.
- Param1: Number of words to send.
- Param2: Offset of 8 word User ID to send the packet to.
- Ret.X3: Index of kernel task or 0x8000 if no more task slots are available.

Sends out a packet from the given source data targeting the given user. The
host manages all the framing guaranteeing that if the packet arrives to the
destination it is correct. Arrival and packet order is not guaranteed.

When sending, local users are ignored, so connecting two RRPGE systems running
the same application with the same User ID's (or no User ID's) set, they
should properly communicate with each other.

The sender User ID is the primary user's User ID (the first user returned by
0x20: Get local users).

The kernel terminates the application if either parameter is invalid:

- The CPU Data memory areas involved must not include the User Peripheral
  Area, neither wrap around to it.

The kernel task's return value is always 0x8000 on completion.


0x29: Poll for packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_recv
- Cycles: 2400 + 10/word acquiring packet data.
- Host:   Required.
- N/S:    May always report zero (0), indicating there are no packets ready.
- Param0: Target CPU Data memory offset for raw data.
- Param1: Maximal number of words to receive.
- Param2: Target CPU Data memory offset for User ID of sender (8 words).
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


0x2A: Task: List accessible users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_listusers
- Cycles: 2400
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating no users are found.
- Param0: Target CPU Data memory offset for the list.
- Param1: Maximal number of User ID's to receive (8 words / ID).
- Param2: Start User ID offset in CPU Data memory (8 words).
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


0x2B: Set network availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_setavail
- Cycles: 400
- Host:   Required.
- N/S:    This function may be ignored (apart from altering the app. state).
- Param0: 0: Not available, Nonzero: Available.

Indicates whether the user should be available for other users running the
same application on the network or not. This only affects the 0x2A: Task: List
accessible users function (for the other parties on the network).

If networking is not supported by the host, this function may only change the
availability bit at 0x05F of the Application state.


0x2C: Query network availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_getavail
- Cycles: 400
- Host:   Not required.
- Ret.X3: 0: Not available, Nonzero: Available.

Returns the current network availability state (as last set by function
0x2B: Set network availability). This data comes from 0x05F in the Application
State.




Kernel functions, Kernel task management (0x2E - 0x2F)
------------------------------------------------------------------------------


0x2E: Query task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_tsk_query
- Cycles: 400
- Host:   Not required.
- Param0: Task index to query.
- Ret.X3: Task status.

Returns information on the given kernel task. The status codes are as follows:

- 0x0000: Empty, next kernel task may take this index.
- 0x0001: Busy, the kernel task was started, and waits for completion.
- 0x8000 - 0xFFFE: Completed, bits 0-14 are completion codes.
- 0xFFFF: Nonexistent index.

The completion codes are described at each kernel function starting a task.


0x2F: Discard task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_tsk_discard
- Cycles: 100
- Host:   Not required.
- Param0: Task index to discard.

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
|   0x00 |    800 | X | M | 4 |  X3  | kc_sfi_loadbin                        |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x01 |    800 | X | O | 6 |  X3  | kc_sfi_load                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x02 |    800 | X | O | 6 |  X3  | kc_sfi_save                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x03 |    800 | X | O | 2 |  X3  | kc_sfi_next                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x04 |    800 | X | O | 4 |  X3  | kc_sfi_move                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x08 |    100 |   | M | 2 |      | kc_vid_setpal                         |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x0A |   2400 |   | O | 1 |      | kc_vid_setst3d                        |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x10 |    800 |   | O | 2 |  X3  | kc_inp_reqdev                         |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x11 |    800 |   | O | 1 |      | kc_inp_dropdev                        |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x12 |    800 |   | O | 0 | C:X3 | kc_inp_pop                            |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x13 |    800 |   | O | 0 | C:X3 | kc_inp_peek                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x14 |    800 |   | O | 0 |      | kc_inp_flush                          |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x1F |  Param |   |   | 1 |      | kc_dly_delay                          |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x20 |   2400 |   | O | 1 |      | kc_usr_getlocal                       |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x21 |   2400 | X | O | 5 |  X3  | kc_usr_getutf                         |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x22 |   2400 |   | O | 1 | C:X3 | kc_usr_getlang                        |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x23 |   2400 |   | O | 0 | C:X3 | kc_usr_getcolors                      |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x24 |   2400 |   | O | 0 |  X3  | kc_usr_getst3d                        |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x28 |   2400 | X | O | 3 |  X3  | kc_net_send                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x29 | C+2400 |   | O | 3 |  X3  | kc_net_recv                           |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x2A |   2400 | X | O | 3 |  X3  | kc_net_listusers                      |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x2B |    400 |   | O | 1 |      | kc_net_setavail                       |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x2C |    400 |   |   | 0 |      | kc_net_getavail                       |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x2E |    400 |   |   | 1 |  X3  | kc_tsk_query                          |
+--------+--------+---+---+---+------+---------------------------------------+
|   0x2F |    100 |   |   | 1 |      | kc_tsk_discard                        |
+--------+--------+---+---+---+------+---------------------------------------+
