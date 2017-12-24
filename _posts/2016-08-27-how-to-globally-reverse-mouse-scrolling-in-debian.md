---
layout: post
title: How to Globally Reverse Mouse Scrolling in Debian
tags: [linux, gnome]
date: 2016-08-27T13:18:28+02:00
---

Recently I faced the problem that after applying some updates (I cannot remember if it was X11 or Gnome) the vertical scrolling direction got inverted. That is, scrolling down in a PDF document with the mouse wheel did not move a page down, but a page up! Unfortunately this behaviour affected every application ...

All settings I found - from Gnome settings, following xmodmap, udev and others - only solved the problem partially or did not solve it at all. With partially solving I mean that it either solved the problem only for applications not written with the GTK toolkit ore vice-versa.

Finally I manged to solve the problem by simply adding a file to my autostart folder for Gnome that triggers `xinput` to set the natural scrolling to zero. To find the right settings I had to do the following:

First I ran `xinput list` to get a list of all available input devices. Here is what it looks like on my virtual machine:

    ⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
    ⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
    ⎜   ↳ VirtualBox mouse integration            	id=9	[slave  pointer  (2)]
    ⎜   ↳ ImExPS/2 BYD TouchPad                   	id=11	[slave  pointer  (2)]
    ⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
        ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
        ↳ Power Button                            	id=6	[slave  keyboard (3)]
        ↳ Sleep Button                            	id=7	[slave  keyboard (3)]
        ↳ Video Bus                               	id=8	[slave  keyboard (3)]
        ↳ AT Translated Set 2 keyboard            	id=10	[slave  keyboard (3)]

From this I took the information that the id of the relevant mouse pointer was 11. Knowing the id I could now run `xinput --list-props 11` which presented me with a list of available settings, their current value as well as their default value(s).

    Device 'ImExPS/2 BYD TouchPad':
        Device Enabled (119):	1
        Coordinate Transformation Matrix (121):	1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        libinput Accel Speed (272):	0.000000
        libinput Accel Speed Default (273):	0.000000
        libinput Accel Profiles Available (274):	1, 1
        libinput Accel Profile Enabled (275):	1, 0
        libinput Accel Profile Enabled Default (276):	1, 0
        libinput Natural Scrolling Enabled (262):	1
        libinput Natural Scrolling Enabled Default (263):	0
        libinput Send Events Modes Available (242):	1, 0
        libinput Send Events Mode Enabled (243):	0, 0
        libinput Send Events Mode Enabled Default (244):	0, 0
        libinput Left Handed Enabled (264):	0
        libinput Left Handed Enabled Default (265):	0
        libinput Scroll Methods Available (266):	0, 0, 1
        libinput Scroll Method Enabled (267):	0, 0, 0
        libinput Scroll Method Enabled Default (268):	0, 0, 0
        libinput Button Scrolling Button (269):	2
        libinput Button Scrolling Button Default (270):	274
        libinput Middle Emulation Enabled (277):	0
        libinput Middle Emulation Enabled Default (278):	0
        Device Node (245):	"/dev/input/event1"
        Device Product ID (246):	2, 6
        libinput Drag Lock Buttons (271):	<no items>
        libinput Horizonal Scroll Enabled (247):	1


Now, what I had to change was the setting for __Natural Scrolling Enabled (262)__. This was set to 1 and I had to set it to 0. Finally, the command I had to run was: 

    xinput set-prop 11 262 0
    
And since I did not want to run this command every time I log in or restart my system, I simply put it in a [startup file](https://developer.gnome.org/integration-guide/stable/desktop-files.html.en) for Gnome. Here is the content of that file which I put into `~/.config/autostart/mouseinvert.desktop`

    #!/usr/bin/env xdg-open
    [Desktop Entry]
    Version=1.0
    Type=Application
    Terminal=true
    Icon[en_US]=gnome-panel-launcher
    Name[en_US]=mouseInvert
    Exec=bash -c 'xinput set-prop 11 262 0'
    Name=mouseInvert
    Icon=gnome-panel-launcher

To apply this system wide for all users you should put this file into `/etc/xdg/autostart`.

The above startup folder should work for Unity, GNOME, XFCE and LXDE. For KDE use `~/.kde/Autostart/` or `/usr/share/autostart/`.

## Supplement
It might happen that numbers from `xinput --list-props` change. That's exactly what happened to me after today's 
update of some Xorg libraries: `libinput Natural Scrolling Enabled (262)` changed to `libinput Natural Scrolling Enabled (261)` and value 262 now pointed to `libinput Natural Scrolling Enabled Default (262)`. Consequently calling `xinput set-prop 11 262 0` changed the wrong value and even raised an error like the following:

    X Error of failed request:  BadAccess (attempt to access private resource denied)
    Major opcode of failed request:  131 (XInputExtension)
    Minor opcode of failed request:  57 ()
    Serial number of failed request:  19
    Current serial number in output stream:  20
    


