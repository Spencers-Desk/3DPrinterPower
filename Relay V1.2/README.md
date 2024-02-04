# Relay V1.2
The Relay V1.2 from BIGTREETECH is a small board with a relay on it. I wired it inbetween the power switch and power supply unit (PSU) of my printer. This means that I could mechanically stop all power coming from the outlet to the printer's power supply if I wanted to. While it requires more effort, supplies, and some danger (working with mains voltage), it has some advantages over using a smart plug.

Unfortunately, the documentation (in my opinion) is very poor. The wiring, and especially the Klipper integration. But, I did some websurfing and compiled everything I did to make it work.

## Functions
The three main functions of the Relay V1.2 are Short Circuit Detection, Power Control, and Power Outage Detection. From my experience (not comprehensive) if the Relay V1.2 Power Control header is not connected to your motherboard, the relay will power off after a few seconds of being on. I'm not sure if it HAS to be the Power Control header, or if any header will do.

### Short Circuit Detection
This feature allows the relay to cut power to your printer if a short is detected. It requires you to add a jumper to enable it, shown in the picture, and to wire the respective header to a 5V pin and a GND pin on your motherboard (assuming you want to detect short circuits on your board). This feature isn't going to save whatever is going to be shorted as your PSU takes a few seconds to shutdown (capacitors take time to discharge). What it will do, is remove the electricity flowing through a short circuit which could prevent a fire. I plan to use this feature. I mean, why wouldn't you add another layer of protection?

### Power Control
This feature allows you to set the relay to an "off" state, meaning it will not let electricity flow through it to the PSU. This is the header you'll be using if you want to cut power to your printer using Klipper whether you setup a macro to do it after a print finishes or if you want to manually toggle it on your web interface's dashboard. This header only allows you to turn off the relay. It does not allow you to turn it on. To get the relay to switch back on, you need to short the RST (reset) pins together. I did this using a manual switch. You could use another relay to toggle it via klipper, but I did not choose to do this.

### Power Outage Detection
This feature is a little confusing. I'm not quite sure what the purpose of it is. If there was a power outage, power to the relay would be cut off, while the capacitors in the PSU would still power the motherboard for a few seconds. So, it's strange. It could be that the relay is the one signalling the motherboard instead. If this is the case, I imagine the relay would lose power, signal the motherboard, which would activate a macro and an uninterruptible power supply (UPS) to safely pause the print before fully shutting off.
