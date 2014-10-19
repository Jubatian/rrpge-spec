
RRPGE System State dump
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, the purpose of the System State dump
------------------------------------------------------------------------------


The System State dump provides a defined set of state variables which are
sufficient for saving and restoring the complete state of an RRPGE application
in an emulator. In actual hardware implementations or in emulators not aiming
to be capable to create application state saves, realizing these the manner
they are described here is not necessary.




System State memory map
------------------------------------------------------------------------------


The System State consists the following major blocks of 16 bit data:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x000  |                                                                   |
| \-     | Reserved for containing the Application header in a state save.   |
| 0x03F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x040  |                                                                   |
| \-     | State variables.                                                  |
| 0x08F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x090  |                                                                   |
| \-     | Mixer registers.                                                  |
| 0x09F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0A0  |                                                                   |
| \-     | Accelerator registers.                                            |
| 0x0BF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x0C0  |                                                                   |
| \-     | User Peripheral Area contents.                                    |
| 0x0FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x100  | Current palette. Colors are defined according to "Palette" in the |
| \-     | Graphics Display Generator's documentation ("vid_arch.rst").      |
| 0x1FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x200  |                                                                   |
| \-     | 16 * 16 words of kernel task data.                                |
| 0x2FF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x300  |                                                                   |
| \-     | Accelerator reindex table.                                        |
| 0x3FF  |                                                                   |
+--------+-------------------------------------------------------------------+

The State variables area's map:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0x040  |                                                                   |
| \-     | CPU general purpose registers: A, B, C, D, X0, X1, X2, X3.        |
| 0x047  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x048  | CPU XM register                                                   |
+--------+-------------------------------------------------------------------+
| 0x049  | CPU XH register                                                   |
+--------+-------------------------------------------------------------------+
| 0x04A  | CPU PC register                                                   |
+--------+-------------------------------------------------------------------+
| 0x04B  | CPU SP register                                                   |
+--------+-------------------------------------------------------------------+
| 0x04C  | CPU BP register                                                   |
+--------+-------------------------------------------------------------------+
| 0x04D  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0x04F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x050  | Current video line (2's complement; 0 - 399: Display; Negatives:  |
|        | Vertical blank).                                                  |
+--------+-------------------------------------------------------------------+
| 0x051  | Cycles until next video line.                                     |
+--------+-------------------------------------------------------------------+
|        | Current video mode.                                               |
| 0x052  |                                                                   |
|        | 0: 640x400, 4bit                                                  |
|        | 1: 320x400, 8bit                                                  |
|        | 2: 640x200, 4bit (double scan)                                    |
|        | 3: 320x200, 8bit (double scan)                                    |
+--------+-------------------------------------------------------------------+
| 0x053  | Cycles until next 48KHz audio base clock tick.                    |
+--------+-------------------------------------------------------------------+
| 0x054  | 48KHz audio base clock (incrementing).                            |
+--------+-------------------------------------------------------------------+
|        | Latched Display List Definition register. Stores the previous     |
| 0x055  | value of the Display List Definition register until the Display   |
|        | List clear completes, then updates to the current value.          |
+--------+-------------------------------------------------------------------+
| 0x056  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0x057  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x058  | Application binary total size in words, high. Copies 0x0000 from  |
|        | the Application descriptor.                                       |
+--------+-------------------------------------------------------------------+
| 0x059  | Application binary total size in words, low. Copies 0x0001 from   |
|        | the Application descriptor.                                       |
+--------+-------------------------------------------------------------------+
| 0x05A  | Count of stack words. Copies 0x0008 from the Application          |
|        | descriptor.                                                       |
+--------+-------------------------------------------------------------------+
| 0x05B  | Start offset of stack. Copies 0x0009 from the Application         |
|        | descriptor.                                                       |
+--------+-------------------------------------------------------------------+
| 0x05C  | Input controller types the application may see. Copies 0x000A     |
|        | from the Application descriptor.                                  |
+--------+-------------------------------------------------------------------+
| 0x05D  | Application flags. Copies 0x000B from the Application descriptor. |
+--------+-------------------------------------------------------------------+
| 0x05E  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0x05F  | Network availability flag on bit 0 (1: Available). Other bits     |
|        | must be zero.                                                     |
+--------+-------------------------------------------------------------------+
| 0x060  | Mixer FIFO write pointer.                                         |
+--------+-------------------------------------------------------------------+
| 0x061  | Mixer FIFO read pointer.                                          |
+--------+-------------------------------------------------------------------+
| 0x062  | Cycles remaining from mixer operation or FIFO fetch, high.        |
+--------+-------------------------------------------------------------------+
| 0x063  | Cycles remaining from mixer operation or FIFO fetch, low.         |
+--------+-------------------------------------------------------------------+
| 0x064  | Mixer FIFO address latch.                                         |
+--------+-------------------------------------------------------------------+
| 0x065  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0x067  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x068  | Graphics FIFO write pointer.                                      |
+--------+-------------------------------------------------------------------+
| 0x069  | Graphics FIFO read pointer.                                       |
+--------+-------------------------------------------------------------------+
| 0x06A  | Cycles remaining from accelerator operation or FIFO fetch, high.  |
+--------+-------------------------------------------------------------------+
| 0x06B  | Cycles remaining from accelerator operation or FIFO fetch, low.   |
+--------+-------------------------------------------------------------------+
| 0x06C  | Graphics FIFO address latch.                                      |
+--------+-------------------------------------------------------------------+
| 0x06D  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0x06F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0x070  | Expected devices at each device ID's. The 0x0410 "Get device      |
| \-     | properties" and the 0x0411 "Drop device" kernel calls manage      |
| 0x07F  | these fields.                                                     |
+--------+-------------------------------------------------------------------+
| 0x080  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0x08F  |                                                                   |
+--------+-------------------------------------------------------------------+




