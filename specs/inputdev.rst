
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


The RRPGE system is capable to use the following types of input devices:

- Pointing devices. Two major types are supported: Mice or similar devices
  which require an on-screen cursor for feeding back position information to
  the user and typically have multiple buttons, and Touch devices which
  operate on the concept of the user interacting with the display itself or a
  surface representing the display, not always requiring an on-screen cursor
  but typically having only a single press information ("button").

- Digital gamepads. These typically provide four buttons for directions, and
  typically four or more additional buttons. They have no capability for
  producing analog inputs. The host may use a physical keyboard to simulate
  one, however on a keyboardless device it may be difficult to emulate.

- Analog joysticks. A generic analog device usually meant to be used for
  flight simulators and similar applications.

- Text input. This device is meant to provide a stream of characters and
  optionally control codes, not necessarily from a physical keyboard.

- Keyboards. The keyboard is supported for those situations where the text
  input feature is not adequate since the application's requirement is rather
  a larger set of buttons in a deterministic layout.




Multiple devices, combinations
------------------------------------------------------------------------------


While the most simple applications may work well with a single type of input
device, some may find it beneficial to work with multiple devices either for
easing the use (such as by a keyboard and a mouse), or for providing better
support for a wider number of devices (such as supporting both mice and touch
devices).

When multiple devices are present, the host may decide to provide multiple
logical presentations for a single physical device. This condition can be
detected using the 0x0410 "Get device properties" kernel call, on bits 0-4 of
the return value. If a valid target device is provided here, it indicates that
device representing better the actual physical device.

This feature may be used to provide very different abstractions to the same
physical device facilitating the use from applications. A typical example
would be a text input device and a keyboard, both supported by a physical
keyboard.




Hotplug support
------------------------------------------------------------------------------


Devices may be plugged in and out in an RRPGE application's lifetime
(especially indirectly through an application state save and later reload).
The system is capable to support this adequately, even without the awareness
of the application.

When the 0x10 "Get device properties" kernel call is called, the kernel also
populates the appropriate field in the application state (see "state.rst")
with the return value. This indicates the type of the device at the given
device ID (or the fact that the device is absent). The 0x0411 "Drop device"
kernel call may be used to drop out devices from this state indicating they
are no longer used.

When an used device is removed, the application might still be expecting it to
function. A removed device however returns complete inactivity (just as a
nonexistent device does) which is the intended behavior.


Host side
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When a device is plugged in, the host should normally check the application
state (0x070 - 0x07F in the state), to see if there is a device ID where the
application expects no device, and add the new device there. If the host is
capable to identify that the device added matches (at least by type) one
previously removed, it may reuse the device ID.

When restoring a state, the host should repopulate the device ID's by simple
type matching (as it has no information on the application's previous device
layout).

If during adding devices, the device ID's are exhausted, the host should
restrain from adding new devices until a device ID frees up naturally (the
application polls with the 0x0410 "Get device properties" kernel call, so
earlier removed devices may be dropped out).

To always utilize the devices the best way even across state saves and
loads, the host should allocate the device ID's incrementally, with the "best"
devices first as far as possible.


Application side
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If an application needs an input device of a particular type, it should use
the device with the lowest ID which matches (it should be the best).

To make an application hotplug aware, it simply needs to call the 0x10 "Get
device properties" kernel call regularly for the devices it uses. This way it
can detect the absence of a previously used device, and may act accordingly.

It is not strictly required, but beneficial to also call the 0x0411 "Drop
device" kernel call on any device the application does not want to use. By
this the host can get a more exact image on what the application actually
uses, so may manage the devices better especially across state saves and
reloads.




The description of device types
------------------------------------------------------------------------------


Here each of the supported input devices are described with their digital and
analog input mappings.


0x0: Mouse pointing device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The mouse pointing device provides a normal computer mouse requiring a cursor
presented for the user to assist tracking it's position. The "0x19: Return
area activity" kernel call should be activated by any button click with the
pointer on the queried area.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
|      | 0     | Scroll up (if there is any button in this role)             |
| 0    +-------+-------------------------------------------------------------+
|      | 1     | Scroll right (if there is any button in this role)          |
|      +-------+-------------------------------------------------------------+
|      | 2     | Scroll down (if there is any button in this role)           |
|      +-------+-------------------------------------------------------------+
|      | 3     | Scroll left (if there is any button in this role)           |
|      +-------+-------------------------------------------------------------+
|      | 4     | Primary (left) mouse button                                 |
|      +-------+-------------------------------------------------------------+
|      | 5     | Secondary (right) mouse button (if any)                     |
|      +-------+-------------------------------------------------------------+
|      | 6     | Middle mouse button (if any)                                |
|      +-------+-------------------------------------------------------------+
|      | 7-15  | Additional mouse buttons (if any)                           |
+------+-------+-------------------------------------------------------------+

