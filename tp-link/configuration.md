# Setup
Setup for the tp-link plug is pretty straight forward. You need to drop some configuration into your moonraker.cfg file, your printer.cfg file, and your macros.cfg file if you have one.

# moonraker.cfg
Add the below to your moonraker.cfg file. Make sure to add the IP of your tp-link plug to its field.
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

# printer.cfg
Add the below to your printer.cfg file. If you have a macros.cfg file, you can move the macros in there. Make sure that the idle_timeout stays in the printer.cfg file.
```
```
