
RRPGE User Library - Low Level Display List management and Double Buffering
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


Low level display list and double buffering covers basic common tasks related
to the Graphics Display Generator, such as:

- Automated double buffering management
- Clearing and filling display lists

The double buffering management also requires some data resources located in
the CPU RAM.




Display list offset calculation
------------------------------------------------------------------------------


Functions in this group help in converting and sanitizing display list offsets
between PRAM word offsets and Display List Definition register format offsets.
See the "Display list definition & Process flags" register (0x0017) in
"vid_arch.rst" for details on this format.


0xE030: Convert from PRAM word offset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dloff_from
- Cycles: 100
- Param0: Display list word offset in PRAM, high
- Param1: Display list word offset in PRAM, low
- Param2: Display list size (low 2 bits used only)
- Ret.X3: Offset in Display List Definition format (size included)

The Display list size parameter specifies display list sizes the same way like
the low 2 bits of the Display list definition & Process flags register.

Low bits of offset in the return are clipped according to the way the Graphics
Display Generator ignores those bits depending on display list size.


0xE032: Convert to PRAM word offset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dloff_to
- Cycles: 100
- Param0: Offset in Display List Definition format (size included and used)
- Ret. C: Display list word offset in PRAM, high
- Ret.X3: Display list word offset in PRAM, low

Low bits of the input offset are clipped first according to the way the
Graphics Display Generator ignores those bits depending on display list size.


0xE040: Sanitize display list offset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dloff_clip
- Cycles: 100
- Param0: Offset in Display List Definition format (size included and used)
- Ret.X3: Sanitized offset (size included)

Low bits of the input offset are clipped according to the way the Graphics
Display Generator ignores those bits depending on display list size. High
(non-offset) bits are cleared.




Double buffering management
------------------------------------------------------------------------------


Assists in performing Display list double buffering, also providing hooks for
adding functions to be processed when certain events happen.

This component uses some CPU RAM locations to perform it's functions. These
locations are not meant to be accessed directly by applications.

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFAD0 |                                                                   |
| \-     | Init hooks (functions to call on init)                            |
| 0xFADD |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFADE | Flip performed flag (bit 0 set after a flip, then cleared by      |
|        | processing the frame hooks)                                       |
+--------+-------------------------------------------------------------------+
| 0xFADF | Absolute offset of first free slot in init hooks                  |
+--------+-------------------------------------------------------------------+
| 0xFAE0 |                                                                   |
| \-     | Frame end hooks (functions to call when frame ends)               |
| 0xFAED |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAEE | Absolute offset of first free slot in frame hooks                 |
+--------+-------------------------------------------------------------------+
| 0xFAEF | Absolute offset of first free slot in flip hooks                  |
+--------+-------------------------------------------------------------------+
| 0xFAF0 |                                                                   |
| \-     | Page flip hooks (functions to call when flipping pages)           |
| 0xFAFD |                                                                   |
+--------+-------------------------------------------------------------------+
| 0xFAFE | Current work display list definition & process flags              |
+--------+-------------------------------------------------------------------+
| 0xFAFF | Current displayed display list definition & process flags         |
+--------+-------------------------------------------------------------------+

Note that several locations are initialized to nonzero, see "ulboot.rst" for
details.


Hook functions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This component accepts hooks to be added to some of it's functions. These hook
functions are expected to follow the standard calling convention (described
in "index.rst"). They may affect the state of hardware (such as the pointers),
however then it is the responsibility of the application's designer to plan
out the application's architecture so the users of this component are also
aware of these.


0xE042: Initialize for double buffering
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_init
- Cycles: Waits for frame end
- Param0: Display list definition 1 (first displayed, provides size)
- Param1: Display list definition 2 (first work buffer)
- Param2: Display list clear controls to fill in
- Ret.X3: Current work display list (size and mode flags included)

Sets up display for double buffering, also calling the init hooks. The display
list definitions provided are sanitized with us_dloff_clip.