Analog input mapping:

+-------+--------------------------------------------------------------------+
| Input | Description                                                        |
+=======+====================================================================+
| 0     | Position X (0-639, even in 8 bit mode)                             |
+-------+--------------------------------------------------------------------+
| 1     | Position Y (0-399)                                                 |
+-------+--------------------------------------------------------------------+
| 2     | Scroll wheel X (infinite, wrapping)                                |
+-------+--------------------------------------------------------------------+
| 3     | Scroll wheel Y (infinite, wrapping)                                |
+-------+--------------------------------------------------------------------+

The scroll wheel inputs represent distance traveled compared to a 0 point
sampled on the device's initialization. Negative values should relate to
scrolling up (Y) or left (X). On a typical mouse Scroll wheel Y is available,
and there are no scroll buttons. On some mice a horizontal scroll wheel, or
buttons associated with left / right scroll are available.

Device specific flags (returned by 0x10: Get device properties):

- bit 5: Set if a cursor should be displayed to track the device.


0x1: Touch pointing device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The touch pointing device assumes a touch display or surface representing the
display. The device may support multi-touch which may be exploited through the
"0x19: Return area activity" kernel call.

Hover activities may be returned if the physical device supports it. These
indicate that the user did not actually press, but the respective analog
inputs are valid.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
|      | 4     | Primary touch press activity                                |
| 0    +-------+-------------------------------------------------------------+
|      | 5     | Secondary touch press activity (if supported)               |
|      +-------+-------------------------------------------------------------+
|      | 12    | Primary touch hover activity (if supported)                 |
|      +-------+-------------------------------------------------------------+
|      | 13    | Secondary touch hover activity (if supported)               |
+------+-------+-------------------------------------------------------------+

Other inputs may be available if the device has additional buttons.

Analog input mapping:

+-------+--------------------------------------------------------------------+
| Input | Description                                                        |
+=======+====================================================================+
| 0     | Primary touch last position X (0-639, even in 8 bit mode)          |
+-------+--------------------------------------------------------------------+
| 1     | Primary touch last position Y (0-399)                              |
+-------+--------------------------------------------------------------------+
| 2     | Secondary touch last position X (0-639, even in 8 bit mode)        |
+-------+--------------------------------------------------------------------+
| 3     | Secondary touch last position Y (0-399)                            |
+-------+--------------------------------------------------------------------+
| 4     | Primary touch pressure (0-0xFFFF)                                  |
+-------+--------------------------------------------------------------------+
| 5     | Secondary touch pressure (0-0xFFFF)                                |
+-------+--------------------------------------------------------------------+

Pressure information should be zero if there is no touch activity.

Device specific flags (returned by 0x10: Get device properties):

- bit 5: Set if a cursor should be displayed to track the device.


0x2: Digital gamepad
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usual digital gamepad with a direction pad and a set of buttons.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
| 0    | 0     | Direction up                                                |
+------+-------+-------------------------------------------------------------+
| 0    | 1     | Direction right                                             |
+------+-------+-------------------------------------------------------------+
| 0    | 2     | Direction down                                              |
+------+-------+-------------------------------------------------------------+
| 0    | 3     | Direction left                                              |
+------+-------+-------------------------------------------------------------+
| 0    | 4     | Primary action button                                       |
+------+-------+-------------------------------------------------------------+
| 0    | 5     | Secondary action button (if any)                            |
+------+-------+-------------------------------------------------------------+
| 0    | 6     | Additional button (if any; "Menu" if possible)              |
+------+-------+-------------------------------------------------------------+
| 0    | 7-15  | Additional buttons (if any)                                 |
+------+-------+-------------------------------------------------------------+


0x3: Analog joystick
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The usual at least 2 axis plus at least one fire button analog stick.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
|      | 0     | Hat/POV switch up (if any)                                  |
| 0    +-------+-------------------------------------------------------------+
|      | 1     | Hat/POV switch right (if any)                               |
|      +-------+-------------------------------------------------------------+
|      | 2     | Hat/POV switch down (if any)                                |
|      +-------+-------------------------------------------------------------+
|      | 3     | Hat/POV switch left (if any)                                |
|      +-------+-------------------------------------------------------------+
|      | 4     | Primary (left) action button                                |
|      +-------+-------------------------------------------------------------+
|      | 5     | Secondary (right) action button (if any)                    |
|      +-------+-------------------------------------------------------------+
|      | 6     | Additional button (if any; "Menu" if possible)              |
|      +-------+-------------------------------------------------------------+
|      | 7-15  | Additional buttons (if any)                                 |
+------+-------+-------------------------------------------------------------+

Analog input mapping:

