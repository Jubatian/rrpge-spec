
RRPGE Read Only Process Descriptor dump
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




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
| \-     | CPU general purpose registers: A, B, C, D, X0, X1, X2, X3.        |
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
| \-     | Unused, must be 0x0000.                                           |
| 0xD4F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD50  | Current video line (2's complement; 0 - 399: Display; Negatives:  |
|        | Vertical blank).                                                  |
+--------+-------------------------------------------------------------------+
| 0xD51  | Cycles until next video line.                                     |
+--------+-------------------------------------------------------------------+
| 0xD52  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0xD53  | Cycles until next audio base clock tick.                          |
+--------+-------------------------------------------------------------------+
| 0xD54  | Cycles remaining from accelerator operation or FIFO fetch, high.  |
+--------+-------------------------------------------------------------------+
| 0xD55  | Cycles remaining from accelerator operation or FIFO fetch, low.   |
+--------+-------------------------------------------------------------------+
| 0xD56  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0xD57  | Current video mode (0: 640x400, 4bit; 1: 320x400, 8bit).          |
+--------+-------------------------------------------------------------------+
| 0xD58  |                                                                   |
| \-     | Unused, must be 0x0000.                                           |
| 0xD5B  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD5C  | Graphics FIFO write pointer.                                      |
+--------+-------------------------------------------------------------------+
| 0xD5D  | Graphics FIFO read pointer.                                       |
+--------+-------------------------------------------------------------------+
| 0xD5E  | Unused, must be 0x0000.                                           |
+--------+-------------------------------------------------------------------+
| 0xD5F  | Graphics FIFO started flag (on bit 0, bits 1-15 are zero).        |
+--------+-------------------------------------------------------------------+
| 0xD60  | Excepted devices at each device ID's. The 0x0410 "Get device      |
| \-     | properties" and the 0x0411 "Drop device" kernel calls manage      |
| 0xD6F  | these fields.                                                     |
+--------+-------------------------------------------------------------------+
| 0xD70  | User peripheral area contents: Graphics FIFO & Graphics Display   |
| \-     | Generator (range: 0xE00 - 0xEFF). See "mem_map.rst" for details.  |
| 0xD7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xD80  |                                                                   |
| \-     | 16 * 16 words of kernel task data.                                |
| 0xE7F  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xE80  | Touch areas, X start (left bound of the area). 0xE80 corresponds  |
| \-     | to the touch area defined for digital input bit 0. Ranges from 0  |
| 0xE8F  | to 639 irrespective of display mode.                              |
+--------+-------------------------------------------------------------------+
| 0xE90  | Touch areas, Y start (top bound of the area). 0xE90 corresponds   |
| \-     | to the touch area defined for digital input bit 0. Ranges from 0  |
| 0xE9F  | to 399 irrespective of display mode.                              |
+--------+-------------------------------------------------------------------+
| 0xEA0  | Touch areas, width. 0xEA0 corresponds to the touch area defined   |
| \-     | for digital input bit 0. Allowed range is so combined with X it   |
| 0xEAF  | fits within the display width of 640 units.                       |
+--------+-------------------------------------------------------------------+
| 0xEB0  | Touch areas, height. 0xEB0 corresponds to the touch area defined  |
| \-     | for digital input bit 0. Allowed range is so combined with Y it   |
| 0xEBF  | fits within the display height of 400 units.                      |
+--------+-------------------------------------------------------------------+
| 0xEC0  | User peripheral area contents: Audio & DMA peripherals (range:    |
| \-     | 0xF00 - 0xFFF). See "mem_map.rst" for details.                    |
| 0xEDF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xEE0  | Graphics registers on the FIFO bus in the 0x000 - 0x0FF range     |
| \-     | (repeating). See "mem_map.rst" for details.                       |
| 0xEFF  |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xF00  | Graphics registers on the FIFO bus in the 0x100 - 0x1FF range     |
| \-     | (reindex table). See "mem_map.rst" for details.                   |
| 0xFFF  |                                                                   |
+--------+-------------------------------------------------------------------+




Usage guides
------------------------------------------------------------------------------


Following some details are provided on how the ROPD dump should be used and
populated within emulators so the desired cross-compatibility may be achieved.


0xD50, Current video line
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Depending on the display standard the emulator follows, the count of VBlank
lines may differ. Up to 250 remaining Vertical blank lines should be
tolerated.


0xD54, Accelerator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After starting an accelerator operation, when exporting state before it's
completion, the emulator should complete the entire operation, and save the
state accordingly. This does not affect applications as they can not access
the Video RAM meanwhile and the Graphics FIFO also waits until the completion
of the operation. It may affect the display if the application performs
operations falling under implementation defined rules (such as performing an
accelerator operation over a display list which is the same time read for
display).

The high part may be used by implementations if they wish to pre-calculate the
number of cycles after which a FIFO beam wait's condition will be fulfilled.
If the beam wait condition can not ever be fulfilled, such implementations are
allowed to either terminate the application or set the maximal possible cycle
count here.


0xD5C, Graphics FIFO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Graphics FIFO operations should be executed before modifying the ROPD data
accordingly. The cycle requirements should be calculated as necessary (also
including the operation of the Accelerator if triggered), and filled in 0xD54
and 0xD55. Then in the same "atomic" operation the Graphics FIFO's state (read
pointer and if needed, the clearing of the started flag) should be updated and
filled in.

Note that the command & data words to store next in the FIFO must be
maintained on the 0xEC6 and 0xEC7 areas respectively. In any loaded state
however the contents of 0xEC7 (the data word) is irrelevant.


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


0xEC0, Last device types
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This area is populated by the types of devices encountered at each device ID,
as returned by the 0x0410 "Get device properties" kernel call. The return
value is stored as-is on these fields (see "kcall.rst" for details). The
0x0411 "Drop device" kernel call may clear these fields. Using this
information the host may manage device hotplugging better, and allocate
devices better on reloading a saved state. See "Hotplug support" in
"inputdev.rst" for details.


0xEC0, DMA operations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An emulator should execute a DMA operation (CPU RAM Copy & Fill DMA, Mixer
DMA, CPU <=> VRAM DMA) as one uninterruptible block, and prepare the state
accordingly.
