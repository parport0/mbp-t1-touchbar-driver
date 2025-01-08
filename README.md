# Import of Ronald Tschalär's Macbook T1 drivers for the touchbar, ambient light sensor, and webcam, for kernel 6.8.0

Original posting: https://lore.kernel.org/lkml/20190612083400.1015-1-ronald@innovation.ch/

Updated for kernel 6.8.0, with a Makefile for out-of-tree building.

## Building 

To build, run `make`. To install, run `make install`.

* apple-ib-als is the module for the ambient light sensor.
* apple-ib-tb is the module for the touchbar.
* apple-ibridge is the module for iBridge, the other two depend on this one.

Make sure before installation that your `lsusb` shows:

```
Bus 001 Device 002: ID 05ac:8600 Apple, Inc. iBridge
```

and *not*

```
Bus 001 Device 003: ID 05ac:1281 Apple, Inc. Apple Mobile Device [Recovery Mode]
```

The second output means your T1 is in recovery mode, maybe its firmware is missing.

## Usage

When testing, after modprobing:

```
sudo bash -c 'echo -n "1-3" > /sys/bus/usb/drivers/usb/unbind'
sudo bash -c 'echo -n 1-3 > /sys/bus/usb/drivers_probe'
sudo bash -c 'echo -n "1-3:1.2" > /sys/bus/usb/drivers/usbhid/unbind'
sudo bash -c 'echo -n "1-3:1.3" > /sys/bus/usb/drivers/usbhid/unbind'
sudo bash -c 'echo -n 1-3:1.2 > /sys/bus/usb/drivers_probe'
sudo bash -c 'echo -n 1-3:1.3 > /sys/bus/usb/drivers_probe'
```

When installed:

```
sudo modprobe apple-ib-tb
sudo bash -c 'echo -n "1-3" > /sys/bus/usb/drivers/usb/unbind'
sudo bash -c 'echo -n 1-3 > /sys/bus/usb/drivers_probe'
```

This driver is strange because it creates four virtual HID devices.

If `lsusb -t` does not show four interfaces appearing on 05ac:8600, then apple-ibridge.ko didn't do its job right.

If the devices are there, but the touchbar still doesn't light up, try to restart the laptop...

Why are those unbind + probe needed? I needed them because otherwise the apple-ibridge driver is not sticking to the /sys/bus/usb/devices/1-3 device.

Maybe it is because the generic "usb" driver steals the device? There was a patch to implement some "anti-stealing" functionality for usb_driver drivers, but I am not knowledgeable enough to say for sure why it does not work in this case. https://patchwork.kernel.org/project/linux-usb/patch/20200727104644.149873-3-hadess@hadess.net/ My suspicion is that the apple-ibridge driver needs to have a usb_driver entry as well. It only has a HID driver, which also kinda manages the usb_device.