+-------+--------------------------------------------------------------------+
| Input | Description                                                        |
+=======+====================================================================+
| 0     | Position X (-0x8000 - 0x7FFF)                                      |
+-------+--------------------------------------------------------------------+
| 1     | Position Y (-0x8000 - 0x7FFF)                                      |
+-------+--------------------------------------------------------------------+
| 2     | Position Z (-0x8000 - 0x7FFF; usually twisting the stick)          |
+-------+--------------------------------------------------------------------+
| 3     | Throttle controller (-0x8000 - 0x7FFF)                             |
+-------+--------------------------------------------------------------------+


0x4: Text input
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The text input device is special in that it is accessible through a separate
kernel call (0x18: "Pop text input FIFO"). It provides no digital or analog
inputs. It may typically be backed by a keyboard, but other physical devices
might be possible.

More on this device can be found in the "Text input control codes" chapter.


0x5: Keyboard
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The keyboard device is provided as a large array of buttons for application
requiring such an input device. Note that for text input, the Text input
device is more suitable.

The descriptions for the digital inputs should be applied by the standard US
QWERTY layout as below (only the alphanumeric portion shown): ::

    +----------------------------------------------------------------...
    | +---+   +---+---+---+---+ +---+---+---+---+ +---+---+---+---+
    | |ESC|   | F1| F2| F3| F4| | F5| F6| F7| F8| | F9|F10|F11|F12|
    | +---+   +---+---+---+---+ +---+---+---+---+ +---+---+---+---+
    | +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
    | | ~ | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 | - | + | | |BKS|
    | +---+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+---+
    | | TAB | Q | W | E | R | T | Y | U | I | O | P | { | } |     |
    | +-----++--++--++--++--++--++--++--++--++--++--++--++--+ENTER|
    | | CAPS | A | S | D | F | G | H | J | K | L | : | " |        |
    | +------+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+--------+
    | | SHIFT  | Z | X | C | V | B | N | M | < | > | ? |  SHIFT   |
    | +----+---++--+-+-+---+---+---+---+---+--++---+---+-----+----+
    | |CTRL|    |ALT |         SPACE          |ALTG|         |CTRL|
    | +----+    +----+------------------------+----+         +----+
    +----------------------------------------------------------------...

If necessary, the actual labeling of the keys may be requestable using the
0x12 "Get digital input description symbols" kernel call.

The first input bank is a combined button state, provided for easing some
typical keyboard uses, and to make it possible to support these uses with
touch in touch aware applications.

Digital input mapping of bank zero:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
|      | 0     | Direction key up; Numpad 8; key 8                           |
| 0    +-------+-------------------------------------------------------------+
|      | 1     | Direction key right; Numpad 6; key 6                        |
|      +-------+-------------------------------------------------------------+
|      | 2     | Direction key down; Numpad 2; key 2                         |
|      +-------+-------------------------------------------------------------+
|      | 3     | Direction key left; Numpad 4; key 4                         |
|      +-------+-------------------------------------------------------------+
|      | 4     | SPACE; ENTER; Numpad Enter                                  |
|      +-------+-------------------------------------------------------------+
|      | 5     | ALT; ALTG; Numpad 0; key 0; Insert                          |
|      +-------+-------------------------------------------------------------+
|      | 6     | ESC; Numpad Del; Delete (+ Optionally "menu" if available)  |
|      +-------+-------------------------------------------------------------+
|      | 7     | F1; Numpad 5; key 5                                         |
|      +-------+-------------------------------------------------------------+
|      | 8     | Numpad 9, key 9, Page Up                                    |
|      +-------+-------------------------------------------------------------+
|      | 9     | Numpad 3, key 3, Page Down                                  |
|      +-------+-------------------------------------------------------------+
|      | 10    | Numpad 1, key 1, End                                        |
|      +-------+-------------------------------------------------------------+
|      | 11    | Numpad 7, key 7, Home                                       |
|      +-------+-------------------------------------------------------------+
|      | 12    | Numpad /                                                    |
|      +-------+-------------------------------------------------------------+
|      | 13    | Numpad *                                                    |
|      +-------+-------------------------------------------------------------+
|      | 14    | Numpad -                                                    |
|      +-------+-------------------------------------------------------------+
|      | 15    | Numpad +                                                    |
+------+-------+-------------------------------------------------------------+

