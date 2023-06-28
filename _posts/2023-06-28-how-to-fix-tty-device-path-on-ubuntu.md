---
title: "Using fixed tty device path on Ubuntu"
last_modified_at: 2023-06-28T09:08
categories:
  - Tech
tags:
  - Linux
  - Embedded
---

# Why fix the tty device path?
Usually you don't need this but if you are working on a project that uses embedded device with some peripherals, it's nice to have fixed device path since `/dev/ttyUSB0` or `/dev/ttyUSB1` can be any device.

Wouldn't it be better if we have `/dev/ttyMyPort0` always be the device 0 and `/dev/ttyMyPort1` be the device 1?

# How do I do this
## Finding the unique attribute
We need to tell Linux that a specific device must go to `/dev/ttyMyPort0`. And the specific device can be distinguished by following command.

`udevadm info -a -n /dev/ttyUSB0`

```shell
user@ubuntu:~$ udevadm info -a -n /dev/ttyUSB0

Udevadm info starts with the device specified by the devpath and then
walks up the chain of parent devices. It prints for every device
found, all possible attributes in the udev rules key format.
A rule to match, can be composed by the attributes of the device
and the attributes from one single parent device.

  looking at device '/devices/3610000.xhci/usb1/1-2/1-2.1/1-2.1:1.0/ttyUSB0/tty/ttyUSB0':
    KERNEL=="ttyUSB0"
    SUBSYSTEM=="tty"
    DRIVER==""

  looking at parent device '/devices/3610000.xhci/usb1/1-2/1-2.1/1-2.1:1.0/ttyUSB0':
    KERNELS=="ttyUSB0"
    SUBSYSTEMS=="usb-serial"
    DRIVERS=="ftdi_sio"
    ATTRS{latency_timer}=="16"
    ATTRS{port_number}=="0"

  looking at parent device '/devices/3610000.xhci/usb1/1-2/1-2.1/1-2.1:1.0':
    KERNELS=="1-2.1:1.0"
    SUBSYSTEMS=="usb"
    DRIVERS=="ftdi_sio"
    ATTRS{authorized}=="1"
    ATTRS{bAlternateSetting}==" 0"
    ATTRS{bInterfaceClass}=="ff"
    ATTRS{bInterfaceNumber}=="00"
    ATTRS{bInterfaceProtocol}=="ff"
    ATTRS{bInterfaceSubClass}=="ff"
    ATTRS{bNumEndpoints}=="02"
    ATTRS{interface}=="Quad RS232-HS"
    ATTRS{supports_autosuspend}=="1"

  ...
  ...
  ...
```

This will tell us device information such as which port is connected to the device. Try this command with different device paths. You will see some parameters are different ones from the another.
In my case, I usually use `KERNELS==1-2.1:1.0` which tells me that this device is connected to `1-2.1:1.0` physical port. 

## Tell Linux to use fixed path that I want
Once we have a unique attribute of the device, it's time to tell Linux that I want to use this device to be this path. Open `/etc/udev/rules.d/99-usb-serial.rules` with root permission and add something like below.

```shell
KERNEL=="ttyUSB*",KERNELS=="1-2.1:1.0",SYMLINK+="ttyTelem"
```

`KERNEL` and `KERNELS` are the unique attributes for `/dev/ttyUSB0` device. And `SYMLINK` tells Linux to make symbolic link of `/dev/ttyUSB0` to `/dev/ttyTelem`.
You can add more attributes prior to `SYMLINK` to make better separation.

Reboot your system or use below commands to apply these changes immediately.

```shell
sudo udevadm control --reload-rules
sudo udevadm trigger
```

See if `/dev/ttyTelem` exists and it is linked to `/dev/ttyUSB0`.
If it doesn't show up, the unique attributes might not be unique.