The display lists themselves are not altered, so they should be set up as
necessary before calling this.

Setting Display list definition 1 for display is treated as a page flip, the
flip hooks are called after it (work and display surfaces however are not
exchanged). Then the function waits for the end of the frame (unless already
reached, using the frame rate limiter flag of the Display List Definition &
Process flags register), sets display list clear controls by the provided
value, finally transferring to us_dbus_getlist to process frame hooks and to
return the work display list.

The display list parameters are written out in the respective CPU RAM
variables after the current mode flags (4 / 8 bit mode, double scan) are added
to them.


0xE050: Flip pages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_flip
- Cycles: Waits for Graphics FIFO draining

First if necessary, it waits for the Graphics FIFO to be drained, so anything
still processing for the current work display list may finish before flipping
it in. Then the pages are flipped, work and display surfaces are exchanged
(0xF990 - 0xF99F in CPU RAM), and the flip hooks are called, also setting the
Flip performed flag (0xFAFD in CPU RAM).

Before starting the above described tasks, it may also call the frame hooks if
calling us_dbuf_getlist or us_dbuf_getsurface was omitted after the last page
flip.

If necessary, the mode flags in the display list CPU RAM variables are updated
according to the currently set display mode.


0xE052: Get work display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_getlist
- Cycles: Waits for frame end (of previous flip), otherwise 25
- Ret.X3: Current work display list (size and mode flags included)

First if necessary, it waits for the frame (in which the pages were last
flipped) to end, also calling the frame hooks when this happens. The wait is
performed by the Frame rate limiter flag (in the Display List Definition &
Process Flags register).

This function is optimized for fast return, simply providing the appropriate
CPU RAM variable. The us_dbuf_init and us_dbuf_flip routines ensure that the
variables have the correct content, and keep being correct.

This function may be called from within a frame end hook. This case it only
returns the current work display list (not attempting to call any frame hook).

This function may be called without double buffering set up, to support
writing double-buffering aware components (which only use it to wait for frame
end when necessary).


0xE060: Add page flip hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_addfliphook
- Cycles: 500
- Param0: Function to add

Adds a function (no parameters, no return) to the page flip hook list. The
hooks are processed in the order they were added. Re-adding a function moves
it to the end of the list.

No effect if the page flip hook list is full.

The list of hooks in CPU RAM grows incrementally (lower locations filled
first).


0xE062: Remove page flip hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_remfliphook
- Cycles: 500
- Param0: Function to remove

Removes a function from the page flip hook list. If it does not exist in the
list, no effect.


0xE064: Add frame end hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_addframehook
- Cycles: 500
- Param0: Function to add

Adds a function (no parameters, no return) to the frame end hook list. The
hooks are processed in the order they were added. Re-adding a function moves
it to the end of the list.

No effect if the frame end hook list is full.

The list of hooks in CPU RAM grows incrementally (lower locations filled
first).


0xE066: Remove frame end hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_remframehook
- Cycles: 500
- Param0: Function to remove

Removes a function from the frame end hook list. If it does not exist in the
list, no effect.


0xE068: Add init hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_addinithook
- Cycles: 500
- Param0: Function to add

Adds a function (no parameters, no return) to the init hook list. The hooks
are processed in the order they were added. Re-adding a function moves it to
the end of the list.

No effect if the init hook list is full.

The list of hooks in CPU RAM grows incrementally (lower locations filled
first).


0xE06A: Remove init hook
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dbuf_reminithook
- Cycles: 500
- Param0: Function to remove

Removes a function from the init hook list. If it does not exist in the list,
no effect.




Basic display list management
------------------------------------------------------------------------------


Provides basic functions for performing various common display list related
operations. They do not rely on the current Display List Definition & Process
Flags register state, rather take it entirely as parameter, so any kind of
display list can be populated with them (useful for example for prefilling
lists to be used after some graphics configuration change). Some of the
functions however use some Graphics Display Definition registers to do their
job, indicated at the descriptions of those.

