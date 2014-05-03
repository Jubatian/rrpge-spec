
RRPGE kernel calls
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction, the role and layout of the kernel calls
------------------------------------------------------------------------------


The kernel calls are the main interface by which an RRPGE application can
communicate with the host system, or perform certain more or less host
specific actions. Moreover kernel calls are in place to support the tight
user mode - supervisor mode separation of the RRPGE CPU, such as to provide
means for switching in and out memory banks from the CPU's address space.

Other important aspects of the kernel including events (interrupts) and their
handlers are detailed in the "kernel.rst" documentation.

The kernel call interface is implemented using the "JSV" opcode of the RRPGE
CPU (see "JSV" in "cpu_inst.rst"). The first parameter of the call selects the
kernel function to execute while further parameters may be passed to the
function itself. Note that there is no kernel function which would accept a
variable amount of parameters, so all functions have to be called with the
appropriate count of parameters as specified in their descriptions.

The layout of the kernel functions (by the value of the first parameter
selecting it) are as follows:

- 0x0000-0x00FF: Memory management
- 0x0100-0x01FF: Storage & File management
- 0x0200-0x02FF: Audio
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

Kernel tasks typically operate from or into memory areas. If the memory area
is a target, it's state is undefined until the completion of the task. If the
memory area is a source and it is modified before the completion of the task,
the result of the task regarding the changed contents, unless otherwise
specified, is undefined.


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
as attempting to access a memory page which is not suitable for the given
function) the kernel typically terminates the application. Exceptions from
this rule are indicated for the appropriate functions.


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

Kernel functions either has no return, return in the register "A", or in "C:A"
depending on the amount of direct return information they produce. No other
registers are affected. If the kernel function requires to produce more return
data, it takes pointers to memory areas it should fill.




Kernel functions, Memory management (0x0000 - 0x00FF)
------------------------------------------------------------------------------


0x0000: Bank in Read memory page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_mem_bankrd
- Cycles: 150
- Host:   Not required.
- Param1: Target CPU Read bank (Only low 4 bits used).
- Param2: Source bank (Data memory, Video memory, Peripheral areas, ROPD).

Banks a new memory page in the CPU's Read address space replacing the previous
bank. If the source bank number is invalid, the kernel terminates the
application.

