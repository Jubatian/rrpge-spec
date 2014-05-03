
RRPGE input device management
==============================================================================

:Author:    Sandor Zsuga (Jubatian)
:Copyright: 2013 - 2014, GNU GPLv3 (version 3 of the GNU General Public
            License) extended as RRPGEv2 (version 2 of the RRPGE License): see
            LICENSE.GPLv3 and LICENSE.RRPGEv2 in the project root.




Introduction, the basic concepts of input
------------------------------------------------------------------------------


The RRPGE input system aims to provide a simple support for most common device
types, providing the application developer a simple interface to work with the
diversity of physical devices it may encounter.

The input peripherals are accessible through kernel functions (see the
"Kernel functions, Input devices (0x0400 - 0x04FF)" section in "kcall.rst" for
further information).

Moreover the application specifies it's device requirements in it's
Application Header (see locations 0xBC1 and 0xBD0 at "Application binary
header map" in "bin_rpa.rst" for details), so the host may select the most
appropriate way it may serve the application.

In this part of the specification, the additional details required for
completing the input system are defined, along with acceptable suggested
means of implementation and usages of the system.




Input device type summary
------------------------------------------------------------------------------


The RRPGE system is capable to use the following types of input devices:

- Pointing devices. Two major types are supported: Mice or similar devices
  which require an on-screen cursor for feeding back position information to
  the user, and typically have multiple buttons, and Touch screens which
  operate on the concept of the user interacting with the display itself, thus
  not requiring an on-screen cursor, but typically having only a single
  "button".

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

In addition to the genres, the application may suggest the following
properties of it's input requirements (using the Application Header's 0xBC1
field):

- The capability to use touch. This indicates that the application is touch -
  aware, that is it will use the appropriate kernel functions to set up touch
  sensitive areas to make it's digital inputs functional even if physical
  digital inputs can not be provided at all by the host. This flag suggests
  the host that it does not need to resort to some awkward emulation to
  provide virtual digital inputs to operate the application.




Multiple devices, combinations
------------------------------------------------------------------------------


While the most simple applications may work well with a single type of input
device, some may find it beneficial to work with multiple devices either for
easing the use (such as by a keyboard and a mouse), or for providing better
support for a wider number of devices (such as supporting both mice and touch
devices).

If an application wants to use more than one device type, it can indicate this
using the 0xBD0 field in the Application Header (see "bin_rpa.rst" for further
details).

When multiple devices are present, the host may decide to provide multiple
logical presentations for a single physical device. This condition can be
detected using the 0x0410 "Get device properties" kernel call, on bits 0-4 of
the return value. If a valid target device is provided here, it indicates that
device representing better the actual physical device.

This feature may be used to provide very different abstractions to the same
physical device facilitating the use from applications. A typical example
would be a text input device and a keyboard, both supported by a physical
keyboard.




Touch awareness
------------------------------------------------------------------------------


An application can indicate that it is touch aware by the 0xBC1 field of the
Application Header (see "bin_rpa.rst").

This flag if set indicates that the application will set up the touch
sensitive areas according to the controller it actively uses (the areas map to
digital input bank zero). This case the host may provide a specific emulated
device (such as a digital gamepad) without actually providing any physical
buttons or switches for it.

If the application does not provide this (has the flag cleared), the host
should assume it won't use the touch sensitive areas to complement a
controller, and so may use different means of emulation or may provide a more
limited set of input devices.

If the application accepts a touch pointing device as input device and
provides this flag, the host should provide the virtual devices which are
supposed to be complemented with touch buttons as mapping to the touch
pointing device (see "Multiple devices, combinations" above). If the
application does not accept a touch pointing device, these may be provided
stand-alone.




The description of device types
------------------------------------------------------------------------------


Here each of the supported input devices are described with their digital and
analog input mappings.


0x0: Mouse pointing device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The mouse pointing device provides a normal computer mouse requiring a cursor
presented for the user to assist tracking it's position. The touch sensitive
areas can be used to specify automatic buttons for this device, which are
activated by a primary mouse button (typically left) press.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
| 0    | 0-15  | Touch sensitive areas (buttons)                             |
+------+-------+-------------------------------------------------------------+
|      | 0     | Scroll up (if any)                                          |
| 1    +-------+-------------------------------------------------------------+
|      | 1     | Scroll right (if any)                                       |
|      +-------+-------------------------------------------------------------+
|      | 2     | Scroll down (if any)                                        |
|      +-------+-------------------------------------------------------------+
|      | 3     | Scroll left (if any)                                        |
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


