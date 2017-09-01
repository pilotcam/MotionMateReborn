# Motion Mate Robot Reborn - Control Language

## Introduction

This document outlines the motion control language that shall be used to program and control the robot movements.

The *programs* shall be stored in a format that conforms to this language such that when a program is read by the
*robot control task*, it can be interpreted and iteritively passed to the robot control task (or other tasks as
deemed appropriate) to achieve the desired outcome.

## TO-DO

This document is a work-in-progress.  Please feel free to contribute.  The folowing parts are known to be missing
(please add other parts if known):
* Fault handling: how should the robot react when something doesn't go as expected (eg. position feedback not detected
within a certain amount of time)
* Auxiliary input handling

## Encoding

The robot control language is designed to be human readable.  The text shall be encoded in 8 bit ASCII format.  All text
streams shall be assumed to ASCII; thus a byte order marker shall be assumed to be part of the control stream.

### Whitespace

Whitespace shall be used to provide seperation between commands and arguments.  Whitespace is defined as one or more
consecutive occurences of the following characters:

* Space (dec 32)
* Tab (dec 9)

### Numbers

Numbers shall be stored as strings within the robot control language.  Numbers shall be seperated by whitespace from
other numbers, commands, or arguments.  All numbers shall be whole decimal (based 10) numbers within a range of -32768
through 32767.  Fractional numbers shall not be supported.

### Command structure

Commands are verbs (plus arguments) that yield a single duty for the robot task (or supporting task) to perform.  
Commands shall be encoded within a single line of the text stream.  A line boundary includes one or more of the following characters:
* Carriage Return (dec 13)
* Line Feed (dec 10)

A command shall be seperated from its arguments by whitespace.  The command definition is completed by a line boundary.


## Axis Numbering

Some axes use an `AxisID` parameter to select which axis should be affected by the command.  An `AxisID` is a single
character used to identify an axis.  The axis IDs for the robot are defined as follows:

* 0: Base Rotation (CW, CCW)
* 1: Lift (up, down)
* 2: Extend (extend, retract)
* 3: Wrist Rotation (CW, CCW)
* 4: Gripper (grip, release)
* A: End effector Axis #1
* B: End effector Axis #2
* C: End effector Axis #3

## Parse Errors

If while parsing the robot control language, an unexpected or invalid character is received, all motion shall stop (to
the extent possible) and an error condition shall be thrown.  The specified location (line and column number) shall be
logged to the debug console.

## Commmand List

The following is a complete list of commands supported by the control language.

### M: Move

    M AxisID Position [Speed] [Acceleration] [Deceleration]

Moves axis `AxisID` to the designated `Position` using the dynamics specified in the arguments.

Most robot axes do not have closed loop position/speed control.  For these axes, a positive number shall cause unbounded motion in
the forward (extend, clockwise, grip, etc.) direction.  A negative number shall have the opposite effect.  A position equal to zero
shall stop motion if the axis supports this ability.  If the axis is controlled by a pneumatic two-position valve, a position of
zero shall have no effect.

If the identified axis supports motion dynamics (eg. stepper motor control), the *Speed*, *Acceleration*, and *Deceleration*
parameters shall used to provision this control.  The units of measure for these values shall be *percent of maximum* and have a
range of 1 to 100. If the user program omits any of these parameters, the value shall be assumed to be 100% of maximum.

### T: Timed dwell

    T DwellTimeMilliseconds

Pauses execution of the robot program for the duration specified in the argument.

### W: Wait for axis in-position

    W AxisID

The robot program shall pause and wait for the designated robot axis to reach the position programmed in the most
recent move command.  There are several methods of satifying the wait requirement depending on the type of axis:
* For axes without any position feedback, the wait shall be satified by a fixed duration from the move start time.
The duration shall be defined as a constant value for each axis.
* For axes that have with position feedback, but only at the extreme ends of travel:
	* If the move command specified a position of `0`, the wait shall be satified immediately
	* For a non-zero position, the `W` command shall be satisfied when the corresponding sensor input is in a state
	  that indicates the axis has indeed reached the intended position.
* For axes with position control but no feedback (eg. stepper motors), the `W` command shall be satisfied when the
  command velocity is zero and the command position has been achieved.
* Axes with analaog position feedback (eg. encoder feedback) are not currently supported

### H: Home

    H AxisID

Home the designated axis using whatever motion is required to bring that axis to a known position.  For pneumatic axes, this command is
identical to a move command.  For axes with closed loop control, the axis must move to a designated home position using a switch or
other absolute position feedback sensor.

Note that a `W` (wait for in-position) command that follows a `H` (home) command shall use the home position as the position that must
be achieved to satisfy the `W` command.