If the CPU's Write address space for the target bank contains a Video memory
page or the Video peripheral area, the kernel also terminates the application
(since the Video memory can only operate correctly if the CPU performs R-M-W
operations on it for all it's writes). Note that this happens even if the
read bank would not be changed by performing the call.


0x0001: Bank in Write memory page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_mem_bankwr
- Cycles: 150
- Host:   Not required.
- Param1: Target CPU Write bank (Only low 4 bits used).
- Param2: Source bank (Data memory, Video memory, Peripheral areas).

Banks a new memory page in the CPU's Write address space replacing the
previous bank. If the source bank number is invalid, the kernel terminates the
application.

A Video memory page can only be banked in if the CPU's Read address space for
the target bank already contains the same Video memory page. Similarly the
Video peripheral area can only be banked in if it is already in the CPU's
Read address space for the target bank. Violating these constraints cause the
kernel terminating the application. (The Video memory can only operate
correctly if the CPU performs R-M-W operations on it for all it's writes).


0x0002: Bank in both Read & Write pages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_mem_bank
- Cycles: 150
- Host:   Not required.
- Param1: Target CPU Read & Write bank (Only low 4 bits used).
- Param2: Source Read bank.
- Param3: Source Write bank.

Banks new memory pages in the CPU's Read & Write address spaces. The same
constraints apply for this function like for the individual banking functions,
however with this is is easier to swap out Video memory pages since only the
requested state's validity is checked.


0x0003: Bank in the same page for Read & Write
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_mem_banksame
- Cycles: 150
- Host:   Not required.
- Param1: Target CPU Read & Write bank (Only low 4 bits used).
- Param2: Source bank (for both Read and Write).

Banks a mamory page in the CPU's Read & Write address spaces. The same
constraints apply for this function like for the individual banking functions,
however with this is is easier to swap in and out Video memory pages.




Kernel functions, Storage & File management (0x0100 - 0x01FF)
------------------------------------------------------------------------------


0x0100: Task: Start loading binary data page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_loadbin
- Cycles: 800
- Host:   Required.
- N/S:    This function must be supported.
- Param1: Target Data memory page.
- Param2: Source binary page high word.
- Param3: Source binary page low word.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Loads a binary page into a Data memory page from the application binary. The
kernel terminates the application if either parameter is invalid. The count of
available source pages is determined by the 0xBC2-0xBC3 area of the
application header, see "bin_rpa.rst" for details.

Binary pages in the application binary start after the Code pages with Page 0
(so the Application header page and Code pages are not included).

The task always returns 0x8000 on completion.


0x0110: Task: Start loading nonvolatile save
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_loadnv
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating unsuccessful load.
- Param1: First target Data memory page.
- Param2: Maximal number of pages to fill.
- Param3 - Param5: 3 word Save ID to load from.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Loads a nonvolatile save starting from the target Data memory page. The
maximal number of pages to fill must select pages in the user accessible Data
memory page range only (pages 0x4000 - 0x40DF). If the parameters don't meet
these constraints, the kernel terminates the application.

The return of the kernel task has bit 15 set (indicating the task is
finished), and in the lower bits (bits 0-14) the count of pages the source
actually had, zero indicating unsuccessful load.

The target area is always completely filled: pages for which no source data
was available are filled with zeroes. If the source had more pages than the
provided range, the excess pages are not loaded.

For guidelines on forming Save ID's, see "names.rst". For more information on
the format of nonvolatile saves, see "bin_rpn.rst".


0x0111: Task: Start saving nonvolatile save
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_savenv
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating unsuccessful save.
- Param1: First source Data memory page.
- Param2: Number of pages to save.
- Param3 - Param5: 3 word Save ID to save into.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Saves an area of Data memory into a nonvolatile save. It will delete and
refill any existing save with the same Save ID. If the number of pages to save
is zero, no new save is created: this can be used to delete existing saves.
The task returns 0x8001 on success indicating that the save could be written.
In the case of deletion the result is always success if after the operation no
save exists by the given Save ID. In case of failure the task returns 0x8000.

Before writing the save the kernel will apply the appropriate nonvolatile save
header on the output data overriding any content on this area. This affects
the first 48 words of the source which will be overwritten by this header
during the execution of this task. Note that this part of the operation is
atomic from the point of the user program, that is it has no possibility to
alter the output of the header in any manner.

For guidelines on forming Save ID's, see "names.rst". For more information on
the format of nonvolatile saves, see "bin_rpn.rst".


0x0112: Task: List available nonvolatile saves
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_listnv
- Cycles: 800
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating no saves are available.
- Param1: Target Data memory page.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Builds a list of all existing saves. In the target bank one save takes 4
words, in total up to 1024 nonvolatile saves are supported for a given
application. The layout of the fields are as follows:

- Word 0: First word of the Save ID.
- Word 1: Second word of the Save ID.
- Word 2: Third word of the Save ID.
- Word 3: Size of the save in pages.

The existing saves are listed in an incremental order by their Save ID's. The
rest of the page is padded with zeros. The size of the save can only be zero
in the padding, note that a save with all three ID words of zero may also
exists. The maximal allowed size for a save is 224 pages (the size of Data
memory accessible to the user), the host should generally not allow larger
saves to exist. If so, their size should be reported to be 224 pages.

The return of the kernel task has bit 15 set (indicating the task is
finished), and on the lower bits the number of saves found (0 - 1024
inclusive).

For guidelines on forming Save ID's, see "names.rst". For more information on
the format of nonvolatile saves, see "bin_rpn.rst".


0x0120: Load arbitrary file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_loadfile
- Cycles: -
- Host:   Required.
- N/S:    May always return zero (0) indicating no file was loaded.
- Param1: First target Data memory page.
- Param2: Maximal number of pages to fill.
- Ret. C: File size in bits, high (number of full pages it occupies).
- Ret. A: File size in bits, low.

Asks the kernel to offer loading an arbitrary file. The kernel takes over
presenting some kind of file selection screen to allow the user loading a
file, then when the user either selected a file or canceled the selection,
returns. If the file load failed for an other reason than the user canceling
it, the kernel produces an appropriate error message within this file
selection screen. In case of an emulated system, these tasks are performed by
the host.

The target area is always filled, excess area beyond the size of the file (or
the whole area if the return is zero) with zeroes.

The target area must be at least one page long. Otherwise (and on any other
invalid parameter combination) the kernel terminates the application.

The return value contains the size of the complete file (if it is larger than
64K words, the size is truncated to fit), or zero if no file was selected or
it could not be loaded.

When executing this function (even if unsupported), on return the kernel
clears the first 32 Video memory pages and the Video peripheral registers are
reset (see "boot.rst" for their initial state) including the Reindex table. No
other registers are affected: The palette is restored by the kernel before
return.

A well-behaving kernel should return on the start of a Vertical blank so the
user program can set up the next display frame without glitching. This however
is not mandatory.

The host may attempt to figure out the preferred file type to load by
examining the data found in the area offered for loading the file into. To
assist this the application may prepare a header resembling the file type it
would like to load.


0x0121: Save arbitrary file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_sfi_savefile
- Cycles: -
- Host:   Required.
- N/S:    May always return zero (0) indicating failure.
- Param1: First source Data memory page.
- Param2: File size in bits to save, high (number of full pages).
- Param3: File size in bits to save, low.
- Ret. A: 1 on success, 0 on failure.

Asks the kernel to offer saving an arbitrary file. The kernel takes over
presenting some kind of file selection screen to allow the user saving a
file, then when the user either selected a file, gave a file name, and the
save was carried out, or the user canceled the selection, returns. If the
file save failed for an other reason than the user canceling it, the kernel
produces an appropriate error message within this file selection screen. In
case of an emulated system, these tasks are performed by the host.

The file is padded with zeroes if necessary to fit with the host's constraints
on file size granularity.

The source size must be at least one bit. Otherwise (and on any other invalid
parameter combination) the kernel terminates the application.

When executing this function (even if unsupported), on return the kernel
clears the first 32 Video memory pages and the Video peripheral registers are
reset (see "boot.rst" for their initial state) including the Reindex table. No
other registers are affected: The palette is restored by the kernel before
return.

A well-behaving kernel should return on the start of a Vertical blank so the
user program can set up the next display frame without glitching. This however
is not mandatory.

The host may attempt to figure out the preferred file type to save by
examining the data offered for saving.




Kernel functions, Audio (0x0200 - 0x02FF)
------------------------------------------------------------------------------


0x0210: Set audio event handler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_aud_sethnd
- Cycles: 150
- Host:   Not required.
- Param1: Event handler entry point.

Sets the audio half-buffer exhausted event handler function. Setting it to
zero cancels the event handler. For more on this handler see the "Audio
half-buffer exhausted" section of "kernel.rst".




Kernel functions, Video (0x0300 - 0x03FF)
------------------------------------------------------------------------------


0x0300: Set palette entry
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_setpal
- Cycles: 100 + Video stall
- Host:   Required.
- N/S:    This function must be supported if the host produces display.
- Param1: Palette index (only low 8 bits used).
- Param2: Color in 4-4-4 RGB format (only low 12 bits used in this layout).

Changes an entry in the video palette. There are 256 palette entries even in
4 bit mode (although this case the upper 240 entries don't contribute to
display).

Irrespective of whether the host actually produces display or not the palette
data in the Read Only Process Descriptor is updated according to the set
colors, immediately (see "Read Only Process Descriptor (ROPD)" in
"mem_map.rst").

For more on the color representation, see "Palette" in "vid_arch.rst".

The change of a color may only affect display data produced after the call: a
conforming implementation must strictly follow this rule (it may be an issue
on true palettized display modes not in sync with the emulator). The actual
palette updates may delay by multiple frames.


0x0310: Set video event handler
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_sethnd
- Cycles: 150 + Video stall
- Host:   Not required.
- Param1: Event handler entry point.
- Param2: Line to fire the event on.

Sets the video raster passed event handler function. Setting it to zero
cancels the event handler. For more on this handler see the "Video raster
passed" section of "kernel.rst".

Lines 0 - 399 can be specified for requesting an event on entering the
Horizontal blanking (preceding the display) of these lines. Line 400 or
above specifies requesting an event on entering the Vertical blanking period
(note that the actual value above 400 is ignored, they all request an event on
the beginning of the vertical blanking).

When designing graphic engines depending on this feature, attention should be
paid to the minimal timing constraints defined in "Basic properties of the
display" of "vid_arch.rst" and "Kernel timing constraints" of "kernel.rst".

Note that if the requested line to fire the event on is equal or smaller than
the currently displaying line, the event will only happen in the next frame.


0x0320: Query current display line
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_vid_getline
- Cycles: 150 + Video stall
- Host:   Not required.
- Ret. A: Current display line (0 - 399) or lines until next frame (<0).

Returns the currently displaying line. When within vertical blanking, it
returns the count of lines remaining from the vertical blanking as a 2's
complement negative number.

Note that at least 49 Vertical blank lines are available, however up to 17 of
these may be taken by the kernel for internal tasks.




Kernel functions, Input devices (0x0400 - 0x04FF)
------------------------------------------------------------------------------


0x0400: Get input device availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getdev
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating no devices are available.
- Ret. A: Bit mask of available devices.

Returns the input devices available, up to 16. Bit 0 of the return refers to
device 0. Devices not necessary occur continuously.


0x0410: Get device properties
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getprops
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the device is not available.
- Param1: Device to query (only low 4 bits used).
- Ret. A: Device properties.

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

If bit 4 is set, it indicates that de device maps to the same physical device
as an another, and that another device is a more accurate representation (for
example a device type of digital gamepad may map to a keyboard).

For more on the behavior and handling of input devices, see "inputdev.rst".


0x0411: Get digital input description symbols
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getdidesc
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the input does not exist.
- Param1: Device to query (only low 4 bits used).
- Param2: Input group to query.
- Param3: Input to query within the group (only low 4 bits used).
- Ret. C: High 16 bits of UTF-32 character.
- Ret. A: Low 16 bits of UTF-32 character.

Returns a descriptive symbol for the given input point of the given device, or
the information that the input is not available. This function may assist
users using their physical controllers within the application by providing
information by which they may identify the appropriate controls on their
hardware.

The highest bit of the 32bit return value (bit 15 of C) if set identifies
special codes for specific (non-keyboard, or special keys on a keyboard)
devices.

A zero return indicates that the input does not exist (however in group 0 it
still may be activated by touch if touch is enabled and the areas are
defined).

See "inputdev.rst" for the usage and the complete mapping of this return
value.


0x0420: Get digital inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getdi
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating none of the inputs are active.
- Param1: Device to query (only low 4 bits used).
- Param2: Input group to query.
- Ret. A: Digital inputs.

The exact role and layout of the directions and buttons vary by device type.
Note that for group 0 touch sensitive areas may also provide input. For more
information see "inputdev.rst".


0x0421: Get analog inputs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_getai
- Cycles: 800
- Host:   Required.
- N/S:    May always return 0 indicating the device is centered.
- Param1: Device to query (only low 4 bits used).
- Param2: Analog input to query.
- Ret. A: 2's complement input value.

The exact role an layout of the analog inputs vary by device type. For more
information see "inputdev.rst".


0x0423: Pop text input FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_popchar
- Cycles: 800
- Host:   Required.
- N/S:    May always return zero (0) indicating the FIFO is empty.
- Param1: Device to query (only low 4 bits used).
- Ret. C: High 16 bits of UTF-32 character.
- Ret. A: Low 16 bits of UTF-32 character.

Note that the text input also returns some text-related control codes which
may be used to assist editing the text. For more information, see
"inputdev.rst".


0x0430: Define touch sensitive area
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_inp_settouch
- Cycles: 800
- Host:   Required.
- N/S:    May ignore this function.
- Param1: Area ID to define (only low 4 bits used).
- Param2: Upper left corner, X (0 - 639).
- Param3: Upper left corner, Y (0 - 399).
- Param4: Width
- Param5: Height

The kernel truncates the rectangle to fit on the display treating the upper
left corners as 2's complement values. Note that valid X positions range from
0 - 639 even on 8bit (320x200) display mode, 639 specifying the rightmost
valid location. A width or height of zero turns off the touch sensitive area.

The definition of the rectangle always updates within the application's state
even if the particular host does not support touch. Saving the application
state and restoring it on a touch capable device will so work properly.

The touch sensitive areas generate digital input activity for the "Get digital
inputs" (0x0420) function for pointing devices or devices emulated on a
physical touch device. The area ID to define matches the bit's number it
activates on the digital input, input group 0.

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
irrespective of the request in the parameter), but will terminate sooner if
either an audio or video event occurs. It might also terminate sooner for any
other implementation specific reasons.

Applications should use this function to "burn" cycles while synchronizing to
absolute time (using audio events): by this they strain less a properly
designed emulator.

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
- Param1: Target Data memory page to load the data into.
- Param2: Start offset within page (only bits 5-11 are used).

The target area is 32 words long for 4 User ID's (one ID is 8 words long). If
the ID is all zeroes, the user is not available. If the first user is not
available, then all the rest are zeroes.

The application may use this for one part to identify users if they are
available, for an other to determine if multiple users want to use the
application simultaneously (such as local multiplayer games).

For the layout of User ID's, see "names.rst".


0x0601: Task: Get UTF-8 representation of User ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getutf
- Cycles: 1200
- Host:   Required.
- N/S:    May not provide this information returning zero strings.
- Param1: Target Data memory page to load the data into.
- Param2: Start offset within page (only bits 8-11 are used).
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

The User ID is broken in two parts, each of which an UTF-8 representation may
be provided for. The target memory area is 256 words, of which the first 128
words are reserved for the main part, while the last 128 words are for the
extended part. Both parts are terminated with a zero word at offsets 127 and
255 (so up to at most 254 useful UTF-8 bytes may be present).

This call can request an UTF-8 representation for any name. The host may
consult a network database to provide this feature.

For the layout of User ID's, see "names.rst".


0x0610: Get user preferred language
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getlang
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide this information returning zero.
- Param1: Language number (0: most preferred, 1: second, etc).
- Ret. C: Preferred language, first bytes.
- Ret. A: Preferred language, last bytes.

An up to 4 character language code is returned in C:A, aligned towards the
high bytes, padded with zeros. A zero returns indicates no language
information is present for this and any subsequent language numbers.


0x0611: Get user preferred colors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_usr_getcolors
- Cycles: 2400
- Host:   Required.
- N/S:    May not provide this information returning zero.
- Ret. C: Preferred foreground color in 5-6-5 RGB.
- Ret. A: Preferred background color in 5-6-5 RGB.

Returns the preferred color set of the user if any. If the two colors match
the user has no such preference provided.

For more on the color representation, see "Palette" in "vid_arch.rst".




Kernel functions, Networking (0x0700 - 0x07FF)
------------------------------------------------------------------------------


0x0700: Task: Send data to user
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_send
- Cycles: 2400
- Host:   Required.
- N/S:    May discard the passed data not sending it out on any network.
- Param1: Source Data memory page.
- Param2: Number of words to send (only low 12 bits used; 0: 4096).
- Param3 - Param10: 8 word User ID to send the packet to.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Sends out a packed from the given source data targeting the given user. The
host manages all the framing guaranteeing that if the packet arrives to the
destination it is correct. Arrival and packet order is not guaranteed.

When sending local users are ignored, so connecting two RRPGE systems running
the same application with the same User ID's (or no User ID's) set, they
should properly communicate with each other.

