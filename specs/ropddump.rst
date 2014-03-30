
RRPGE Read Only Process Descriptor dump
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv1 (version 1 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv1 in the project root.




Introduction, the purpose of the ROPD dump
------------------------------------------------------------------------------


The Read Only Process Descriptor (ROPD; also see the appropriate section in
"mem_map.rst") supplies application state information along with a few other
things to the application. It also provides some constant data.

To provide a definite application state, making it possible to produce cross
emulator state saves, this state, for one, has to be defined, for another, has
to be stored somewhere.

The primary location of storage is the ROPD, the area "underneath" the
constant data. When saving the application state (see "bin_rps.rst"), this
state information shows in this area instead of the for this purpose redundant
constant data.

This area stores not only internal data, but also the state of the
peripherals, that is, the contents of the peripheral registers. Note that the
palette is also contained in the ROPD, but that area (0xC00 - 0xCFF) is also
accessible to the user.




ROPD dump memory map
------------------------------------------------------------------------------


The memory map of the ROPD dump matches that of the ROPD until and including
address 0xD3F. See "Read Only Process Descriptor (ROPD)" in "mem_map.rst" for
details. Further only the area 0xD40 - 0xFFF is defined. Note that this area
is not visible to the application, and is only meant to be used by emulators.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xD40  |                                                                   |
|   -    | CPU general purpose registers: A, B, C, D, X0, X1, X2, X3.        |
| 0xD47  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD48  | CPU XM register                                                   |
+--------+-------------------------------------------------------------------+
| 0xD49  | CPU XH register                                                   |
+--------+-------------------------------------------------------------------+
| 0xD4A  | CPU PC register                                                   |
+--------+-------------------------------------------------------------------+
| 0xD4B  | CPU SP register                                                   |
+--------+-------------------------------------------------------------------+
| 0xD4C  | CPU BP register                                                   |
+--------+-------------------------------------------------------------------+
| 0xD4D  |                                                                   |
|   -    | Unused, must be 0x0000.                                           |
| 0xD4F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD50  | Current video line (0 - 399: Display; 400 -: VBlank)              |
+--------+-------------------------------------------------------------------+
| 0xD51  | Current cycle within line (0 - 79: HBlank; 80 - 399: Display)     |
+--------+-------------------------------------------------------------------+
| 0xD52  | Cycles until next audio event, high.                              |
+--------+-------------------------------------------------------------------+
| 0xD53  | Cycles until next audio event, low.                               |
+--------+-------------------------------------------------------------------+
| 0xD54  | Cycles remaining from video accelerator operation, high.          |
+--------+-------------------------------------------------------------------+
| 0xD55  | Cycles remaining from video accelerator operation, low.           |
+--------+-------------------------------------------------------------------+
| 0xD56  | Currently output audio buffer half; 0: low, 1: high.              |
+--------+-------------------------------------------------------------------+
| 0xD57  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0xD58  | Display layer 0, offset of current line, whole.                   |
+--------+-------------------------------------------------------------------+
| 0xD59  | Display layer 0, offset of current line, fraction.                |
+--------+-------------------------------------------------------------------+
| 0xD5A  | Display layer 1, offset of current line, whole.                   |
+--------+-------------------------------------------------------------------+
| 0xD5B  | Display layer 1, offset of current line, fraction.                |
+--------+-------------------------------------------------------------------+
| 0xD5C  | Display layer 2, offset of current line, whole.                   |
+--------+-------------------------------------------------------------------+
| 0xD5D  | Display layer 2, offset of current line, fraction.                |
+--------+-------------------------------------------------------------------+
| 0xD5E  | Display layer 3, offset of current line, whole.                   |
+--------+-------------------------------------------------------------------+
| 0xD5F  | Display layer 3, offset of current line, fraction.                |
+--------+-------------------------------------------------------------------+
| 0xD60  |                                                                   |
|   -    | First level interrupt CPU state (saves 0xD40 - 0xD4C).            |
| 0xD6C  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD6D  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
|        | Video interrupt flags:                                            |
| 0xD6E  |                                                                   |
|        | - bit 0: Set if video interrupt is flagged.                       |
|        | - bit 1: Set if within video interrupt.                           |
+--------+-------------------------------------------------------------------+
|        | Audio interrupt flags:                                            |
| 0xD6F  |                                                                   |
|        | - bit 0: Set if audio interrupt is flagged.                       |
|        | - bit 1: Set if within audio interrupt.                           |
+--------+-------------------------------------------------------------------+
| 0xD70  |                                                                   |
|   -    | Second level interrupt CPU state (saves 0xD60 - 0xD6C).           |
| 0xD7C  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD7D  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0xD7E  | Current bottom of stack.                                          |
+--------+-------------------------------------------------------------------+
| 0xD7F  | Previous bottom of stack.                                         |
+--------+-------------------------------------------------------------------+
| 0xD80  |                                                                   |
|   -    | 16 * 16 words of kernel task data.                                |
| 0xE7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xE80  | Touch areas, X start (left bound of the area). 0xE80 corresponds  |
|   -    | to the touch area defined for digital input bit 0. Ranges from 0  |
| 0xE8F  | to 639 irrespective of display mode.                              |
+--------+-------------------------------------------------------------------+
| 0xE90  | Touch areas, Y start (top bound of the area). 0xE90 corresponds   |
|   -    | to the touch area defined for digital input bit 0. Ranges from 0  |
| 0xE9F  | to 399 irrespective of display mode.                              |
+--------+-------------------------------------------------------------------+
| 0xEA0  | Touch areas, width. 0xEA0 corresponds to the touch area defined   |
|   -    | for digital input bit 0. Allowed range is so combined with X it   |
| 0xEAF  | fits within the display width of 640 units.                       |
+--------+-------------------------------------------------------------------+
| 0xEB0  | Touch areas, height. 0xEB0 corresponds to the touch area defined  |
|   -    | for digital input bit 0. Allowed range is so combined with Y it   |
| 0xEBF  | fits within the display height of 400 units.                      |
+--------+-------------------------------------------------------------------+
| 0xEC0  |                                                                   |
|   -    | Unused, must be 0x0000.                                           |
| 0xECF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xED0  |                                                                   |
|   -    | Mixer DMA peripheral registers. See "mix_arch.rst" for details.   |
| 0xEDF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xEE0  | Graphics Display & Accelerator registers. See "vid_arch.rst" and  |
|   -    | "acc_arch.rst" for details.                                       |
| 0xEFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xF00  |                                                                   |
|   -    | Reindex table. See "acc_arch.rst" for details.                    |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+




Usage guides
------------------------------------------------------------------------------


Following some details are provided on how the ROPD dump should be used and
populated within emulators so the desired cross-compatibility may be achieved.


0xD50, Current video line
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Depending on the display standard the emulator follows, the count of VBlank
lines may differ. When loading a state which contains a line number larger
than allowed by the emulated video standard, the display should immediately
continue with the next frame, at line 0.


0xD54, Accelerator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After starting an accelerator operation, when exporting state before it's
completion, the emulator should complete the entire operation, and save the
state accordingly. This does not affect applications as they can not access
the Video RAM and the Graphics Display & Accelerator peripheral until the
completion of the operation. It may affect the display if the application
performs operations falling under implementation defined rules (such as
performing an accelerator operation over a display list which is the same
time read for display).


0xD58, Display lists
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An emulator should evaluate the line start offsets when transitioning to the
next line and update the offsets immediately. Note that as defined in "Layer
display lists" from "vid_arch.rst", Line 0 assumes a start offset of zero if a
relative pointer is specified in the display list.


0xD60, Interrupts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These locations are used to keep the necessary kernel state (mostly supposedly
on a kernel stack) around for restoring interrupt levels stacked upon each
other.

The "interrupt flagged" flags indicate if an event is waiting to be serviced.
These stay set until the kernel gets the chance to service these, and upon
entry, before any user handler is called (if any is enabled), they are
cleared.

If both "within interrupt" flags are set, an audio interrupt is stacked upon a
video interrupt (the audio interrupt has a higher priority).

The stack bottom is a kernel barrier, guarding against accesses below it. In
the mainline it is 0, upon entering an interrupt it is set to the location
where BP + SP was before entry. The 0xD7E and 0xD7F fields realize the
necessary stack of these barriers (note that while an audio interrupt which
interrupted a video interrupt executes, both fields may be nonzero).

For more information on interrupts, see "Interrupts" in "cpu_arch.rst" and
"Supported events" in "kernel.rst".


0xD80, Kernel tasks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Up to 16 simultaneously executing kernel tasks are supported whose states are
saved on these locations. 0xD80 - 0xD8F refers to kernel task zero, and so on.

The first 15 words of each kernel task provide the parameters with which the
task was started (these are the parameters of the supervisor call which
started the task). The first of these is the kernel call identifier.

The last word is the task status as readable by the 0x0800 "Query task" kernel
function.

When restoring a state having an incomplete kernel task, the task should be
restarted. This normally shouldn't affect the application (except if it
attempts to rely on an undefined behavior described in the "Kernel tasks"
chapter of "kcall.rst").


0xED0, Mixer DMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An emulator should execute a Mixer operation as one uninterruptible block, and
prepare the state accordingly.
