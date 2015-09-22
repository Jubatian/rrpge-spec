
RRPGE User Library - Audio buffer management
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction
------------------------------------------------------------------------------


Audio buffer management assists in working with the audio output buffer,
providing functions to track the advancement of playback and set up the Mixer
for output in the buffer.

Note that when using these routines, the audio output registers 0x0004, 0x0005
and 0x0006 should not be written (they are meant to be set up by the
us_aubuf_init function). Register 0x0007 may be written to change playback
rate.

This component uses the following CPU RAM locations to perform its functions:

+--------+-------------------------------------------------------------------+
| Range  | Description                                                       |
+========+===================================================================+
| 0xFDAE | Buffer size as 512 bit << value. At most 11.                      |
+--------+-------------------------------------------------------------------+
| 0xFDAF | Block size within buffer as 512 bit << value.                     |
+--------+-------------------------------------------------------------------+
| 0xFDB0 | Current write offset in buffer in words (not size clipped).       |
+--------+-------------------------------------------------------------------+
| 0xFDB1 | Blocks to keep empty in the buffer.                               |
+--------+-------------------------------------------------------------------+




Functions
------------------------------------------------------------------------------


0xE148: Set up audio buffers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_init
- Cycles: Depends on range to clear (using us_set_p)
- Param0: Left channel buffer start, 16 cell (512 bit) unit.
- Param1: Right channel buffer start, 16 cell (512 bit) unit.
- Param2: Buffer size as 512 bit << Param2.
- Param3: Block size within buffer as 512 bit << Param3.

Param2 and Param3 are limited to at most 11 (the buffers can be at most 32K
cells or 64K samples long).

The simplest setup is using a Block size one less than the Buffer size
parameter, which realizes ordinary double buffering. Smaller blocks sizes
give more protection from skipping over the same audio buffer size since a
longer section of it can be filled in advance, and also more fine-grained
control over the timing of samples (like changing frequency or amplitude). A
small block size however demands more CPU time to work with.

Before setting the buffer start offsets, the buffer is cleared (to silence:
0x8000 sample value) using the CPU. The initial state of the buffer is fully
populated, achieved by setting the Current write offset (0xFDB0) from the
Audio DMA sample counter to point at the block which is currently read by it.

The count of blocks to keep empty is reset to zero.


0xE14A: Set count of blocks to keep empty
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_setecnt
- Cycles: 25
- Param0: Count of blocks to keep empty.

Sets the count of blocks to keep empty in the audio buffer. If it is set zero,
the entire buffer is used, giving the most protection against skipping,
however also producing the most lag. Setting it above zero can order the
us_aubuf_isempty function to return zero if only the given number or fewer
blocks are empty (the Audio DMA sample counter passed them), thus reducting
the effective size of the buffer (reducing audio lag).

It can be set any time, so allowing to dynamically adjust the buffer size as
needed.


0xE14C: Check if the buffer should be filled
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_isempty
- Cycles: 35
- Ret.X3: 0 if the buffer is filled, 1 if there are blocks to fill.

Returns 1 if there is at least one block which can be filled (the Audio DMA
sample counter passed it). It does not mark the block filled.


0xE14E: Signal block completion
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_blockdone
- Cycles: 30

Signals the completion of a single block. Should only be called if
us_aubuf_isempty returned 1 (has no guard against the current write offset
advancing the Audio DMA sample counter).

The completion of Mixer FIFO calls need not be waited before calling this: it
only indicates that by the CPU every task necessary to get the block filled
was carried out, so processing may move on to the next block.


0xE150: Get left block start offset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_getlf
- Cycles: 50
- Ret. C: PRAM bank of block.
- Ret.X3: Block start offset in 32 bit cell units.

Returns 32 bit offset of the current left audio buffer block which should be
written if us_aubuf_isempty returned 1.

In mono setups, either this or us_aubuf_getrt can be called to get the block.


0xE152: Get right block start offset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_getrt
- Cycles: 50
- Ret. C: PRAM bank of block.
- Ret.X3: Block start offset in 32 bit cell units.

Returns 32 bit offset of the current right audio buffer block which should be
written if us_aubuf_isempty returned 1.

In mono setups, either this or us_aubuf_getlf can be called to get the block.


0xE154: Set up mixer for left block
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_mixlf
- Cycles: 90

Sets up the Mixer's destination registers (0x0008, 0x0009, 0x000A) to write
into the current left audio buffer block by writing the respective commands
in the Mixer FIFO. Uses Destination overwrite mode (which the Mixer will
cancel after the first operation on the block).

In mono setups, either this or us_aubuf_mixrt can be called to set up the
mixer for writing the block.


0xE156: Set up mixer for right block
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- F.name: us_aubuf_mixrt
- Cycles: 90

Sets up the Mixer's destination registers (0x0008, 0x0009, 0x000A) to write
into the current right audio buffer block by writing the respective commands
in the Mixer FIFO. Uses Destination overwrite mode (which the Mixer will
cancel after the first operation on the block).

In mono setups, either this or us_aubuf_mixlf can be called to set up the
mixer for writing the block.




Audio buffer management functions
------------------------------------------------------------------------------


The abbreviations used in the table are as follows:

- P: Count of parameters.
- R: Return value registers used.
- S: For cycle counts see function's description.

The cycle counts are to be interpreted with function entry / exit overhead
included, and are maximal counts.

+--------+---------------+---+------+----------------------------------------+
| Addr.  | Cycles        | P |   R  | Name                                   |
+========+===============+===+======+========================================+
| 0xE148 |             S | 4 |      | us_aubuf_init                          |
+--------+---------------+---+------+----------------------------------------+
| 0xE14A |            25 | 1 |      | us_aubuf_setecnt                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE14C |            35 | 0 |  X3  | us_aubuf_isempty                       |
+--------+---------------+---+------+----------------------------------------+
| 0xE14E |            30 | 0 |      | us_aubuf_blockdone                     |
+--------+---------------+---+------+----------------------------------------+
| 0xE150 |            50 | 0 | C:X3 | us_aubuf_getlf                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE152 |            50 | 0 | C:X3 | us_aubuf_getrt                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE154 |            90 | 0 |      | us_aubuf_mixlf                         |
+--------+---------------+---+------+----------------------------------------+
| 0xE156 |            90 | 0 |      | us_aubuf_mixrt                         |
+--------+---------------+---+------+----------------------------------------+
