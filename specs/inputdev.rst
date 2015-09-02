
RRPGE input device management
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2015, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEvt (temporary version of the RRPGE
            License): see LICENSE.GPLv3 and LICENSE.RRPGEvt in the project
            root.




Introduction, the basic concepts of input
------------------------------------------------------------------------------


The RRPGE input system aims to provide a simple support for most common device
types, providing the application developer a simple interface to work with the
diversity of physical devices it may encounter.

The input peripherals are accessible through kernel functions (see the
"Kernel functions, Input devices (0x10 - 0x1E)" section in "kcall.rst" for
further information).

Moreover the application specifies it's device requirements in it's
Application Header (see locations 0x000A in the Application descriptor at
"Application binary maps" in "bin_rpa.rst" for details), so the host may
select the most appropriate way it may serve the application.

In this part of the specification, the additional details required for
completing the input system are defined, along with acceptable suggested
means of implementation and usages of the system.




Input device type summary
------------------------------------------------------------------------------


The RRPGE system supports the following types of input devices:

- 0x0: Pointing device. Typically a mouse. These devices provide a single
  position information which the application should indicate with a cursor,
  two (or more) buttons, and optionally a scroll wheel.

- 0x1: Touch device. Somewhat similar to pointing devices. No cursor is
  required. If a touch device is present, it may indicate that it is the
  preferred form of input, see the description of this device type.

- 0x2: Digital gamepad. These provide four buttons for directions, and
  typically three or more additional buttons. They have no capability for
  producing analog inputs. The host may use a physical keyboard to simulate
  one.

- 0x3: Analog joystick. A generic analog device usually meant to be used for
  flight simulators and similar applications.

- 0x4: Text input. This device is meant to provide a stream of characters and
  optionally control codes, not necessarily from a physical keyboard.

- 0x5: Keyboard. The keyboard is supported for those situations where the text
  input feature is not adequate since the application's requirement is rather
  a larger set of buttons in a deterministic layout.

A physical RRPGE system may relaized in the following forms:

- Microcomputer. This provides a keyboard (see the keyboard device for its
  layout), which also represents a single digital gamepad or a text input
  device. Applications targeting this may expect these being present. If
  digital gampads are connected to this machine, the first of these would
  (normally) share identifier with the keyboard, mapping to the CTRL (primary
  action button), ALT (secondary action button), ESC (menu button) and the
  directional keys. A pointing device (mouse) is optional.

- Handheld console. This form only provides a digital gamepad.

In addition, RRPGE may be realized by emulation over a mobile phone or tablet
which only provides a touch device. A text input device should only be
provided unless it is realized as an on-screen keyboard which would obscure
the RRPGE system's display (it may still be provided if the display is shrunk
this case, so remains entirely visible).




Supporting the Request device kernel call
------------------------------------------------------------------------------


The "0x10: Request device" kernel call binds physical devices seen by the
kernel to IDs expected by the application. The application must only receive
events from devices it requested.

The application can only request devices by their type, it is up to the kernel
or host to decide which device should it provide upon a call. As a general
rule, a single device must never appear under two identifiers for the
application, that is, for example if the application requests a keyboard, then
a digital gamepad, the latter can not be provided using the keyboard (that is,
it must be a physical gamepad, or maybe another keyboard if a second happens
to be attached to the machine).

Devices should be provided in a consistent manner, that is if two devices of
the same type (or capable to service the given type) are present, they should
always be provided to the application in the same order.

The following special situations should be considered:

- If a keyboard is present, it should be the first device to select when
  serving requests for Digital gamepad, Text input and Keyboard devices. Note
  that if a physical gamepad is also present, it will also be used for input
  by the next condition.

- If a keyboard is present and at least one physical gamepad, by default the
  physical gamepad should be part of the keyboard (as the Microcomputer form
  described in the "Input device type summary" defined).

- If only one physical gamepad is present, and the application requests both a
  keyboard and a digital gampead (or two digital gamepads), it should be
  detached from the keyboard to be able to serve such a request (typically may
  be used for two player games). This detachment should only persist until the
  combination requiring it is dropped by the application (by appropriate Drop
  Device calls).

- If more than one text input capable devices are present, they should map to
  a single text input device by default.

- If more than one pointing devices are present, they should map to a single
  pointing device by default (usually this behaviour is expected on most
  contemporary personal computers).

