
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
Application Header (see locations 0xBC1, 0xBD0 and 0xBD1 at "Application
binary header map" in "bin_rpa.rst" for details), so the host may select the
most appropriate way it may serve the application.

In this part of the specification, the additional details required for
completing the input system are defined, along with acceptable suggested
means of implementation and usages of the system.




Input device types
------------------------------------------------------------------------------


The RRPGE system is capable to use the following genres of input devices:

- Digital gamepads. These typically provide four buttons for directions, and
  typically four or more additional buttons. They have no capability for
  producing analog inputs. The host may use a physical keyboard to simulate
  one, however on a keyboardless device it may be difficult to emulate.

- Pointing devices. Two major types are understood: Mice or similar devices
  which require an on-screen cursor for feeding back position information to
  the user, and typically have multiple buttons, and Touch screens which
  operate on the concept of the user interacting with the display itself, thus
  not requiring an on-screen cursor, but typically having only a single
  "button".

- Analog joysticks. A generic analog device usually meant to be used for
  flight simulators and similar applications.

- Steering wheels. An analog device meant to be used for driving simulation.
  Typically this is not a real physical device, but suggests the host to
  provide a configuration more resembling to such a device (such as using a
  handheld device's orientation sensors to provide an impression of a steering
  wheel).

- Tilt sensors. An analog device meant to rely on tilt to simulate gravity
  based applications, such as ball guiding toys.

In addition to the genres, the application may suggest the following
properties of it's input requirements (using the Application Header's 0xBC1
field):

- The capability to use touch. This indicates that the application is touch -
  aware, that is it will use the appropriate kernel functions to set up touch
  sensitive areas to make it's digital inputs functional even if physical
  digital inputs can not be provided at all by the host. This flag suggests
  the host that it does not need to resort to some awkward emulation to
  provide virtual digital inputs to operate the application.

- The use of text input. This indicates that the application makes use of text
  input devices. Note that these devices not necessarily have to be keyboards.
  If this flag is set, if possible, the host should provide some mean of
  providing text input for the application, otherwise it need not provide such
  a feature.




Additional devices, combinations
------------------------------------------------------------------------------


While the most simple applications may work well with a single type of input
device, some may find it beneficial to directly support alternatives, or may
need a combination of devices for proper control.

There are two distinct ways to add extra devices:

- Alternative controllers (0xBD0 in the Application Header)
- Secondary controllers (0xBD1 in the Application Header)


Alternative controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

An alternative controller is meant to be supported as an alternate "fallback"
solution if the controller originally suggested is not available or would be
cumbersome to be emulated by the host or to the user.

If the application specifies an alternate controller, it allows the host to
discard cumbersome emulation solutions probably replacing those with directly
providing the alternative controller.

For this property these input devices may map directly to an other provided by
the host. To identify this mapping when it happens, the Kernel function 0x0410
"Get device properties" may be used.

Note that always the alternate controller is which maps to a primary
controller and not vice-versa even if the primary provided by the host is a
poor quality emulation.

The application using this feature must be aware of these, and should not
attempt to use an alternative controller and the primary controller it maps to
simultaneously.


Secondary controllers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A secondary controller is meant to expand the functionality of a primary or
alternative controller.

If the application specifies it benefits from a secondary, the host should not
use suitable secondary controllers to emulate primary or alternate
controllers, instead it should provide the secondary.

The secondary may also map to a primary (or alternate) controller which this
case means it is physically part of that device. A typical example may be a
pointing device combined with a steering wheel on a handheld device.

If the mapping is present, the buttons of the secondary controller may be
absent. This case the application should combine the two controllers and
use them as one by relying on the primary controller's buttons.




The uses of tilt compensation
------------------------------------------------------------------------------


