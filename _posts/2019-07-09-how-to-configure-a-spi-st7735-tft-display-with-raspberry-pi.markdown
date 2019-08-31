---
layout: post
title: How to configure a SPI (ST7735) TFT display with Raspberry Pi
date: 2019-07-03T21:30:49+02:00
tags: [raspberry pi, linux]
---

For some time I have been playing with different sensors on the Raspbarry Pi. While it is still fun (and gives me the playground for different programming tasks), I now felt it was time to see some output not only on my remote screen, but directly on the breadboard. So, I ordered a 1.8 inch display with the famous SPI [ST7735 chip](http://www.alldatasheet.com/datasheet-pdf/pdf/326213/SITRONIX/ST7735.html) controller.

Unfortunately the documentation is very thin and most of the setups only target Arduino. So, what follows is the result of my little journey to hook up the 1.8 TFT display with my Raspberry Pi.

##Pinout
First, it might help to see what board I have. Here is what the command `pinout` gave me for my system:

    ,--------------------------------.
    | oooooooooooooooooooo J8     +====
    | 1ooooooooooooooooooo        | USB
    |                             +====
    |      Pi Model 3B  V1.2         |
    |      +----+                 +====
    | |D|  |SoC |                 | USB
    | |S|  |    |                 +====
    | |I|  +----+                    |
    |                   |C|     +======
    |                   |S|     |   Net
    | pwr        |HDMI| |I||A|  +======
    `-| |--------|    |----|V|-------'
    
    Revision           : a02082
    SoC                : BCM2837
    RAM                : 1024Mb
    Storage            : MicroSD
    USB ports          : 4 (excluding power)
    Ethernet ports     : 1
    Wi-fi              : True
    Bluetooth          : True
    Camera ports (CSI) : 1
    Display ports (DSI): 1
    
    J8:
       3V3  (1) (2)  5V    
     GPIO2  (3) (4)  5V    
     GPIO3  (5) (6)  GND   
     GPIO4  (7) (8)  GPIO14
       GND  (9) (10) GPIO15
    GPIO17 (11) (12) GPIO18
    GPIO27 (13) (14) GND   
    GPIO22 (15) (16) GPIO23
       3V3 (17) (18) GPIO24
    GPIO10 (19) (20) GND   
     GPIO9 (21) (22) GPIO25
    GPIO11 (23) (24) GPIO8 
       GND (25) (26) GPIO7 
     GPIO0 (27) (28) GPIO1 
     GPIO5 (29) (30) GND   
     GPIO6 (31) (32) GPIO12
    GPIO13 (33) (34) GND   
    GPIO19 (35) (36) GPIO16
    GPIO26 (37) (38) GPIO20
       GND (39) (40) GPIO21
       
##Wireing
The TFT display I use has 8 connectors. Here is a table with their names, mapped GPIO and some additional infos:


| ID | Pin name |  GPIO | SPI0 | FB    |
|:---:|---------|------:|------|-------|
|  1 | VCC      |    5V |      |       |
|  2 | GND      |   GND |      |       |
|  3 | CS       |     8 | CE0  |       |
|  4 | RESET    |    25 |      | reset |
|  5 | A0 / RS  |    24 |      | dc    |
|  6 | SDA      |    10 | MOSI |       |
|  7 | SCK      |    11 | SCLK |       |
|  8 | LED      |    23 |      | led   |
     

Pins 2,3 and 6 will be used by SPI. Pins 1,4 and 5 will be used by the frame buffer (FB) driver. Finally, pins 7 and 8 will be used for the power supply.

Note that pin 5 may have different labels. On one TFT display it was labeled as `A0`, on another it is labeled `RS`.

Make sure you connect all SPI pins to the same bus (in this example 0). If you check the GPIO layout on [https://pinout.xyz/pinout/spi](https://pinout.xyz/pinout/spi) you will see that there are actually two SPI channels available. With the mapping above, all will be connected to SPI0.

When you have wired your display, you will also have to enable SPI through `raspi-config`. Then restart your system.

##Driver setup
Finally you have to load the driver as a kernel module. According to the above mapping, you have to load the kernel module like this:

    sudo fbtft_device name=adafruit18 gpios=reset:25,dc:24,led:23
    
There are a lot more parameters available to `fbtft_device`. To see all of them, run: `modinfo fbtft_device`. For additional information check [https://github.com/notro/fbtft/wiki/fbtft_device#parameters](https://github.com/notro/fbtft/wiki/fbtft_device#parameters). For instance the required`led` part in the `gpios` parameter is missing from the module description of `fbtft_device`. 
       
Now, run `dmsg` and check the last line. It should look something like this:

    [ 7956.283381] graphics fb1: fb_st7735r frame buffer, 160x128, 40 KiB video memory, 4 KiB buffer memory, fps=20, spi0.0 at 32 MHz
    
`graphics fb1` tells you that you will have to use `/dev/fb1` to access the display.

Try it out:

    con2fbmap 1 1
    
The first number denotes the console, e.g. `tty1`. The second number denotes the frame buffer. If all works well, you should now see a login screen on your little TFT display.

##Running JavaFX

I've also tried to run a simple JavaFX application on the display with OpenJFX. Here is what I had to run:
    
    java -Dprism.order=sw -Dmonocle.screen.fb=/dev/fb1 -jar testapp.jar
    
Because we cannot use hardware acceleration, we have to fallback to the software rendering pipeline. JavaFX will always give precedence to hardware acceleration. And though I've explicitly specified to use `/de/fb1`, without `-Dprism.order=sw` any output would still be routed to `/dev/fb0`. So, we either have to [mirror fb0](https://github.com/notro/fbtft/wiki/Framebuffer-use#framebuffer-mirroring) or explicitly use software rendering.