The mapping of the individual keys are shown on the following tables. Empty
indicates unused slots. If the keyboard does not contain a numeric pad, but a
switch, then the switch should be interpreted by the host and keys should be
returned accordingly. Notes (#x) in the table are described below it.

+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
|Bnk|  Area  | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |10 |11 |12 |13 |14 |15 |
+===+========+===+===+===+===+===+===+===+===+===+===+===+===+===+===+===+===+
| 1 | Numpad | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |ENT|Del| / | * | - | + |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 2 | F-Row  |ESC| F1| F2| F3| F4| F5| F6| F7| F8| F9|F10|F11|F12| #0        |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 3 | NumRow | ~ | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 | - | + | | |BKS|   |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 4 | UpRow  |TAB| Q | W | E | R | T | Y | U | I | O | P | { | } |           |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 5 | HomeRow|#1 | A | S | D | F | G | H | J | K | L | : | " |#2 |ENT|       |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 6 | BotRow |SHL|#3 | Z | X | C | V | B | N | M | < | > | ? |#3 |SHR|       |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 7 | Control|CTL|#4 |ALT|SPC|ALG| #4    |CTR|#5 |                           |
+---+--------+---+---+---+---+---+---+---+---+---+---+-----------------------+
| 8 | Dirs   |Up |Rig|Dwn|Lft|Ins|Del|Hom|End|PgU|PgD|                       |
+---+--------+---+---+---+---+---+---+---+---+---+---+-----------------------+
| 9 | Extra  | #6                                                            |
+---+--------+---------------------------------------------------------------+

- #0: If the host supports returning presses for the Print Screen, Scroll Lock
  and Break keys, they may be provided here.

- #1: If the host supports returning presses for the Caps Lock key, it may be
  returned here.

- #2: Place for an extra key in the Home row if any.

- #3: Places for extra keys in the Bottom row if any.

- #4: If the host supports returning presses for the menu keys, they may be
  returned here.

- #5: If the host supports returning presses for the Num Lock key, it may be
  returned here.

- #6: If the keyboard contains additional keys to those defined, they may be
  implemented in this area.

On banks 17 - 25 a similar map must be made available, but mapping by symbol
correspondence (so for example a QWERTZ keyboard's 'Z' would produce an
activity on bank 4, bit 6, and bank 22, bit 2). If the host is not capable to
support symbol correspondence, it is allowed to replicate the same mapping
like used for banks 1 - 9.




Get digital / analog input descriptor
------------------------------------------------------------------------------


The kernel functions 0x12 and 0x13 ("Get digital input descriptor" and "Get
analog input descriptor") can be used to gather information about the controls
provided by a device.

The purpose of these functions are twofold:

- They can return whether the input is available or not: the application may
  use this information to fine-tune it's controls, such as by not expecting
  input from a non-existent point.

- The textual representations may provide feedback for the user (if printed by
  the application), so the user may easier find the appropriate buttons on his
  device.




Text input control codes
------------------------------------------------------------------------------


The kernel function 0x18 "Pop text input FIFO" returns the next character or
control code in the text input buffer if any.

Normally the input is an UTF-32 character, however special control codes also
need to be supplied to serve for text editing.

Note that the text input device is not necessarily a keyboard.

The host may or may not provide control codes to position a text cursor.
Initially applications which want to handle a text cursor should assume the
cursor is after the last received character. Applications which do not want to
realize a text cursor may simply discard cursor control codes if any arrives.
Unsupported characters or control codes may always be simply discarded by
applications.

Following the special codes are listed:

+--------------+-------------------------------------------------------------+
| Code (32bit) | Description                                                 |
+==============+=============================================================+
| 0x00000000   | Text input FIFO is empty                                    |
+--------------+-------------------------------------------------------------+
| 0x00000008   | Backspace: Delete character before text cursor              |
+--------------+-------------------------------------------------------------+
| 0x00000009   | TAB: May produce a horizontal tabulation                    |
+--------------+-------------------------------------------------------------+
| 0x0000000A   | New line                                                    |
+--------------+-------------------------------------------------------------+
| 0x00000020   | Whitespace                                                  |
+--------------+-------------------------------------------------------------+
| 0x0000007F   | Delete: Delete character after the text cursor (if any)     |
+--------------+-------------------------------------------------------------+
| 0x80000090   | Up: Move text cursor up a line                              |
+--------------+-------------------------------------------------------------+
| 0x80000091   | Right: Move text cursor right a character                   |
+--------------+-------------------------------------------------------------+
| 0x80000092   | Down: Move text cursor down a line                          |
+--------------+-------------------------------------------------------------+
| 0x80000093   | Left: Move text cursor left a character                     |
+--------------+-------------------------------------------------------------+
| 0x80000094   | Insert: Toggle insertion mode                               |
+--------------+-------------------------------------------------------------+
| 0x80000096   | Home: Position the text cursor at the beginning of the line |
+--------------+-------------------------------------------------------------+
| 0x80000097   | End: Position the text cursor at the end of the line        |
+--------------+-------------------------------------------------------------+
| 0x80000098   | Page Up: Move text cursor up a page                         |
+--------------+-------------------------------------------------------------+
| 0x80000099   | Page Down: Move text cursor down a page                     |
+--------------+-------------------------------------------------------------+
