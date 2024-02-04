# Setup
If you're going to be using your motherboard to signal the relay, you don't need to worry about this section. Just skip to the configuration section. If you're using a Raspberry Pi that has the Klipper host software on it, then you need to do some further configuration. The first is to enable your Pi to act as an additional MCU.

## Raspberry Pi MCU
Go to the below link and follow the instructions.
https://github.com/Klipper3d/klipper/blob/master/docs/RPi_microcontroller.md
After completing that, add the following to your printer.cfg file. It adds the Pi as an additional MCU.
```
[mcu host]
serial: /tmp/klipper_host_mcu
```
The following link is a config file that shows how to use multi-MCU setups.
https://github.com/Klipper3d/klipper/blob/master/config/sample-multi-mcu.cfg

# moonraker.cfg
Add everything below here to your moonraker.cfg file. This enables moonraker to toggle your power devices from the klipper software.
```
##################################################
#     Power
##################################################
[power relay]
type: gpio
#   The type of device.  Can be either gpio, klipper_device, rf,
#   tplink_smartplug, tasmota, shelly, homeseer, homeassistant, loxonev1,
#   smartthings, mqtt or hue.
#   This parameter must be provided.
pin: gpiochip0/gpio23
initial_state: on
#    The state the power device should be initialized to.  May be on or
#    off.  When this option is not specifed no initial state will be set.
off_when_shutdown: False
#   If set to True the device will be powered off when Klipper enters
#   the "shutdown" state.  This option applies to all device types.
#   The default is False.
off_when_shutdown_delay: 0
#   If "off_when_shutdown" is set, this option specifies the amount of time
#   (in seconds) to wait before turning the device off. Default is 0 seconds.
on_when_job_queued: False
#   If set to True the device will power on if a job is queued while the
#   device is off.  This allows for an automated "upload, power on, and
#   print" approach directly from the slicer, see the configuration example
#   below for details. The default is False.
locked_while_printing: False
#   If True, locks the device so that the power cannot be changed while the
#   printer is printing. This is useful to avert an accidental shutdown to
#   the printer's power.  The default is False.
restart_klipper_when_powered: True
#   If set to True, Moonraker will schedule a "FIRMWARE_RESTART" to command
#   after the device has been powered on. If it isn't possible to immediately
#   schedule a firmware restart (ie: Klippy is disconnected), the restart
#   will be postponed until Klippy reconnects and reports that startup is
#   complete.  Prior to scheduling the restart command the power device will
#   always check Klippy's state.  If Klippy reports that it is "ready", the
#   FIRMWARE_RESTART will be aborted as unnecessary.
#   The default is False.
restart_delay: 1.
#   If "restart_klipper_when_powered" is set, this option specifies the amount
#   of time (in seconds) to delay the restart.  Default is 1 second.
bound_services:
#   A newline separated list of services that are "bound" to the state of this
#   device.  When the device is powered on all bound services will be started.
#   When the device is powered off all bound services are stopped.
#
#   The items in this list are limited to those specified in the allow list,
#   see the [machine] configuration documentation for details.  Additionally,
#   the Moonraker service can not be bound to a power device.  Note that
#   service names are case sensitive.
#
#   When the "initial_state" option is explcitly configured bound services
#   will be synced with the current state.  For example, if the initial_state
#   is "off", all bound services will be stopped after device initialization.
#
#   The default is no services are bound to the device.

#object_name: output_pin my_pin
#    The Klipper object_name (as defined in your Klipper config).  Valid examples:
#      output_pin my_pin
#      gcode_macro MY_MACRO
#    Currently, only `output_pin` and `gcode_macro` Klipper devices are
#    supported.  See the note below for macro restrictions. Keep in mind that
#    the object name is case sensitive.  This parameter must be provided.
#timer: this might be good to run a filtration device
#    A time (in seconds) after which the device will power off after being.
#    switched on. This effectively turns the device into a  momentary switch.
#    This option is available for gpio, klipper_device, tplink_smartplug,
#    shelly, and tasmota devices.  The timer may be a floating point value
#    for gpio types, it should be an integer for all other types.  The
#    default is no timer is set.
```
# macros.cfg or printer.cfg
Place these in either your macros.cfg file (with an [include macros.cfg] in your printer.cfg) or just add them to your printer.cfg file. These are the macros that will call on moonraker to toggle your power device when your printer has been idle for a certain amount of time.
```
##################################################
#     Power Macros
##################################################
# Description: This section has a macro that calls moonraker to toggle our relay, a macro for delayed gcode, and a macro to call the delayed gcode when our printer enters the state "idle"

[gcode_macro POWER_OFF_PRINTER]
# Description: This macro asks moonraker to toggle our "relay" state to "off"
gcode:
  {action_call_remote_method(
    "set_device_power", device="relay", state="off"
  )}
# Comments:
# action_call_remote_method is used to call a method registered by a remote client (moonraker)
# "set_device_power" is the method we want to call from the moonraker.cfg
# "device" is the name of the deivce defined in moonraker.cfg
# "state" is the state of the relay, on, off, or toggle

  
[delayed_gcode delayed_printer_off]
# Description: This macro calls the "POWER_OFF_PRINTER" macro once the "printer" object is in state "idle"
initial_duration: 0.
gcode:
  {% if printer.idle_timeout.state == "Idle" %}
    POWER_OFF_PRINTER
  {% endif %}
# Comments:
# "delayed_gcode" is a way to execute gcode after a set delay
# "initial_duration"
#   The duration of the initial delay (in seconds). If set to a
#   non-zero value the delayed_gcode will execute the specified number
#   of seconds after the printer enters the "ready" state. This can be
#   useful for initialization procedures or a repeating delayed_gcode.
#   If set to 0 the delayed_gcode will not execute on startup.
#   Default is 0.
# "POWER_OFF_PRINTER" calls the "POWER_OFF_PRINTER" macro defined above


[idle_timeout]
# Description: This macro is called when the printer goes into an idle state and calls the previous delayed_gcode. It updates the delay to the duration defined below.
# After 10 minutes (600seconds) of being idle, the gcode below will be called.
gcode:
  M84
  TURN_OFF_HEATERS
  UPDATE_DELAYED_GCODE ID=delayed_printer_off DURATION=600
# Comments:
# "idle_timeout" is a module that tracks the "state" of the printer. The "state" can be "Idle", "Printing", or "Ready"
# "M84" turns off the stepper motors
# "TURN_OFF_HEATERS" turns of the heaters
# "UPDATE_DELAYED_GCODE"
#   UPDATE_DELAYED_GCODE [ID=<name>] [DURATION=<seconds>]:
#   Updates the delay duration for the identified [delayed_gcode] and starts the timer for gcode execution.
#   A value of 0 will cancel a pending delayed gcode from executing.
```
# end gcode
Add the below line to your end gcode in your slicer. This will call the delayed gcode function when your printer finishes. You could also add it to your "end_print" macro if you have one. I added it to my slicer so that I could change between the options without messing with my config files.
```
delayed_printer_off
```