Usage guides
------------------------------------------------------------------------------


Following some details are provided on how the application state should be
used and populated within emulators so the desired cross-compatibility may be
achieved.


0x050: Current video line
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Depending on the display standard the emulator follows, the count of VBlank
lines may differ. Up to 250 remaining Vertical blank lines should be
tolerated.


0x063; 0x06B: Cycle counters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After starting an Accelerator or Mixer operation, when exporting state before
it's completion, the emulator should complete the entire operation, and save
the state accordingly. This usually should not affect applications as there
are no reference points to adequately rely on individual operation timings.

If an Accelerator and a Mixer operation is running simultaneously, the
Accelerator operation's remaining cycle count must not include the Mixer
operation's cycle count (which stalls the Accelerator).


0x060; 0x068: FIFOs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

FIFO operations should be executed before modifying the state data
accordingly. The cycle requirements should be calculated as necessary (also
including the operation of the Accelerator or Mixer if triggered), and filled
in the remaining cycle count registers. Then in the same "atomic" operation
the FIFO's read pointer should be incremented.


0x070, Expected device types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This area is populated by the types of devices encountered at each device ID,
as returned by the 0x0410 "Get device properties" kernel call. The return
value is stored as-is on these fields (see "kcall.rst" for details). The
0x0411 "Drop device" kernel call may clear these fields. Using this
information the host may manage device hotplugging better, and allocate
devices better on reloading a saved state. See "Hotplug support" in
"inputdev.rst" for details.


0x200, Kernel tasks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Up to 16 simultaneously executing kernel tasks are supported whose states are
saved on these locations, each kernel task having a 16 word data block in this
range.

The first 15 words of each kernel task provide the parameters with which the
task was started (these are the parameters of the supervisor call which
started the task). The first of these is the kernel call identifier.

The last word is the task status as readable by the 0x0800 "Query task" kernel
function.

When restoring a state having an incomplete kernel task, the task should be
restarted. This normally shouldn't affect the application (except if it
attempts to rely on an undefined behavior described in the "Kernel tasks"
chapter of "kcall.rst").