- If a touch device is present, and it is not explicitly requested by the
  application, it should operate as a pointing device (providing events for
  the single pointing device as described above).

As a general rule unless more than one of a given device type is requested by
the application, all suitable physical devices may provide events for the
bound indentifier. Implementing in this manner offers a more pleasant user
experience since any physical device may be used for control without the need
for additional configuration.




Emulation on touch oriented devices
------------------------------------------------------------------------------


The RRPGE system may be emulated over mobile (smart) phones or tablets, which
are touch oriented devices without a keyboard. Normally these devices should
only provide a touch device for input (and optionally a text input device
unless it would be provided by an on-screen keyboard obscuring the RRPGE
system's display).

Applications which aim to support such hosts in addition to the more
conventional targets with at least digital gamepads (optionally as part of a
keyboard) present should try to allocate a digital gamepad first. This request
should fail on a touch oriented device. Then the allocation of a touch device
should succeed, and the application may go on.

Note that requesting a keyboard is not adequate if the application would
normally require only a digital gamepad: a handheld console may not have one.
Requesting a pointing device is neither adequate since the touch device would
normally operate as one. Requesting a touch device right away should neither
be done as some conventional systems with keyboards and mice might have a
touch capable display while their user usually might rather prefer not using
that.

Applications which aim to be touch-centric should attempt to request a touch
device right away, only falling back to other devices if none is present.

An application which aims to support a minimal pointing or touch device
oriented interface should request a touch device, which if fails, should fall
back to requesting a pointing device (this path indicating it requires a
cursor). Then it may use the events of the device like a single button mouse,
only using X and Y location updates to draw the cursor (if necessary).

An application only requiring a single button should attempt to request both a
pointing device and a digital gamepad, accepting presses and releases with
button ID of 1 from each. This way the application would respond proper to
touches (providing pointing device input), mouse presses and digital gamepad
or keyboard buttons.




The description of device types
------------------------------------------------------------------------------


The following section provides descriptions for each of the device types,
defining the format of their events. Long event messages (see "0x12: Pop input
event queue" kernel call) are provided by defining content for data indices 1
and 2 in addition to data index 0. The specified input devices never provide
messages longer than 3 events. The "Msgt" column of the tables provide the
value of the Event message type field of the event.

If a physical device is uncapable to send some of the described event message
types, it should never send such a type. Message types not marked as
"Optional" or "(O)" in the table should always be provided, if necessary, by
some sort of emulation.


0x0: Pointing device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The pointing device supports a normal computer mouse or similar devices
requiring a cursor presented for the user to assist tracking it's position.

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 0    | Button press           | Button ID    | X location   | Y location   |
+------+------------------------+--------------+--------------+--------------+
| 1    | Button release         | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+
| 2    | X location update      | X location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 3    | Y location update      | Y location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 4    | Scrollwheel action (O) | Amount       |              |              |
+------+------------------------+--------------+--------------+--------------+

The following button codes are defined:

- 1: Left (primary) button
- 2: Right button (Optional)
- 3: Middle button (Optional)

The X and Y locations are absolute coordinates on the display, ranging from
0 - 639 and 0 - 399 respectively.

The scrollwheel action provides a signed 2's complement number indicating the
relative amount of scroll. On a typical mouse a full revolution of the
scrollwheel should correspond to the values 0x7FFF (+32767; downwards; towards
the hand scroll) or 0x8000 (-32768; upwards; away from the hand scroll).


0x1: Touch device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The touch device assumes a touch display, that is, the user interacting with
the display surface itself by touch, not requiring a cursor.

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 0    | Touch press            | Touch ID     | X location   | Y location   |
+------+------------------------+--------------+--------------+--------------+
| 1    | Touch release          | Touch ID     |              |              |
+------+------------------------+--------------+--------------+--------------+
| 2    | X location update 1    | X location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 3    | Y location update 1    | Y location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 2    | X location update 2    | X location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 3    | Y location update 2    | Y location   |              |              |
+------+------------------------+--------------+--------------+--------------+

The Touch press event supports multi-touch by allocating new IDs for
subsequent touches, beginning with 1 for the first touch. The X and Y
locations provided in the message locate the touch. The first two touches (ID
1 and 2) may be tracked by X and Y location update events.

The X and Y location update 1 events may arrive without a touch being present
in case the device supports the detection of hovering.

Before Touch press events of ID 1 and 2, the appropriate X and Y location
update events are sent, so the location of these touches can be identified
without reading the location data in the Touch press event (and more
conveniently supports using the Touch device as a Pointing device).

The X and Y locations are absolute coordinates on the display, ranging from
0 - 639 and 0 - 399 respectively.

Multi-touch support and hovering are not required features, if neither is
present, only touch ID 1 events may be generated, X and Y location update 2
events never arriving, and X and Y location update 1 events only arriving when
touch ID 1 is active.


0x2: Digital gamepad
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usual digital gamepad with a direction pad and a set of buttons.

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 0    | Button press           | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+
| 1    | Button release         | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+

The following button codes are defined:

- 1: Primary action button
- 2: Secondary action button
- 3: Menu button
- 4: Up direction
- 5: Right direction
- 6: Down direction
- 7: Left direction

When a digital gamepad device is requested and is served by a keyboard, the
following keys should map to digital gamepad controls:

- CTRL:  Primary action button
- SPACE: Primary action button
- ALT:   Secondary action button
- ESC:   Menu button
- UP:    Up direction
- RIGHT: Right direction
- DOWN:  Down direction
- LEFT:  Left direction


0x3: Analog joystick
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usual at least two axis plus at least two fire button analog stick.

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 0    | Button press           | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+
| 1    | Button release         | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+
| 2    | X location update      | X location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 3    | Y location update      | Y location   |              |              |
+------+------------------------+--------------+--------------+--------------+
| 4    | Throttle update (O)    | Throttle     |              |              |
+------+------------------------+--------------+--------------+--------------+

The following button codes are defined:

- 1: Primary action button
- 2: Secondary action button
- 3: Menu button (Optional)

The X and Y locations are signed 2's complement numbers ranging from -0x4000
to 0x4000. Positive referst to rightwards and downwards directions.

The throttle is a value ranging from 0x0000 to 0x4000. Positive refers to
increasing, direction depending on the construction of the stick.

Calibration, proper centering should be done by the host. The value of 0x0000
indicating center should be stable, and the stick should be capable to reach
all of the limits.


0x4: Text input
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The text input device is provided for reading UTF text from arbitrary sources,
typically a keyboard (but not limited to it).

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 8    | Control input          | Value        |              |              |
+------+------------------------+--------------+--------------+--------------+
| 9    | UTF32 character input  | Value, high  | Value, low   |              |
+------+------------------------+--------------+--------------+--------------+

The control input provides commands to direct cursor movement and text
editing, while the UTF32 character input provides the characters themselves.

When providing text input from a keyboard, the host should implement key
repeating as needed.

For control input, the same codes are used like for Keyboard, in the range
0x0081 - 0x00AE. SHIFTLOCK, SHIFT, CTRL, ALT and FN are not returned as these
are meaningless for text input. INS, and navigational actions (Directionals,
Page Up / Down, Home, End) are provided in particular.

The following special characters should be provided (and recognized) as UTF32
input:

- 0x00000008: Backspace: Delete character before the cursor
- 0x00000009: TAB: A horizontal TAB character
- 0x0000000A: New line (ENTER, also used for confirm in such contexts)
- 0x00000020: Whitespace
- 0x0000007F: Delete: Delete character after the cursor

When a physical gamepad is used to provide text input events, its buttons map
to the following events:

- Primary action: 0x9 / 0x0000000A; New line or ENTER
- Secondary action: Same effect like FN key on the directionals
- Menu: 0x8 / 0x0094; ESC
- Directionals: Either directionals or Page up / down, Home and End.

When the Secondary action button is held down, the directionals so provide the
following events:

- UP: Page Up (0x8 / 0x0086)
- RIGHT: End (0x8 / 0x0087)
- DOWN: Page Down (0x8 / 0x0088)
- LEFT: Home (0x8 / 0x0089)


0x5: Keyboard
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The keyboard device is provided as a large array of buttons for application
requiring such an input device. Note that for text input, the Text input
device is more suitable.

Associated message types:

+------+------------------------+--------------+--------------+--------------+
| Msgt | Description            | Data index 0 | Data index 1 | Data index 2 |
+======+========================+==============+==============+==============+
| 0    | Button press           | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+
| 1    | Button release         | Button ID    |              |              |
+------+------------------------+--------------+--------------+--------------+

The RRPGE keyboard has the following layout: ::

    +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | + | - | / | * |BKS|
    +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---++---+   +---+
    | TAB | Q | W | E | R | T | Y | U | I | O | P | [ | ] |     ||INS|   |ESC|
    +-----++--++--++--++--++--++--++--++--++--++--++--++--+     |+---+   +---+
    | SHLC | A | S | D | F | G | H | J | K | L | : | ; | ENTER  ||DEL|   |FN |
    +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------++---+---+---+
    | SHIFT  | Z | X | C | V | B | N | M | , | . | = |  SHIFT   |    | A |
    +--------+---+---++--+---+---+---+---+---++--+---+-+--------++---+---+---+
    |  CTRL  |  ALT   |          SPACE        |  ALT   |  CTRL  || < | V | > |
    +--------+--------+-----------------------+--------+--------++---+---+---+

By hardware it is realized as a 8x8 matrix with 63 positions used. The two
ALT, CTRL and SHIFT keys share the same intersection each, so it is not
possible to distinguish which one was pressed of these. Moreover the SHIFT,
CTRL, ALT, SPACE and directional keys share the same row to allow for the
detection of simultaneous presses on keyfoils, so the keyboard can be useful
as a digital gamepad.

The FN key is used to access some alternate functions. When held down, the
upper row becomes function keys (F0 - F14 from key "0" to Backspace), and the
directional keys operate as Page Up, Page Down, Home and End.

The Button IDs are normally formed by providing the ASCII value of the
appropriate keys as above. The following special keys are returned:

- 0x0008: BKS (Backspace)
- 0x0009: TAB
- 0x000A: ENTER
- 0x0020: SPACE
- 0x007F: DEL (Delete)
- 0x0081: INS (Insert)
- 0x0082: UP
- 0x0083: RIGHT
- 0x0084: DOWN
- 0x0085: LEFT
- 0x0086: Page Up (FN + UP)
- 0x0087: End (FN + RIGHT)
- 0x0088: Page Down (FN + DOWN)
- 0x0089: Home (FN + LEFT)
- 0x0090: SHLC (Shift Lock)
- 0x0091: SHIFT (Either left or right)
- 0x0092: CTRL (Either left or right)
- 0x0093: ALT (Either left or right)
- 0x0094: ESC
- 0x0095: FN (Note that its normal function is also performed!)
- 0x00A0: F0 (FN + '0')
- 0x00A1: F1 (FN + '1')
- 0x00A2: F2 (FN + '2')
- 0x00A3: F3 (FN + '3')
- 0x00A4: F4 (FN + '4')
- 0x00A5: F5 (FN + '5')
- 0x00A6: F6 (FN + '6')
- 0x00A7: F7 (FN + '7')
- 0x00A8: F8 (FN + '8')
- 0x00A9: F9 (FN + '9')
- 0x00AA: F10 (FN + '+')
- 0x00AB: F11 (FN + '-')
- 0x00AC: F12 (FN + '/')
- 0x00AD: F13 (FN + '*')
- 0x00AE: F14 (FN + BKS)

When emulating, optionally the native layout of the host's keyboard may also
be used.

When a physical gamepad is used to provide keyboard events, its buttons map to
the following events:

- Primary action: 0x000A; ENTER
- Secondary action: 0x0095: FN (Changes function of directionals)
- Menu: 0x0094; ESC
- Directionals: Either directionals or Page up / down, Home and End.

When the Secondary action button is held down, the directionals so provide the
following events:

- UP: Page Up (0x0086)
- RIGHT: End (0x0087)
- DOWN: Page Down (0x0088)
- LEFT: Home (0x0089)

For more information on the implementation and properties of a physical RRPGE
keyboard, see the hardware details ("impl_hw/keyboard.rst").




Event frequency management
------------------------------------------------------------------------------


The kernel (or host) should ensure that no redundant events arrive during a
display frame wherever this is necessary. If combining events is necessary to
achieve this, it should be done on the application interface (so if multiple
sources are present, those should be sent combined in this manner).

Combining events should proceed by the following principles:

- Event message types 0 and 1 for every device type are press and release
  events. It should be ensured that for any button ID, at most one such event
  (either press or release) arrives. In general if either source's appropriate
  button is pressed, the application should receive events so it interprets it
  as pressed.

- Event message types 2 to 7 for every device type are analog style inputs,
  receiving state changes. Of these the last event should be retained,
  discarding the rest.

Implementing this feature is not critical, however lessens probable load on
the application trying to interpret unnecessarily frequent events.