All functions populating the display list in some manner use the
us_dlist_setptr function to initialize pointers to walk them, so the
definition of this function applies to all.


0xE034: Set up PRAM pointers for list walking
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_setptr
- Cycles: 230
- Param0: Display list column to use
- Param1: Y position to start at (must be either 0 - 199 or 0 - 399)
- Param2: Display List Definition & Process Flags to use
- Ret.X3: Display list line size in bit units (128 / 256 / 512 / 1024 / 2048)

Sets up PRAM pointers 2 and 3 for walking a specific column of the display
list. Pointer 2 is set up to walk (incrementally) the high word of the entry,
Pointer 3 is set up to walk the low word.

The double scan flag in parameter 2 is used to determine the display list's
line size (in addition to the display list line size bits). See the definition
of the Display List Definition & Process flags register (0x0017) in
"vid_arch.rst".

Note that the column and the Y position parameters are not checked in any
manner, values out of range for a given display list produce undefined
results. The display list definition's offset part is sanitized as defined for
us_dloff_clip.


0xE036: Add graphics component to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_add
- Cycles: 430 + 15 / line
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Display List Definition & Process Flags to use
- Param5: Y position to start at (signed 2's complement, can be off-display)

The first source line position is taken from the Render command, subsequent
positions are calculated according to the source selected by the Render
command, using the Source definition registers in the GDG (see registers
0x0018 - 0x001F in "vid_arch.rst").

The source is clipped to the display list's height (either 200 or 400 lines
depending on whether the Double Scan flag in parameter 4 is set or not), first
line's source position adjusted accordingly. The display list column is not
affected if the source falls entirely off-display.

PRAM pointers 2 and 3 are used and not preserved.