The sender User ID is the primary user's User ID (the first user returned by
0x0600: Get local users).

The kernel task's return value is always 0x8000 on completion.


0x0701: Poll for packets
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_recv
- Cycles: 2400 + 10/word acquiring packet data.
- Host:   Required.
- N/S:    May always report zero (0), indicating there are no packets ready.
- Param1: Target Data memory page for raw data.
- Param2: Target Data memory page for User ID of sender.
- Param3: Target start offset for User ID (bits 3-11 used).
- Ret. A: Count of received data words, 0 indicating no packet is ready.

If there is a packet in the receive buffer, it is popped off and copied to the
target area. Correctness of packages are guaranteed, but not delivery and
neither packet order.

Incoming packets from the network are dropped if they don't fit in the
kernel's receive buffer. This buffer must be at least one page (4096 words)
providing for up to 4095 words of packet data. At least up to 63 distinct
packets must be bufferable.

The User ID of the sender is 8 words long.


0x0710: Task: List accessible users
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_listusers
- Cycles: 2400
- Host:   Required.
- N/S:    The task may always return 0x8000 indicating no users are found.
- Param1: Target Data memory page for the list.
- Param2 - Param9: Start User ID.
- Ret. A: Index of kernel task or 0x8000 if no more task slots are available.

Collects and list users available for the application on the network. Only
users running the same application and having their network availability set
are listed.

