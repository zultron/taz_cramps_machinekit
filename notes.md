# INI file
INI [docs][1]

[1]: http://www.machinekit.io/docs/config/ini_config/

## Homing

    HOME_SEARCH_VEL =       -5.0
    HOME_LATCH_VEL =        5.0

# HAL file

- [Pin mappings][2]

[2]: https://drive.google.com/open?id=1xLqDcoijuftPpDJuLmIFQLNdOEL-pKPV24phtdy3Io8

- `hal_temp_bbb` broken; high pin limit needs to be set to 6:

        $ halcmd loadusr -Wn Therm hal_temp_bbb -n Therm -c \
	      04:epcos_B57560G1104,05:epcos_B57560G1104,06:epcos_B57560G1104 -b CRAMPS
        Pin not available

# Testing

## Connector diagram

             +-+ Large notch
		  3 X X X 1
		 7 X X X X 4
		11 X X X X 8
		 13 X X X 12

        Rear:        Top:
		+-----+      +-----+
		| X   | XZ   | X   | Ext1
		|   X | YZ   | X   | Ext2
		| x   | Bed  |   | |
		|   x | Pwr  |   | |
		+-----+      +-----+

## Power continuity
- Unplug power and printer
- Check continuity between power connector red wires and power switch
- With power switch on, check continuity between power connector red
  wires and:
  - Wire 1:  Motor and extruder power `+` terminals
  - Wire 2:  Bed power `+` terminal
- Repeat with switch off, verifying no continuity
- Check continuity between power connector yellow wires and:
  - Wire 1:  Motor, extruder and system power `-` terminals
  - Wire 2:  Bed power `-` terminal
- 5V checks FIXME
- E2 MOSFET module checks FIXME

## Power voltage
- Unplug wire from 5V converter module OUT+ to CRAMPS 5V system power
- Remove three fuses
- Remove Pololu modules
- Turn enclosure power switch off
- Plug in power supply
- Turn power supply switch on
- Check for 24V DC between power connector red and yellow wires
- Check for 0V DC at motor and bed power `+` terminals
- Turn enclosure power switch on
- Check for 24V DC at motor, extruder and bed power `+` terminals, and
  5V converter module `IN+`
- Check for 5V DC at 5V converter module OUT
- Check for 0V DC at Motor, extruder and bed power `-` terminals
- Turn enclosure power switch off
- Replace three fuses, with 15A fuse in middle holder
- Turn enclosure power switch on
- Check for 24V DC at P401 Bed Heater and P405 E0 power out `+` terminals
- Turn enclosure power switch off
- Plug wire from 5V converter module OUT+ to CRAMPS 5V system power
- Turn enclosure power switch on
- Verify `BB ON` LED is lit
- Check for 5V DC at P404 5V System Power terminal

## Limit switches and motor step/dir

- Setup
  - Disconnect control box cables except for power
  - Remove Pololu/stepper driver boards
  - Power on, boot up, `ssh -X` in
  - `cd` to this directory
  - `linuxcnc CRAMPS.ini &`; wait for Axis GUI to appear
- Test each limit switch:
  - Run `halcmd show signal limit-*-min`, where `*` is `x`, `y` or
	`z`; verify the value is false
  - Short the correct pins on the correct connector
	- X:  XZ connector pins 6 and 10
	- Y:  YZ connector pins 5 and 9
	- Z:  XZ connector pins 9 and 13
  - Rerun the same command; verify the value is true
- Test enable signal and each motor step and dir signals:
  - In Axis GUI, toggle estop and power buttons
  - On the "Manual Control" tab, select the axis to test, `X`, `Y`,
    `Z` or `A`
  - Put a volt meter on `DIR` pin 8 for the CRAMPS Pololu connector
    corresponding to the axis:  `X` = `X`; `Y` = `Y`; `Z` = `Z` and
    `E2`; `A` = `E0` (`E1` not yet enabled)
  - With the jog increment set to continuous, jog with the `+` and `-`
    buttons, verifying the voltage goes between 0V and 3.3V
  - Put the volt meter on the `Step` pin 7, and verify the voltage
    increases some small amount above 0V
- Test ESTOP:
  - Remove the jumper from CRAMPS `ESTOP` connector P302; verify that
    the machine has reset in Axis; verify the CRAMPS `ESTOP` LED
    turns on
  - Replace the jumper; in Axis, toggle ESTOP and machine power;
    verify the CRAMPS `ESTOP` LED turns off
- Test machine power signal:
  - Toggle machine power button in Axis; verify `STATUS` LED lights
    with power on
- Test bed and extruder heater FETs: FIXME
- Test bed and extruder heater thermistors: FIXME
