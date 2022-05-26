# PineBunny

# An attempt to implement something similar to what a Bash Bunny does on the PinePhone (Pro)
### WARNING! Very Alpha state, right now it works only manually. Automation will be added in the future

This is a recollection of scripts to enable the virtual /dev/hidg0 device on USBGadget enabled kernels and perform keystroke injection attacks emulating a keyboard. This tool is mostly useful in penetration testing assesments and engagements and the ideal devices to use this with would be any kind of Raspberry Pi model/any single board linux computer for that matter, and the PinePhone (Pro) or any other linux phone with a compatible kernel.
Even though those devices are the best ones to use, the tool should work on any device as long as it has USBGadget enabled.


# ¿How to check if my kernel is compatible?

To see if your kernel was compiled with USBGadget enabled, you can typically check the file "/proc/config.gz" to see your current kernel configuration.
Do `gunzip -c /proc/config.gz | grep CONFIG_USB_GADGET=` and check if it ends in `y`. If not you'll have to recompile the kernel yourself and enable it; which is beyond the scope of this repository.


# Installation

To install the scripts clone the repositoryy first and cd into it, and then simply compile usleep and hid-gadget-test:
`gcc -o hid-gadget-test.c hid-gadget-test`
`gcc -o usleep.c usleep`
Then, you can simply remove the .c files and copy the binaries to /usr/bin.
Keep in mind you'll need a different 'badusb' binary for each keyboard layout, so copy the one for your keyboard like so:
`sudo cp badusb-[YOUR_LAYOUT] /usr/bin/badusb`

Only en_US and es_ES are supported right now, I'll work on more layouts when I get the time.


# Usage

The badusb script uses conventional Duckyscript, and its usage is:

`sudo mkhidg0` (you need to enable /dev/hidg0 first, tho mkhidg0 also enables the ecm module of USB Gadget, so you'll have ethernet over USB, too. Just keep in mind that your target won't get an IP address automatically assigned without a proper dhcp server running. I personally like to use dnsmasq)
`sudo badusb duckypayload.txt`
