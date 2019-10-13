---
layout: post
title: From Raspbian Stretch to Buster
date: 2019-10-01T17:42:37+02:00
tags: [raspberry pi, linux]
---

Well, it's some time now since the Raspberry Pi foundation released all new and shiny Raspbian image based on Debian Buster. Unfortunately, they do not support any upgrade path. But it is understandable they do not "support" it. This little SoC is probably to much tweaked anyways so nobody knows what users might have configured.

But, for simple setups you can still try and follow the standard Debian upgrade procedures. So did I. And here is a little journey of what happened to me or better say, what I should have known before I started the distribution upgrade.

So, my take away of a half-days upgrade journey with Raspian light is this:

- Uninstall aufs-dkms and docker. You can reinstall it afterwards, but it will simply fail to compile
- Stop dphys-swafile service and remove /var/swap. It will be created again after a reboot

The rest just worked like a charm. Then and when I had to make a decision if I wanted to keep or replace a configuration file, but that's about it.

If you have the desktop version of Raspbian installed, you'll have to remove the following packages:

    sudo apt purge timidity lxmusic gnome-disk-utility deluge-gtk evince wicd wicd-gtk clipit usermode gucharmap gnome-system-tools pavucontrol
    
There is also a nice [blog post](https://www.raspberrypi.org/blog/buster-the-new-version-of-raspbian/) on raspberrypi.org about the new features and some general information about a possible upgrade procedure.

## The actual upgrade

The steps involved are simple.

1. Modify `/etc/apt/source.list` and `/etc/apt/sources.list.d/raspi.list` file and replace `stretch` with `buster`.
2. Then as root, run the following commands:
    
        apt update
        apt upgraade
        apt dist-upgrade
        apt autoremove -y
        apt autoclean
        
Some people also remove the `apt-listchanges` package to save time. But I believe, it depends on your installation (the amount of packages) and if you really care about 5 minutes more or less ;-)
        

    

    

