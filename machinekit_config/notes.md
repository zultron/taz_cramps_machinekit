# Dual extruders

From [flexydually OHAI][fd-ohai]:

- Only one switchable fan
- [Duallie docs][duallie]
- Set temps:
  - Change tool head with `T0` (rear) and `T1` (front)
  - GUI has entry fields for extruder temp and bed (e.g. 240 and 100
    deg. C)
  - Wait until temp hits target, then click 'print' button
- Fan
  - Seems to be turned on per print; like coolant?
  - Apparently needs PWM to control speed?
  - `M106 P0 S255` sets fan 1 PWM on 100%
  - Repetier usage: `M106 P1 T45 H1:2` sets fan 2 to 'thermostatic'
    mode, on when either heater 1 or 2 is above 45 C.
  - `M107` is fan off (deprecated)
  - PWM period [may matter][fan-pwm]
  
After testing:
- Using one stepgen for both motors doesn't work; switching the
  extruder causes a 'joint following error'
- Instead, we probably need two stepgens wired to look like one to
  motion:
  - Two `position-fb` pins are added before going into `motor-pos-fb`
  - The deselected motor `position-cmd` input must be locked, and the
    difference of that and `motor-pos-cmd` coming out must be passed
    into the active `position-cmd` input
  - Probably needs a custom comp.

[fd-ohai]: https://ohai.lulzbot.com/project/lulzbot-taz-flexydually-tool-head-v2-installation/accessories/
[duallie]: http://devel.lulzbot.com/TAZ/accessories/javelin/
[fan-pwm]: https://groups.google.com/forum/#!topic/machinekit/BSLiZSQKoDQ

# 3D printer G code
- G codes at [reprap.org][rro-gcode]; note that 'Machinekit' is
  included in instruction implementation notes
- Some G codes in Machinekit `nc_files/` subdir:
  - `M104` set extruder temp
  - `M106 S255` fan off/on
- Daren Schwenke has a lot of M codes [implemented][ds-mcodes]


[rro-gcode]: http://reprap.org/wiki/G-code
[ds-mcodes]: https://github.com/Arcus-3d/Arcus-3D-M1

# Notes from TAZ configuration
- From the [TAZ firmware][TAZ-fw], `Marlin/Configuration.h`
- Heater limits:
  - They set e.g. `HEATER_0_MINTEMP` to 5, below which assume
    thermistor short/failure, and heater is disabled
  - `HEATER_0_MAXTEMP` is 250, then heater shut off; prevents
    accidental overheating
- Extruder PID terms:
  - Preset for 24V Buda 2.0:  Kp 6, Ki .3, Kd 125
  - `PID_MAX` 74; limits current to nozzle while PID is active (see
    `PID_FUNCTIONAL_RANGE` below); 255=full current
  - `PID_FUNCTIONAL_RANGE` 16; if the temperature difference between
    the target temperature and the actual temperature is more then
    `PID_FUNCTIONAL_RANGE` then the PID will be shut off and the
    heater will be set to min/max.
  - `PID_INTEGRAL_DRIVE_MAX` 70; limit for the integral term
  - `K1` 0.96; smoothing factor within the PID
  - `PID_dT` `((OVERSAMPLENR * 8.0)/(F_CPU / 64.0 / 256.0))` sampling
    period of the temperature routine
  - `EXTRUDE_MINTEMP` 120
- Bed PID terms:
  - Presets for 24V 360W silicone heater from NPH on 3mm borosilicate
    (TAZ 2.2+)
  - `MAX_BED_POWER` 206; limits duty cycle to bed; 255=full current
  - `DEFAULT_bedKp` 20; `DEFAULT_bedKi` 5; `DEFAULT_bedKd` 275
- Geometry:
  - `X_MAX_POS` 298; `Y_MAX_POS` 275; `Z_MAX_POS` 250
- Good stuff about auto bed leveling
- Homing speeds, mm/min:  50*60, 50*60, 8*60
- Axis steps per unit:   100.5,100.5,1600,850
- Max feedrate, mm/S:  800, 800, 8, 50
- Max accel:  9000,9000,100,10000 "max start speed for accelerated moves"
- Default accel:  500 mm/S^2 all joins max accel for printing moves
- Default retract accel:  3000 mm/S^2 max accel for retracts
- Offset of extruders, mm, X,Y:  E0 0,0; E1 0, -52
- Default jerk, mm/S: XY 8.0; Z 0.4; E 10.0; The speed change that
  does not require acceleration (i.e. the software might assume it can
  be done instantaneously)
- Preheat constants:
  - PLA:  hotend temp 180; hpb temp 50; fan speed 0/255;
  - ABS:  hotend temp 230; hpb temp 85; fan speed 0/255;
- More stuff in `Configuration_adv.h`
- Also `thermistortables.h`

[TAZ-fw]: http://devel.lulzbot.com/TAZ/Holly/software/2014Q3_rB/Marlin/Marlin-2014Q3_rB.tar

# INI file
INI [docs][1]

[1]: http://www.machinekit.io/docs/config/ini_config/

## Steps/mm

From [flexydually OHAI][fd-ohai]:

- X 100.5
- Y 100.5
- Z 1600.0
- Extruder:  configurable, from label
- Retract 3000

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

        Enclosure connector layouts
        Rear:        Top:
        +-----+      +-----+
        | X   | XZ   | X   | Ext1
        |   X | YZ   | X   | Ext2
        | x   | Bed  |   | | D Sub
        |   o | Pwr  |   | | (Empty)
        +-----+      +-----+

        Large 'X' connector pin numbering
             +-+ Large notch
          3 X X X 1
         7 X X X X 4
        11 X X X X 8
         13 X X X 12

        Small 'x' connector pin numbering
           +-+ Large notch
          1 X
         3 X X 2
          4 X

        Pololu header pin numbering
        1 .    . 16
          .    .
          .    .    .
          .    .  | . (Motor conn.)
          .    .  | .
          .    .    .
          .    .
        8 .    . 9

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
    `E2`; `A` = extruder (see below); use any ground, or pin 9
  - With the jog increment set to continuous, jog with the `+` and `-`
    buttons, verifying the voltage goes between 0V and 3.3V
  - Put the volt meter on the `Step` pin 7, and verify the voltage
    increases some small amount above 0V; try increasing the "Jog
    Speed" slider for better results
  - For the `A` axis, test as above for the default `E0` pololu socket
    first; then run `halcmd sets extruder.select 1` and repeat for the
    `E1` socket
- Test ESTOP:
  - Remove the shunt from CRAMPS `ESTOP` connector P302; verify that
    the machine has reset in Axis; verify the CRAMPS `ESTOP` LED
    turns on
  - Replace the shunt; in Axis, toggle ESTOP and machine power;
    verify the CRAMPS `ESTOP` LED turns off
- Test machine power signal:
  - Toggle machine power button in Axis; verify `STATUS` LED lights
    with power on
- Test bed and extruder heater FETs:
  - Run `halcmd setp hpg.pwmgen.00.out.*.value 1.0`, where `*` is
    `00`, `01` or `02` for bed, E0 or E1, respectively
  - Verify voltage at output
  - Run `halcmd setp hpg.pwmgen.00.out.*.value 0.0`
  - Verify 0V at output
- Test bed and extruder heater thermistors: FIXME
