============================
Atlantic 3D Printer Firmware
============================

[![Flattr this git repo](http://api.flattr.com/button/flattr-badge-large.png)](https://flattr.com/submit/auto?user_id=TerawattIndustries&url=https://github.com/Terawatt-Industries/Atlantic.git&title=Atlantic&language=&tags=github&category=software)

Quick Information
===================
This is the official firmware for Terawatt Industries 3D Printers.  It is based on Marlin v1.0.0.  By default it's configured for the PrismX 3D Printer, but it can be used with our MendelMax 1.5 for LM8UU or any other RAMPS1.4-based 3D printer.

This firmware is based on Marlin, which is in turn based on grbl and many original parts.  This fork is a "sanitized" version of Marlin which focuses on the equipment used on Terawatt Industries printers.  We also deleted all LCD hacks from the code.

 (https://github.com/ErikZalm/Marlin/tree/Marlin_v1/Marlin)
 (https://github.com/simen/grbl/tree)

Features:
*   Interrupt based movement with real linear acceleration
*   High steprate
*   Look ahead (Keep the speed high when possible. High cornering speed)
*   Advanced temperature protection
*   preliminary support for Matthew Roberts advance algorithm
    For more info see: http://reprap.org/pipermail/reprap-dev/2011-May/003323.html
*   Full endstop support
*   SD Card support
*   SD Card folders (works in pronterface)
*   SD Card autostart support
*   Support for 2nd serial port (multiplexing)
*   EEPROM storage of e.g. max-velocity, max-acceleration, and similar variables
*   many small but handy things originating from bkubicek's fork.
*   Arc support
*   Temperature oversampling
*   Dynamic Temperature setpointing aka "AutoTemp"
*   Support for QTMarlin, a very beta GUI for PID-tuning and velocity-acceleration testing. https://github.com/bkubicek/QTMarlin
*   Endstop trigger reporting to the host software.
*   Updated sdcardlib
*   Heater power reporting. Useful for PID monitoring.
*   PID tuning
*   CoreXY kinematics (www.corexy.com/theory.html)
*   Delta robot kinematics
*   Configurable serial port to support connection of wireless adaptors.
*   Automatic operation of extruder/cold-end cooling fans based on nozzle temperature
*   RC Servo Support, specify angle or duration for continuous rotation servos.
*   Specification for attaching external control interfaces such as touch-screen and LCD panels

This firmware is optimized for Ramps1.4 electronics.

The default baudrate is 115200.  This accommodates older versions of Python which is commonly used on PCs that connect to this firmware.  This is not an optimal baud rate (in terms of jitter) but has negligible impact on overall performance, since there is an ACK sequence that must be performed and this is the primary bottleneck on PC-to-printer communication.

Differences and additions to previous firmwares:
================================================================

*Hotend Overheat Protection:*
This firmware implements a much improved algorithm for detecting the unlikely but catastrophic event of a thermistor becoming dislodged during prints.  It does this with an averaged second-order differential of analog temperature readings.  Multiple extruders are supported.  Unlike previous methods, this algorithm is not inherently prone to missed edge cases that could result in no protection.  Unlike previous methods, this algorithm is not dependent on the mass of the hot end.

*Look-ahead:*

Marlin has look-ahead. While sprinter has to break and re-accelerate at each corner,
lookahead will only decelerate and accelerate to a velocity,
so that the change in vectorial velocity magnitude is less than the xy_jerk_velocity.
This is only possible, if some future moves are already processed, hence the name.
It leads to less over-deposition at corners, especially at flat angles.

*Arc support:*

Slic3r can find curves that, although broken into segments, were ment to describe an arc.
Marlin is able to print those arcs. The advantage is the firmware can choose the resolution,
and can perform the arc with nearly constant velocity, resulting in a nice finish.
Also, less serial communication is needed.

*Temperature Oversampling:*

To reduce noise and make the PID-differential term more useful, 16 ADC conversion results are averaged.

*AutoTemp:*

If your gcode contains a wide spread of extruder velocities, or you realtime change the building speed, the temperature should be changed accordingly.
Usually, higher speed requires higher temperature.
This can now be performed by the AutoTemp function
By calling M109 S<mintemp> T<maxtemp> F<factor> you enter the autotemp mode.

You can leave it by calling M109 without any F.
If active, the maximal extruder stepper rate of all buffered moves will be calculated, and named "maxerate" [steps/sec].
The wanted temperature then will be set to t=tempmin+factor*maxerate, while being limited between tempmin and tempmax.
If the target temperature is set manually or by gcode to a value less then tempmin, it will be kept without change.
Ideally, your gcode can be completely free of temperature controls, apart from a M109 S T F in the start.gcode, and a M109 S0 in the end.gcode.

*EEPROM:*

If you know your PID values, the acceleration and max-velocities of your unique machine, you can set them, and finally store them in the EEPROM.
After each reboot, it will magically load them from EEPROM, independent what your Configuration.h says.

*LCD Menu:*
FIXME:  rewrite -- we now have a specification on how vendors should attach custom gadgets such as LCD panels.  This firmware is _not_ backwards compatible.  If another vendor is willing to maintain a backwards-compatible fork that works for us.

If your hardware supports it, you can build yourself a LCD-CardReader+Click+encoder combination. It will enable you to realtime tune temperatures,
accelerations, velocities, flow rates, select and print files from the SD card, preheat, disable the steppers, and do other fancy stuff.
One working hardware is documented here: http://www.thingiverse.com/thing:12663
Also, with just a 20x4 or 16x2 display, useful data is shown.

*SD card folders:*

If you have an SD card reader attached to your controller, also folders work now. Listing the files in pronterface will show "/path/subpath/file.g".
You can write to file in a subfolder by specifying a similar text using small letters in the path.
Also, backup copies of various operating systems are hidden, as well as files not ending with ".g".

*SD card folders:*

If you place a file auto[0-9].g into the root of the sd card, it will be automatically executed if you boot the printer. The same file will be executed by selecting "Autostart" from the menu.
First *0 will be performed, than *1 and so on. That way, you can heat up or even print automatically without user interaction.

*Endstop trigger reporting:*

If an endstop is hit while moving towards the endstop, the location at which the firmware thinks that the endstop was triggered is outputed on the serial port.
This is useful, because the user gets a warning message.
However, also tools like QTMarlin can use this for finding acceptable combinations of velocity+acceleration.

*Coding paradigm:*

Not relevant from a user side, but Marlin was split into thematic junks, and has tried to partially enforced private variables.
This is intended to make it clearer, what interacts which what, and leads to a higher level of modularization.
We think that this is a useful prestep for porting this firmware to e.g. an ARM platform in the future.
A lot of RAM (with enabled LCD ~2200 bytes) was saved by storing char []="some message" in Program memory.
In the serial communication, a #define based level of abstraction was enforced, so that it is clear that
some transfer is information (usually beginning with "echo:"), an error "error:", or just normal protocol,
necessary for backwards compatibility.

*Interrupt based temperature measurements:*

An interrupt is used to manage ADC conversions, and enforce checking for critical temperatures.
This leads to less blocking in the heater management routine.

Implemented G Codes:
====================

*  G0  -> G1
*  G1  - Coordinated Movement X Y Z E
*  G2  - CW ARC
*  G3  - CCW ARC
*  G4  - Dwell S<seconds> or P<milliseconds>
*  G10 - retract filament according to settings of M207
*  G11 - retract recover filament according to settings of M208
*  G28 - Home all Axis
*  G90 - Use Absolute Coordinates
*  G91 - Use Relative Coordinates
*  G92 - Set current position to cordinates given

RepRap M Codes
*  M0   - Unconditional stop - Wait for user to press a button on the LCD (Only if ULTRA_LCD is enabled)
*  M1   - Same as M0
*  M104 - Set extruder target temp
*  M105 - Read current temp
*  M106 - Fan on
*  M107 - Fan off
*  M109 - Wait for extruder current temp to reach target temp.
*  M114 - Display current position

Custom M Codes
*  M17  - Enable/Power all stepper motors
*  M18  - Disable all stepper motors; same as M84
*  M20  - List SD card
*  M21  - Init SD card
*  M22  - Release SD card
*  M23  - Select SD file (M23 filename.g)
*  M24  - Start/resume SD print
*  M25  - Pause SD print
*  M26  - Set SD position in bytes (M26 S12345)
*  M27  - Report SD print status
*  M28  - Start SD write (M28 filename.g)
*  M29  - Stop SD write
*  M30  - Delete file from SD (M30 filename.g)
*  M31  - Output time since last M109 or SD card start to serial
*  M42  - Change pin status via gcode Use M42 Px Sy to set pin x to value y, when omitting Px the onboard led will be used.
*  M80  - Turn on Power Supply
*  M81  - Turn off Power Supply
*  M82  - Set E codes absolute (default)
*  M83  - Set E codes relative while in Absolute Coordinates (G90) mode
*  M84  - Disable steppers until next move, or use S<seconds> to specify an inactivity timeout, after which the steppers will be disabled.  S0 to disable the timeout.
*  M85  - Set inactivity shutdown timer with parameter S<seconds>. To disable set zero (default)
*  M92  - Set axis_steps_per_unit - same syntax as G92
*  M114 - Output current position to serial port
*  M115 - Capabilities string
*  M117 - display message
*  M119 - Output Endstop status to serial port
*  M126 - Solenoid Air Valve Open (BariCUDA support by jmil)
*  M127 - Solenoid Air Valve Closed (BariCUDA vent to atmospheric pressure by jmil)
*  M128 - EtoP Open (BariCUDA EtoP = electricity to air pressure transducer by jmil)
*  M129 - EtoP Closed (BariCUDA EtoP = electricity to air pressure transducer by jmil)
*  M140 - Set bed target temp
*  M190 - Wait for bed current temp to reach target temp.
*  M200 - Set filament diameter
*  M201 - Set max acceleration in units/s^2 for print moves (M201 X1000 Y1000)
*  M202 - Set max acceleration in units/s^2 for travel moves (M202 X1000 Y1000) Unused in Marlin!!
*  M203 - Set maximum feedrate that your machine can sustain (M203 X200 Y200 Z300 E10000) in mm/sec
*  M204 - Set default acceleration: S normal moves T filament only moves (M204 S3000 T7000) im mm/sec^2  also sets minimum segment time in ms (B20000) to prevent buffer underruns and M20 minimum feedrate
*  M205 -  advanced settings:  minimum travel speed S=while printing T=travel only,  B=minimum segment time X= maximum xy jerk, Z=maximum Z jerk, E=maximum E jerk
*  M206 - set additional homeing offset
*  M207 - set retract length S[positive mm] F[feedrate mm/sec] Z[additional zlift/hop]
*  M208 - set recover=unretract length S[positive mm surplus to the M207 S*] F[feedrate mm/sec]
*  M209 - S<1=true/0=false> enable automatic retract detect if the slicer did not support G10/11: every normal extrude-only move will be classified as retract depending on the direction.
*  M218 - set hotend offset (in mm): T<extruder_number> X<offset_on_X> Y<offset_on_Y>
*  M220 S<factor in percent>- set speed factor override percentage
*  M221 S<factor in percent>- set extrude factor override percentage
*  M240 - Trigger a camera to take a photograph
*  M280 - Position an RC Servo P<index> S<angle/microseconds>, ommit S to report back current angle
*  M300 - Play beepsound S<frequency Hz> P<duration ms>
*  M301 - Set PID parameters P I and D
*  M302 - Allow cold extrudes
*  M303 - PID relay autotune S<temperature> sets the target temperature. (default target temperature = 150C)
*  M304 - Set bed PID parameters P I and D
*  M400 - Finish all moves
*  M500 - stores paramters in EEPROM
*  M501 - reads parameters from EEPROM (if you need reset them after you changed them temporarily).
*  M502 - reverts to the default "factory settings".  You still need to store them in EEPROM afterwards if you want to.
*  M503 - print the current settings (from memory not from eeprom)
*  M540 - Use S[0|1] to enable or disable the stop SD card print on endstop hit (requires ABORT_ON_ENDSTOP_HIT_FEATURE_ENABLED)
*  M600 - Pause for filament change X[pos] Y[pos] Z[relative lift] E[initial retract] L[later retract distance for removal]
*  M907 - Set digital trimpot motor current using axis codes.
*  M908 - Control digital trimpot directly.
*  M350 - Set microstepping mode.
*  M351 - Toggle MS1 MS2 pins directly.
*  M928 - Start SD logging (M928 filename.g) - ended by M29
*  M999 - Restart after being stopped by error


Configuring and compilation:
============================

Install the arduino software IDE/toolset v23 (Some configurations also work with 1.x.x)
   http://www.arduino.cc/en/Main/Software

For gen6/gen7 and sanguinololu the Sanguino directory in the Marlin dir needs to be copied to the arduino environment.
  copy ArduinoAddons\Arduino_x.x.x\sanguino <arduino home>\hardware\Sanguino

Copy the Marlin firmware
   https://github.com/ErikZalm/Marlin/tree/Marlin_v1
   (Use the download button)

Start the arduino IDE.
Select Tools -> Board -> Arduino Mega 2560    or your microcontroller
Select the correct serial port in Tools ->Serial Port
Open Marlin.pde

Click the Verify/Compile button

Click the Upload button
If all goes well the firmware is uploading

That's ok.  Enjoy Silky Smooth Printing.

========================================================================================

Touch Screen and LCD Support:
This firmware specifies a different way to attach LCD panels to our products that use RAMPS 1.4.  In short all secondary control devices will use USART2 whether or not they have a GUI (i.e. touch screen, LCD panel, or bluetooth).  No vendors should even consider submitting a pull request that resembles the LCD hacks in the original Marlin codebase.  They should fork and set us as an upstream repo instead.