The users are listed in incremental order starting from (inclusive if the
user exists) the passed User ID. The list does not contain local users (but
may contain the same User ID's if they reoccur on the network).

The return of the kernel task has bit 15 set (indicating the task is
finished), and on the lower bits the number of users found (0 - 256
inclusive). The target page is padded with zeros if less than 256 users are
returned.


0x0720: Set network availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_net_setavail
- Cycles: 400
- Host:   Required.
- N/S:    This function may be ignored (apart from altering the ROPD).
- Param1: 0: Not available, Nonzero: Available.

Indicates whether the user should be available for other users running the
same application on the network or not. This only affects the 0x0710: Task:
List accessible users function (for the other parties on the network).

If networking is not supported by the host, this function may only change the
availability bit of the Read Only Process Descriptor.




Kernel functions, Kernel task management (0x0800 - 0x08FF)
------------------------------------------------------------------------------


0x0800: Query task
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: kc_tsk_query
- Cycles: 400
- Host:   Not required.
- Param1: Task index to query.
- Ret. A: Task status.

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
- H:  Host requirement: 'M': mandatory, 'O': Optional.
- P:  Count of parameters.
- R:  Return value registers used.
- VS: Video Stall.
- C:  Copy cycles (only for 0x0701: kc_net_recv).

+--------+--------+---+---+----+-----+---------------------------------------+
| Fun.ID | Cycles | T | H |  P |  R  | Function name                         |
+========+========+===+===+====+=====+=======================================+
| 0x0000 |    150 |   |   |  2 |     | kc_mem_bankrd                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0001 |    150 |   |   |  2 |     | kc_mem_bankwr                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0002 |    150 |   |   |  3 |     | kc_mem_bank                           |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0003 |    150 |   |   |  2 |     | kc_mem_banksame                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0100 |    800 | X | M |  3 |  A  | kc_sfi_loadbin                        |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0110 |    800 | X | O |  5 |  A  | kc_sfi_loadnv                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0111 |    800 | X | O |  5 |  A  | kc_sfi_savenv                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0112 |    800 | X | O |  1 |  A  | kc_sfi_listnv                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0120 |     \- |   | O |  2 | C:A | kc_sfi_loadfile                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0121 |     \- |   | O |  3 |  A  | kc_sfi_savefile                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0210 |    150 |   |   |  1 |     | kc_aud_sethnd                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0300 | VS+100 |   | M |  2 |     | kc_vid_setpal                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0310 | VS+150 |   |   |  2 |     | kc_vid_sethnd                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0320 | VS+150 |   |   |  0 |  A  | kc_vid_getline                        |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0400 |    800 |   | O |  0 |  A  | kc_inp_getdev                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0410 |    800 |   | O |  1 |  A  | kc_inp_getprops                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0411 |    800 |   | O |  3 | C:A | kc_inp_getdidesc                      |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0420 |    800 |   | O |  2 |  A  | kc_inp_getdi                          |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0421 |    800 |   | O |  1 |  A  | kc_inp_getai                          |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0423 |    800 |   | O |  1 | C:A | kc_inp_popchar                        |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0430 |    800 |   | O |  5 |     | kc_inp_settouch                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0500 |  Param |   |   |  1 |     | kc_dly_delay                          |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0600 |   2400 |   | O |  2 |     | kc_usr_getlocal                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0601 |   2400 | X | O |  2 |  A  | kc_usr_getutf                         |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0610 |   2400 |   | O |  1 | C:A | kc_usr_getlang                        |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0611 |   2400 |   | O |  0 | C:A | kc_usr_getcolors                      |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0700 |   2400 | X | O | 10 |  A  | kc_net_send                           |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0701 | C+2400 |   | O |  3 |  A  | kc_net_recv                           |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0710 |   2400 | X | O |  9 |  A  | kc_net_listusers                      |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0720 |    400 |   | O |  1 |     | kc_net_setavail                       |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0800 |    400 |   |   |  1 |  A  | kc_tsk_query                          |
+--------+--------+---+---+----+-----+---------------------------------------+
| 0x0801 |    100 |   |   |  1 |     | kc_tsk_discard                        |
+--------+--------+---+---+----+-----+---------------------------------------+
