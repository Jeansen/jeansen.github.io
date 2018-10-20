---
layout: post
title: Upgrade elementaryOS Loki to Juno
date: 2018-10-19T11:59:21+02:00
tags: [linux, elementaryOS]
---

The team behind elementaryOS released version 5.0, aka Juno. But it seems there is no official upgrade path. Everywhere I searched I was confronted with that it is not possible and I would have to do a fresh install. I was shocked. A Linux system I cannot upgrade?

So, it is true there is no official upgrade path. At least, I did not find one. And there is no official tool that makes upgrading a one-click or one-command event. But under the hood elementaryOS is nothing more than Ubuntu with some additions and replacements, anyway. And as we all know Ubuntu is based on Debian which in turn __has__ a defined upgrade path. So why shouldn't the Debian upgrade path also work for all systems based on Debian?!

As it turns out, it does! And it's quite simple, actually. All you have to do is replace every occurrence of the term `xenial` with `bionic` in `/etc/apt/resources.list` and `/ect/apt/resources.list.d`. While your are on it you might also delete all those `*.save` files.

After that you simply do:
  
    sudo apt update
    sudo apt upgrade
    sudo apt dist-upgrade #This will also remove packages!
    
That's it. Reboot and your are fully upgraded. Check '/ect/os-release' and prove it to yourself!


## Really, that's it?

To be honest, I actually did a bit more. Before I upgraded the system, I installed Juno from scratch in a VM and  extracted a list of installed packages, e.g. with `dpkg --get-selections | grep -v deinstall > file_name`.

After I upgraded Loki I did the same and checked what packages from Juno's fresh install list were missing in the upgraded Loki installation:

    while read -r e; do package=$(echo $e | cut -d ' ' -f 1); grep -iq "$a" /path/to/loki_list || echo "$package"; done < /path/to/juno_list
    

From that I got the following list:

    adwaita-icon-theme-full
    btrfs-progs
    btrfs-tools
    cryptsetup-bin
    gsignond-plugin-oauth
    io.elementary.code
    jfsutils
    libcurl4:amd64
    libido3-0.1-0:amd64
    libnet-libidn-perl
    libpam-cap:amd64
    pinentry-curses
    
Quite short, isn't it? The other packages seem to come from the previous Ubuntu base. I have not encrypted my system (cryptsetup-bin) and I do not use btrfs. I also do not use IBM's journaled file system technology (jfsutils). So I think it is save to say you can ignore the list, except for the package `io.elementary.code`. This one replaces the Scratch editor and I suggest you install it.


## No Desktop Icons

elementaryOS does not allow desktop icons. That is why I installed Nautilus and followed [this article](https://elementaryos.stackexchange.com/questions/3856/how-to-enable-desktop-icons-and-right-click-in-elementary-os-freya) to have them in Loki. Unfortunately Nautilus [no longer supports desktop icons](https://csorianognome.wordpress.com/2018/08/22/desktop-icons-goes-beta/).

But you can use Nemo instead:

    sudo apt-get install --no-install-recommends nemo dconf-tools
    cp /usr/share/applications/nemo-autostart.desktop ~/.config/autostart
    sed -i '/OnlyShowIn/d' ~/.config/autostart/nemo-autostart.desktop
    
Use the dconf Editor and navigate to /org/nemo/destkop to further define what desktop icons you would like to see. In addition I recommend you deactivate auto arrangement. You can do this via the context menu available from the desktop. Simply right-click somewhere on the desktop. Open the submenu `Desktop` and  uncheck `Auto-arrange`. Now you can freely arrange your icons.


## No Tray Icons

Honestly I do not know what is going on at the moment but for whatever reason the people behind GNOME think tray (status) icons have been used as ["a crutch for far too long"](https://blogs.gnome.org/aday/2017/08/31/status-icons-and-gnome/). Personally, I utterly disagree. Those little icons somewhere in the corner (preferably the top right) are so quick to glimpse at while at the same time keep you focused on things you do. I simply cannot live without them. [And I seem to bee not the only one](https://www.reddit.com/r/gnome/comments/7x7qc6/by_what_logic_was_system_tray_removed/).

Sadly, the same is true for elementaryOS. But fear not, the fix is here and very simple. Just do the following:

    cp /etc/xdg/autostart/indicator-application.desktop ~/.config/autostart/
    sed -i '/OnlyShowIn/d' ~/.config/autostart/indicator-application.desktop
    
Reboot and enjoy!