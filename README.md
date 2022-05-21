# BadUSB-USBGadget

# BadUSB and hidg0 virtual device enabler for USBGadget enabled kernels.

This is a recollection of scripts to enable the virtual /dev/hidg0 device on USBGadget enabled kernels and perform keystroke injection attacks emulating a keyboard. This tool is mostly useful in penetration testing assesments and engagements and the ideal devices to use this with would be any kind of Raspberry Pi model/any single board linux computer for that matter, and the PinePhone(Pro) or any other linux phone with a compatible kernel.
Even though those devices are the best ones to use, the tool should work on any device as long as it has USBGadget enabled.


# ¿How to check if my kernel is compatible?

To see if your kernel was compiled with USBGadget enabled, you can typically check the file "/proc/config.gz" to see your current kernel configuration.
Copy that file to a directory you like, and then gunzip it with `gunzip config.gz` and now you should have a file named simply "config". Open the file and search for the line `CONFIG_USB_GADGET=y`. If it ends in `y` it is enabled, otherwise you'll have to recompile the kernel yourself and enable it; which is beyond the scope of this repository.


# Installation

To install the scripts clone the repositoryy first and cd into it, and then simply compile usleep and hid-gadget-test:
`gcc -o hid-gadget-test.c hid-gadget-test`
`gcc -o usleep.c usleep`
Then, you can simply remove the .c files and copy everything to /usr/bin.


# Usage

The badusb script uses conventional Duckyscript, and its usage is:

`sudo mkhidg0` (you need to enable /dev/hidg0 first)
`sudo badusb duckypayload.txt`