Using the tilt compensation flag (returned by the kernel function 0x0410 "Get
device properties") the host may indicate that the actions on the controller
cause a change in the orientation of the device's display which the
application may want to compensate.

The meaning of the flag is described in the following subsections for each
device type it may affect.


Steering wheels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

At the centered (X = 320) position the display is landscape oriented.
Increasing X also indicates a clockwise rotation of 1/3 degree for each step
in X, so the maximal X = 639 indicating a rotation of 106.33 degrees
clockwise. Decreasing X has the opposing effect.


Tilt sensors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In the case of a tilt sensor the display is assumed to be tilted. The centered
position is X = 320, Y = 200. For each step in either direction an 1/8 degree
of tilt is assumed. X increments correspond to tilts to the right, Y
increments correspond to tilts to the bottom.




The mapping of digital inputs
------------------------------------------------------------------------------


The kernel function 0x0420 "Get digital inputs" returns the currently active
digital inputs. The layout and roles of these inputs depend on the type of the
device. If touch buttons are enabled, either of these inputs may be provided
from a touch sensitive area as defined using function 0x0430 "Define touch
sensitive area".

Following the roles of the buttons are defined for each input device type.


Digital gamepads
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- bit 15: Direction Up
- bit 14: Direction Right
- bit 13: Direction Down
- bit 12: Direction Left
- bit 11: Primary fire / action
- bit 10: Secondary fire / action

The rest of the buttons (if any) may be assigned ordered by accessibility on
the following positions.


Pointing devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- bit 15: Mouse wheel Up
- bit 13: Mouse wheel Down
- bit 11: Primary (left) mouse button or press on touch device
- bit 10: Secondary (right) mouse button or secondary press on touch device
- bit  9: Middle mouse button

The rest of the buttons (if any) may be assigned ordered by accessibility on
the following positions. Bits 14 and 12 may be used for Right and Left
respectively if the mouse has a horizontal scroll feature.

Note that a touch device may only provide a single press input, any other
buttons may have to be provided using physical buttons on the device or as
touch sensitive areas. Multi-touch devices may provide a secondary press
accompanied with further analog inputs. Touch aware applications should use
the pointing device accordingly.


Analog joysticks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- bit 11: Primary fire / action
- bit 10: Secondary fire / action

The rest of the buttons (if any) may be assigned ordered by accessibility on
the following positions. Bits 12-15 may be used if the joystick is capable to
provide digital directions by some extra device.


Steering wheels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- bit 11: Primary fire / action
- bit 10: Secondary fire / action
- bit  9: Gear Up
- bit  8: Gear Down
- bit  7: Apply hand brakes

The rest of the buttons (if any) may be assigned ordered by accessibility on
the following positions.




Extended analog inputs
------------------------------------------------------------------------------


Extended analog inputs may be provided by kernel function 0x0422 "Get further
analog inputs". The assignment of these inputs depend on the device type.


Pointing devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A touch device may use these information to report secondary touch if it is
capable to do so. If so, the secondary button press should be connected with
these position information.


Analog joysticks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the joystick has a throttle controller, it's information should return in
A. The throttle controller is meant to be interpreted as zero or above, so the
return value accordingly has to ramp from 200 to 399 inclusive (since 200
refers to the idle position). If the joystick's design suggests the
possibility of negative throttle having a well defined idle position, it
should be exploited by assigning 0 to 199 to the positions below idle level.

The C return should be used as a horizontal orientation control; if the stick
may be twisted, this control might be assigned to this return.


Steering wheels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The pressure on the gas pedal may be returned in A. This is a positive only
value, so it should ramp accordingly from 200 to 399 inclusive.

The pressure on the brakes may be returned in C. This is a positive only
value, so it should ramp accordingly from 320 to 639 inclusive.




Pointing device specific concerns
------------------------------------------------------------------------------


Touch aware applications (as set by the 0xBC1 field in the Application Header)
should be designed so they work both with a mouse and a touch device.

The type of the pointing device should be determined by reading bit 10 of the
return of kernel function 0x0410 "Get device properties". If this bit
indicates that it is not a touch device, a mouse has to be assumed.

In case of having a mouse, the application should provide a cursor displayed
at the position acquired by function 0x0421 "Get analog positions".

For most part a mouse and a touch device may be assumed to behave similarly to
each other. No matter which is the actual device, touch sensitive areas may be
defined and will behave appropriately.

The following differences may be kept in mind however:

- Touch devices may provide secondary touch while mice don't. This should be
  paid attention to when assigning functions to the secondary mouse button.

- Touch devices have no additional controls (extra buttons, scroll wheels), so
  these features should be provided using touch sensitive areas.

- Functions requiring secondary touch are not directly possible with a mouse
  or a touch device not capable to detect secondary touch.

- Pressing touch buttons likely change the analog positions reported on a
  touch device. This should be kept in mind when assigning functions to those
  with a mouse in mind.




Steering wheel specific concerns
------------------------------------------------------------------------------


In the case of a steering wheel acceleration and brake information (pressures)
are provided as extended analog inputs. These, particularly if the steering
wheel is emulated, may come from a single source, so some simple applications
might want to simplify the model as well.

For this purpose on the Y return of 0x0421 "Get analog positions" a combined
acceleration / brake information is available. Above 200 values indicate
acceleration, below 200 indicate braking. If these information come from
separate sources, they are merged by making the brake dominating if possible
(but may be merged by adding as well).




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
physically matches to the device type it represents.




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
