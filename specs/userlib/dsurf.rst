
RRPGE User Library - Destination surfaces
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


A destination surface structure (object) with associated methods primarily for
using with the Graphics Accelerator.

The structure is formed as follows:

- Word0: Peripheral RAM Write mask, high (for accelerator register 0x0000)
- Word1: Peripheral RAM Write mask, low (for accelerator register 0x0001)
- Word2: Partition size (for accelerator register 0x0002)
- Word3: Surface A bank select (for accelerator register 0x0002)
- Word4: Surface B bank select (for accelerator register 0x0002)
- Word5: Surface A partition select (for accelerator register 0x0003)
- Word6: Surface B partition select (for accelerator register 0x0003)
- Word7: Width of surface in cells (for accelerator register 0x0004)

All of these except Partition size are used as-is, submitted to the
Accelerator as they are. From Partition size only bits 12-15 have effect
(setting the destination partition size), OR combined with the bank select.

Some CPU RAM locations are used as well:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFDBF | Double buffer flipflop. Only bit 0 is used. If clear, Surface A   |
|        | is assumed to be the display surface in surface structures.       |
+--------+-------------------------------------------------------------------+
| 0xFDC0 |                                                                   |
| \-     | Default surface structure (up_dsurf).                             |
| 0xFDC7 |                                                                   |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xE094: Set up surface
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_new
- Cycles: 100
- Param0: Target surface pointer (8 words)
- Param1: PRAM Bank (64K cell unit)
- Param2: PRAM Partition select
- Param3: Width of surface in cells
- Param4: Partition size (0 - 15 for 2 to 64K cells)

Sets up a destination surface structure. The Peripheral RAM write masks are
all set (so it is written normally to all bits). Surface A and B are set up to
be identical, suitable for single buffered operation.


0xE096: Set up surface pair
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_newdbuf
- Cycles: 120
- Param0: Target surface pointer (8 words)
- Param1: Surface A PRAM Bank (64K cell unit)
- Param2: Surface A PRAM Partition select
- Param3: Surface B PRAM Bank (64K cell unit)
- Param4: Surface B PRAM Partition select
- Param5: Width of surface in cells
- Param6: Partition size (0 - 15 for 2 to 64K cells)

Sets up a destination surface structure. The Peripheral RAM write masks are
all set (so it is written normally to all bits).


0xE098: Set up surface with mask
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_newm
- Cycles: 120
- Param0: Target surface pointer (8 words)
- Param1: PRAM Write mask, high
- Param2: PRAM Write mask, low
- Param3: PRAM Bank (64K cell unit)
- Param4: PRAM Partition select
- Param5: Width of surface in cells
- Param6: Partition size (0 - 15 for 2 to 64K cells)

Sets up a destination surface structure. Surface A and B are set up to be
identical, suitable for single buffered operation.


0xE09A: Set up surface pair with mask
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_newmdbuf
- Cycles: 130
- Param0: Target surface pointer (8 words)
- Param1: PRAM Write mask, high
- Param2: PRAM Write mask, low
- Param3: Surface A PRAM Bank (64K cell unit)
- Param4: Surface A PRAM Partition select
- Param5: Surface B PRAM Bank (64K cell unit)
- Param6: Surface B PRAM Partition select
- Param7: Width of surface in cells
- Param8: Partition size (0 - 15 for 2 to 64K cells)

Sets up a destination surface structure, explicitly initializing all members.


0xE09C: Get surface pointer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_get
- Cycles: 80 + Wait for frame end
- Param0: Source surface pointer (8 words)
- Ret. C: Bank of work surface
- Ret.X3: Partition select of work surface

Retrieves pointer to the current work surface of a given surface structure. It
waits for frame end as needed if called in double buffered mode (by
us_dbuf_getlist), otherwise it returns immediately.


0xE09E: Get surface pointer and fill accelerator
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_getacc
- Cycles: 170 + Wait for frame end
- Param0: Source surface pointer (8 words)
- Ret. C: Bank of work surface
- Ret.X3: Partition select of work surface

Fills up Accelerator with destination surface parameters and retrieves pointer
to the current work surface of a given surface structure. It waits for frame
end as needed if called in double buffered mode (by us_dbuf_getlist),
otherwise it returns immediately.

Accelerator registers 0x0000 - 0x0005 are filled by this function. The work
surface is used for setting up the bank and partition selects of the
destination.


0xE0A0: Get width and partitioning settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_getpw
- Cycles: 50
- Param0: Source surface pointer (8 words)
- Ret. C: Partitioning setting (0 - 15)
- Ret.X3: Width of surface in cells

Returns the width in cell and the partitioning setting of the surface,
reflecting the physical width and height of it.


0xE0A2: Initialize surface manager
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_init
- Cycles: 20

Initializes surface manager so Surface B will be used as display surface until
the next flip.

This function may be used as an initialization hook in the Double Buffering
Manager to assist in initializing double buffered constructs.

Note that the Double Buffering Manager's initialization also calls the flip
hooks, so after it's return, Surface A will be the display surface, so
Surface A should be paired with the Display List Definition 1 parameter of
0xE042: "Initialize for double buffering".


0xE0A4: Flip surfaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_dsurf_flip
- Cycles: 25

Flips work and display surfaces, so subsequent us_dsurf_get calls will return
the other surface as work surface.

This function may be used as a flip hook in the Double Buffering Manager to
provide automatic double buffering support.




Entry point table of Destination surface functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- W: May wait for a specific event.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE094 |           100 | 5 |      | us_dsurf_new                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE096 |           120 | 7 |      | us_dsurf_newdbuf                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE098 |           120 | 7 |      | us_dsurf_newm                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE09A |           130 | 9 |      | us_dsurf_newmdbuf                      |
+--------+---------------+---+------+----------------------------------------+
| 0xE09C |        80 + W | 1 | C:X3 | us_dsurf_get                           |
+--------+---------------+---+------+----------------------------------------+
| 0xE09E |       170 + W | 1 | C:X3 | us_dsurf_getacc                        |
+--------+---------------+---+------+----------------------------------------+
| 0xE0A0 |            50 | 1 | C:X3 | us_dsurf_getpw                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE0A2 |            20 | 0 |      | us_dsurf_init                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE0A4 |            25 | 0 |      | us_dsurf_flip                          |
+--------+---------------+---+------+----------------------------------------+