0xE038: Add graphics component at X:Y to list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_addxy
- Cycles: 530 + 15 / line
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Display List Definition & Process Flags to use
- Param5: X position to start at (signed 2's complement, can be off-display)
- Param6: Y position to start at (signed 2's complement, can be off-display)

The X position after determining whether the source is on-display at least
partially is used to override the low 10 bits of the Render command low word,
then us_dlist_add is called with the result.

X position respects the 4 / 8 bit mode flag in parameter 4, in 8 bit mode
on-display coordinates ranging from 0 - 319.

Width of the source is calculated according to the selected Source definition
register of the GDG (see registers 0x0018 - 0x001F in "vid_arch.rst"). Note
that if the source is wider than 384 (4 bit) or 192 (8 bit) pixels, it may
partially show on the "wrong" side of the display (this behavior is caused by
the architecture of the Graphics Display Generator).

Shift sources are not supported by this function, the behavior for attempting
to add a shift source with this function is undefined.

PRAM pointers 2 and 3 are used and not preserved.


0xE03A: Add background pattern to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_addbg
- Cycles: 380 + 11 / line
- Param0: Background pattern high word
- Param1: Background pattern low word
- Param2: Height in lines
- Param3: Display List Definition & Process Flags to use
- Param4: Y position to start at (signed 2's complement, can be off-display)

Adds the provided background pattern to Display list column 0.

The source is clipped to the display list's height (either 200 or 400 lines
depending on whether the Double Scan flag in parameter 4 is set or not). The
display list is not affected if the source falls entirely off-display.

PRAM pointers 2 and 3 are used and not preserved.


0xE03C: Add render command list to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_addlist
- Cycles: 500 + 19 / line
- Param0: PRAM word offset of render command list, high
- Param1: PRAM word offset of render command list, low
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Display List Definition & Process Flags to use
- Param5: Y position to start at (signed 2's complement, can be off-display)

The source is clipped to the display list's height (either 200 or 400 lines
depending on whether the Double Scan flag in parameter 4 is set or not), start
offset of the render command list adjusted accordingly. The display list
column is not affected if the source falls entirely off-display.

The render commands in the render command list take 2 words each, and are in
Big Endian order (high word first).

PRAM pointers 1, 2 and 3 are used and not preserved.


0xE03E: Clear display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_clear
- Cycles: 280 + 12 / entry
- Param0: Display List Definition & Process Flags to use

Clears the entire display list to zero. The passed display list definition is
sanitized as defined for us_dloff_clip.

Uses us_set_p for the clear, taking 6 cycles for a word, or 12 cycles for a 32
bit display list entry. Total cycle counts are 19480 / 38680 / 77080 / 153880
cycles depending on display list size.

PRAM pointer 3 is used and not preserved.




Single buffered display list management
------------------------------------------------------------------------------


The functions below are simple wrappers for the Basic display list management
functions, using the current Display List Definition & Process flags register
contents (see register 0x0017 is "vid_arch.rst") for the respective parameter.


0xE044: Set up PRAM pointers for list walking
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_setptr
- Cycles: 250
- Param0: Display list column to use
- Param1: Y position to start at (must be either 0 - 199 or 0 - 399)
- Ret.X3: Display list line size in bit units (128 / 256 / 512 / 1024 / 2048)

Wrapper for us_dlist_setptr using the current Display List Definition &
Process flags register contents.


0xE046: Add graphics component to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_add
- Cycles: 450 + 15 / line
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_add using the current Display List Definition & Process
flags register contents.

PRAM pointers 2 and 3 are used and not preserved.


0xE048: Add graphics component at X:Y to list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_addxy
- Cycles: 550 + 15 / line
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: X position to start at (signed 2's complement, can be off-display)
- Param5: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addxy using the current Display List Definition & Process
flags register contents.

PRAM pointers 2 and 3 are used and not preserved.


0xE04A: Add background pattern to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_addbg
- Cycles: 400 + 11 / line
- Param0: Background pattern high word
- Param1: Background pattern low word
- Param2: Height in lines
- Param3: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addbg using the current Display List Definition & Process
flags register contents.

PRAM pointers 2 and 3 are used and not preserved.


0xE04C: Add render command list to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_addlist
- Cycles: 520 + 19 / line
- Param0: PRAM word offset of render command list, high
- Param1: PRAM word offset of render command list, low
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addlist using the current Display List Definition &
Process flags register contents.

PRAM pointers 1, 2 and 3 are used and not preserved.


0xE04E: Clear display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_sb_clear
- Cycles: 300 + 12 / entry
- Param0: Display List Definition & Process Flags to use

Wrapper for us_dlist_clear using the current Display List Definition & Process
flags register contents.

PRAM pointer 3 is used and not preserved.




Double buffered display list management
------------------------------------------------------------------------------


The functions below are simple wrappers for the Basic display list management
functions, using the return value of us_dbuf_getlist for the display list
definition & process flags parameter.

Due to the use of us_dbuf_getlist, the functions might stall if the frame of
the page flip was not completed yet.


0xE054: Set up PRAM pointers for list walking
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_setptr
- Cycles: 270 + Wait for frame end
- Param0: Display list column to use
- Param1: Y position to start at (must be either 0 - 199 or 0 - 399)
- Ret.X3: Display list line size in bit units (128 / 256 / 512 / 1024 / 2048)

Wrapper for us_dlist_setptr using the return of us_dbuf_getlist for display
list definition & process flags.


0xE056: Add graphics component to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_add
- Cycles: 470 + 15 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_add using the return of us_dbuf_getlist for display list
definition & process flags.

PRAM pointers 2 and 3 are used and not preserved.


0xE058: Add graphics component at X:Y to list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_addxy
- Cycles: 570 + 15 / line + Wait for frame end
- Param0: Render command high word
- Param1: Render command low word
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: X position to start at (signed 2's complement, can be off-display)
- Param5: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addxy using the return of us_dbuf_getlist for display
list definition & process flags.

PRAM pointers 2 and 3 are used and not preserved.


0xE05A: Add background pattern to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_addbg
- Cycles: 420 + 11 / line + Wait for frame end
- Param0: Background pattern high word
- Param1: Background pattern low word
- Param2: Height in lines
- Param3: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addbg using the return of us_dbuf_getlist for display
list definition & process flags.

PRAM pointers 2 and 3 are used and not preserved.


0xE05C: Add render command list to display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_addlist
- Cycles: 540 + 19 / line + Wait for frame end
- Param0: PRAM word offset of render command list, high
- Param1: PRAM word offset of render command list, low
- Param2: Height in lines
- Param3: Display list column to add to
- Param4: Y position to start at (signed 2's complement, can be off-display)

Wrapper for us_dlist_addlist using the return of us_dbuf_getlist for display
list definition & process flags.

PRAM pointers 1, 2 and 3 are used and not preserved.


0xE05E: Clear display list
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dlist_db_clear
- Cycles: 320 + 12 / entry + Wait for frame end
- Param0: Display List Definition & Process Flags to use

Wrapper for us_dlist_clear using the return of us_dbuf_getlist for display
list definition & process flags.

Note that on a double buffered layout using an appropriate Display List Clear
is much more effective (see us_dbuf_init, and "Display list clear function"
in "vid_arch.rst").

PRAM pointer 3 is used and not preserved.




Entry point table of Display List management & Double Buffering functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- U: Cycles taken for processing one unit of data.
- W: May wait for a specific event.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE030 |           100 | 3 |  X3  | us_dloff_from                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE032 |           100 | 1 | C:X3 | us_dloff_to                            |
+--------+---------------+---+------+----------------------------------------+
| 0xE034 |           230 | 3 |  X3  | us_dlist_setptr                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE036 |     15U + 430 | 6 |      | us_dlist_add                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE038 |     15U + 530 | 7 |      | us_dlist_addxy                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE03A |     11U + 380 | 5 |      | us_dlist_addbg                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE03C |     19U + 500 | 6 |      | us_dlist_addlist                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE03E |     12U + 280 | 1 |      | us_dlist_clear                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE040 |           100 | 1 |  X3  | us_dloff_clip                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE042 |             W | 3 |  X3  | us_dbuf_init                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE044 |           250 | 2 |  X3  | us_dlist_sb_setptr                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE046 |     15U + 450 | 5 |      | us_dlist_sb_add                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE048 |     15U + 550 | 6 |      | us_dlist_sb_addxy                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE04A |     11U + 400 | 4 |      | us_dlist_sb_addbg                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE04C |     19U + 520 | 5 |      | us_dlist_sb_addlist                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE04E |     12U + 300 | 0 |      | us_dlist_sb_clear                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE050 |             W | 0 |      | us_dbuf_flip                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE052 |             W | 0 |  X3  | us_dbuf_getlist                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE054 |       270 + W | 2 |  X3  | us_dlist_db_setptr                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE056 | 15U + 470 + W | 5 |      | us_dlist_db_add                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE058 | 15U + 570 + W | 6 |      | us_dlist_db_addxy                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE05A | 11U + 420 + W | 4 |      | us_dlist_db_addbg                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE05C | 19U + 540 + W | 5 |      | us_dlist_db_addlist                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE05E | 12U + 320 + W | 0 |      | us_dlist_db_clear                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE060 |           500 | 1 |      | us_dbuf_addfliphook                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE062 |           500 | 1 |      | us_dbuf_remfliphook                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE064 |           500 | 1 |      | us_dbuf_addframehook                   |
+--------+---------------+---+------+----------------------------------------+
| 0xE066 |           500 | 1 |      | us_dbuf_remframehook                   |
+--------+---------------+---+------+----------------------------------------+
| 0xE068 |           500 | 1 |      | us_dbuf_addinithook                    |
+--------+---------------+---+------+----------------------------------------+
| 0xE06A |           500 | 1 |      | us_dbuf_reminithook                    |
+--------+---------------+---+------+----------------------------------------+