0x1: Touch pointing device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The touch pointing device assumes a touch display where no cursor is necessary
to feed back to the user's actions. The device may support multi-touch, and
so the touch sensitive areas may return press information simultaneously even
if they don't overlap.

Digital input mapping:

+------+-------+-------------------------------------------------------------+
| Bank | Input | Description                                                 |
+======+=======+=============================================================+
| 0    | 0-15  | Touch sensitive areas (buttons)                             |
+------+-------+-------------------------------------------------------------+
|      | 4     | Primary touch activity                                      |
| 1    +-------+-------------------------------------------------------------+
|      | 5     | Secondary touch activity (if supported)                     |
+------+-------+-------------------------------------------------------------+

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
kernel call (0x0423: Pop text input FIFO). It provides no digital or analog
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
    | |CTRL|    |ALTG|         SPACE          |ALT |         |CTRL|
    | +----+    +----+------------------------+----+         +----+
    +----------------------------------------------------------------...

If necessary, the actual labeling of the keys may be requestable using the
0x0411 "Get digital input description symbols" kernel call.

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
|      | 6     | ESC; Numpad Del; Delete                                     |
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
| 2 | F-Row  |ESC| F1| F2| F3| F4| F5| F6| F7| F8| F9|F10|F11|F12|#0 |#0 |#0 |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 3 | NumRow | ~ | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0 | - | + | | |BKS|   |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
| 4 | UpRow  |TAB| Q | W | E | R | T | Y | U | I | O | P | { | } |           |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 5 | HomeRow|#1 | A | S | D | F | G | H | J | K | L | : | " |#2 |ENT|       |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 6 | BotRow |SHL|#3 | Y | X | C | V | B | N | M | < | > | ? |#3 |SHR|       |
+---+--------+---+---+---+---+---+---+---+---+---+---+---+---+---+---+-------+
| 7 | Control|CTL|#4 |ALG|SPC|ALT|#4 |#4 |CTR|#5 |                           |
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




Digital input description symbols
------------------------------------------------------------------------------


The kernel function 0x0411 "Get digital input description symbols" return the
assignment of digital inputs to specific physical devices, typically the keys
on a keyboard.

The purpose of this function is twofold: for one, it provides information on
whether the particular input is available (returning zero unless so), for an
other, it may be use to assist users of the application to locate the physical
inputs required to control the application.

For most keyboard keys simply the UTF-32 character code is returned. This way
aware applications may even display some international characters if the
keyboard is known to have such. Note that always the uppercase variant of the
character should be returned by the host for this purpose unless separate keys
are provided for the lowercase and uppercase variants of the character. Note
that several keys map to certain ASCII control codes, these are also listed.

Otherwise the following special codes are available:

+--------------+-------------------------------------------------------------+
| Code (32bit) | Description                                                 |
+==============+=============================================================+
| 0x00000000   | Input does not exist (may only be provided by touch)        |
+--------------+-------------------------------------------------------------+
| 0x00000008   | 'Backspace' key                                             |
+--------------+-------------------------------------------------------------+
| 0x00000009   | 'TAB' key                                                   |
+--------------+-------------------------------------------------------------+
| 0x0000000A   | Main 'Enter' key                                            |
+--------------+-------------------------------------------------------------+
| 0x0000001B   | 'ESC' key                                                   |
+--------------+-------------------------------------------------------------+
| 0x00000020   | 'Space' key                                                 |
+--------------+-------------------------------------------------------------+
| 0x0000007F   | 'Delete' key                                                |
+--------------+-------------------------------------------------------------+
| 0x8000000A   | Numeric pad 'Enter'                                         |
+--------------+-------------------------------------------------------------+
| 0x8000002A   | Numeric pad '*'                                             |
+--------------+-------------------------------------------------------------+
| 0x8000002B   | Numeric pad '+'                                             |
+--------------+-------------------------------------------------------------+
| 0x8000002C   | Numeric pad ',' (Del)                                       |
+--------------+-------------------------------------------------------------+
| 0x8000002D   | Numeric pad '-'                                             |
+--------------+-------------------------------------------------------------+
| 0x8000002F   | Numeric pad '/'                                             |
+--------------+-------------------------------------------------------------+
| 0x80000030   |                                                             |
| \-           | Numeric pad '0' - '9'                                       |
| 0x80000039   |                                                             |
+--------------+-------------------------------------------------------------+
| 0x80000081   |                                                             |
| \-           | 'Fxx' function keys, typically 'F1' - 'F12'.                |
| 0x8000008C   |                                                             |
+--------------+-------------------------------------------------------------+
| 0x80000090   | Left 'Shift' key                                            |
+--------------+-------------------------------------------------------------+
| 0x80000091   | Right 'Shift' key                                           |
+--------------+-------------------------------------------------------------+
| 0x80000092   | Left 'Ctrl' key                                             |
+--------------+-------------------------------------------------------------+
| 0x80000093   | Right 'Ctrl' key                                            |
+--------------+-------------------------------------------------------------+
| 0x80000094   | Left 'Alt' key                                              |
+--------------+-------------------------------------------------------------+
| 0x80000095   | Right 'Alt' key (Alt Gr)                                    |
+--------------+-------------------------------------------------------------+
| 0x80000096   | 'Insert' key                                                |
+--------------+-------------------------------------------------------------+
| 0x80000098   | 'Home' key                                                  |
+--------------+-------------------------------------------------------------+
| 0x80000099   | 'End' key                                                   |
+--------------+-------------------------------------------------------------+
| 0x8000009A   | 'Page Up' key                                               |
+--------------+-------------------------------------------------------------+
| 0x8000009B   | 'Page Down' key                                             |
+--------------+-------------------------------------------------------------+
| 0x8000009C   | Left direction key                                          |
+--------------+-------------------------------------------------------------+
| 0x8000009D   | Down direction key                                          |
+--------------+-------------------------------------------------------------+
| 0x8000009E   | Right direction key                                         |
+--------------+-------------------------------------------------------------+
| 0x8000009F   | Up direction key                                            |
+--------------+-------------------------------------------------------------+
| 0xFFFFFFFD   | Special keyboard control                                    |
+--------------+-------------------------------------------------------------+
| 0xFFFFFFFE   | Special other controller control                            |
+--------------+-------------------------------------------------------------+
| 0xFFFFFFFF   | Native control                                              |
+--------------+-------------------------------------------------------------+

The "Special keyboard control" code (0xFFFFFFFD) indicates a keyboard button
which can not be identified (either for the limitations of the host or the
specialty of the actual keyboard button).

The "Special other controller control" code (0xFFFFFFFE) indicates a button or
other mean of control on a non-keyboard device which is neither a native
device. Native device is a device which physically matches to the device type
it represents (for example a physical joystick serving a joystick type input
device).

The "Native control" indicates a control on the device itself if the device
physically matches to the device type it represents (except for keyboard).




Text input control codes
------------------------------------------------------------------------------


The kernel function 0x0423 "Pop text input FIFO" returns the next character or
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
| 0x80000096   | Insert: Toggle insertion mode                               |
+--------------+-------------------------------------------------------------+
| 0x80000098   | Home: Position the text cursor at the beginning of the line |
+--------------+-------------------------------------------------------------+
| 0x80000099   | End: Position the text cursor at the end of the line        |
+--------------+-------------------------------------------------------------+
| 0x8000009A   | Page Up: Move text cursor up a page                         |
+--------------+-------------------------------------------------------------+
| 0x8000009B   | Page Down: Move text cursor down a page                     |
+--------------+-------------------------------------------------------------+
| 0x8000009C   | Left: Move text cursor left a character                     |
+--------------+-------------------------------------------------------------+
| 0x8000009D   | Down: Move text cursor down a line                          |
+--------------+-------------------------------------------------------------+
| 0x8000009E   | Right: Move text cursor right a character                   |
+--------------+-------------------------------------------------------------+
| 0x8000009F   | Up: Move text cursor up a line                              |
+--------------+-------------------------------------------------------------+
