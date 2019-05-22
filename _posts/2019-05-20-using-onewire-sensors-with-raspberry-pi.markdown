---
layout: post
title: Using Onewire sensors with Raspberry Pi
date: 2019-05-20T11:04:19+02:00
tags: [raspberry pi, linux]
---


A couple days ago I ordered my first sensor for measuring temperature. I thought this could be a good start to monitor my little data center I neatly installed in a cupboard. Although I have installed an external fan so the risk of heat accumulation is quite low, the fan could fail and then heat would build up.

The idea now is to simply install a temperature sensor and constantly read it's value. If a threshold is reached, a warning should be sent.

Therefore the first task was to get the temperature sensor attached and be able to read from it. The sensor I have selected is a so called 'onewire' sensor (DS1820). These are sensors using the One wire (1-wire) bus for communication[^1].

During my research I found a lot of different tutorials explaining how to get this sensor working. Some of them are obsolete, others simply do too much.

The best source I could find was from the Raspberry Pi foundation explaining how the device tree (DT) mechanism introduced with kernel version 4.4 works[^2].

To summarize: Since kernel version 4.4 all optional devices are deactivated. There are no blacklists any longer. Most of the time, for instance with HATs (Hardware Attached on Top), required overlays will be activated automatically without any user intervention. 

But with some devices you will have to tell the kernel to load the right modules manually. The Onewire interface (w1-gpio) is such a device. You have two options to activate this overlay, which in turn will then have the kernel load the necessary modules: You can either load the overlay manually, at runtime. Or, you have the system load it automatically during the boot process.


## Loding overlays at runtime

To load the Onewire overlay, run the following command:

    dtoverlay w1-gpio


## Loading overlays during boot

To load the Onewire overlay whenever you start the system, just add the following entry to `/boot/config.txt`

    dtoverlay=w1-gpio


## Finding overlays

All available overlays can be listed with `dtoverlay -a`. More information can be obtained when by reading the README file located at `/boot/overlays/README`. This folder also holds all the device tree files listed with the former command.


## Reading data from the 1-wire bus

When all went well, running `lsmod | grep w1` should give you the following output:

    w1_therm               16384  0
    w1_gpio                16384  0
    wire                   45056  2 w1_gpio,w1_therm

You should now have a new device in `/sys/bus/w1/devices`. For instance, on my system `ls /sys/bus/w1/devices` gives me this output:

    28-011f63070002  w1_bus_master1

And `cat /sys/bus/w1/devices/28-011f63070002/w1_slave` finally presents me with the raw data from the sensor:

    45 01 11 f0 7f ff 0b 10 b6 : crc=b6 YES
    45 01 11 f0 7f ff 0b 10 b6 t=20312
    
The value of interest here is `t=20312`. This is the temperature in micro degrees Celsius. So, the current temperature is `20.312 °C`



[^1]: https://en.wikipedia.org/wiki/1-Wire
[^2]: https://www.raspberrypi.org/documentation/configuration/device-tree.md
