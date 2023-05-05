---
layout: post
title: Upgrade elementaryOS
date: 2023-05-05T21:00:11+02:00
tags: [linux, elementaryOS]
---

The team behind elementaryOS released their next major release. But it seems there is no official upgrade path. Everywhere I searched I was confronted with that it is not possible, and I would have to do a fresh installation. I was shocked. A Linux system I cannot upgrade?

So, it is true there is [no official upgrade path, yet](https://github.com/orgs/elementary/projects/20). But as we all know Ubuntu is based on Debian which in turn __has__ a defined upgrade path. So why shouldn't the Debian upgrade path also work for all systems based on Debian?! Well, yes and no. While my experience is that it works quite well, it might not be the best way. Maybe it would be a better idea to [use aptik](https://github.com/elementary/wingpanel-indicator-notifications/issues/57#issuecomment-437696278). But, if you are still interested, read on!

At this point I also want to point out, that I ran through this procedure now two times. First, when I wrote this post back in October 2018 upgrading Loki to Juno and again when upgrading Juno to Odin. Therefore, I also rewrote this article to make it more generic. You will find additional notes for an upgrade from Juno to Odin at the end of this post. 

All you have to do is replace every occurrence of the term `xenial` with `bionic` in `/etc/apt/resources.list` and `/ect/apt/resources.list.d`. While you are on it, you might also delete all those `*.save` files.

After that you simply do:
  
    sudo apt update
    sudo apt upgrade
    sudo apt dist-upgrade #This will also remove packages!
    
That's it. Reboot and you are fully upgraded. Check '/ect/os-release' and prove it to yourself!


## Really, that's it?

To be honest, I actually did a bit more. Before I upgraded the system, I installed the new release from scratch in a VM and  extracted a list of installed packages, e.g. with `dpkg --get-selections | grep -v deinstall > fresh_install_list`.

After I upgraded my current system, I did the same and checked what packages from the fresh installation were missing:

    while read -r e; do package=$(echo $e | cut -d ' ' -f 1); grep -iq "$package" /path/to/upgraded_os_list || echo "$package"; done < /path/too/fresh_install_list
    

From that I got the following list (after upgrading from Loki to Juno):

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
    
Quite short, isn't it? The other packages seem to come from the previous Ubuntu base. I have not encrypted my system (cryptsetup-bin) and I do not use btrfs. I also do not use IBM's journaled file system technology (jfsutils). So I think it is safe to say you can ignore the list, except for the package `io.elementary.code`. This one replaces the Scratch editor and I suggest you install it.


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

# From Juno to Odin

'dist-upgrade' finished with an error, but I could fix this with `apt --fix-broken install`. A reboot worked totally fine. I then created a list of installed packages and compared it with the file from a fresh installation (see above). Here is, what I got:
    
    cryptsetup-bin
    cryptsetup-initramfs
    cryptsetup-run
    io.elementary.initial-setup
    jfsutils
    libpam-cap:amd64
    linux-image-5.11.0-27-generic
    linux-image-generic-hwe-20.04
    linux-modules-5.11.0-27-generic
    linux-modules-extra-5.11.0-27-generic
    xserver-xephyr

This time, I installed everything listed and ran `sudo apt autoremove` afterwards.

## Flatpak

Odin removed some applications and runtimes from the system-level installation to flatpak. Unfortunately I could not figure out how to export and import the configuration. To be on the safe side, I decided to do as much as possible from scratch so configuration settings do not get mixed up! Maybe it would have sufficed to copy over the flatpak folders. Anyway, I decided against it.

### Remotes

First, I had to add some remotes to flatpak. This took me some time to figure out. With the command `flatpak remotes --show-details` I listed the remotes on a fresh Odin installation. Here is what I got:

    Name        Title     URL                                Collection ID Subset Filter                          Priority Options         … … Homepage                         Icon
    appcenter   AppCenter https://flatpak.elementary.io/repo -             -      -                               1        system          … … https://elementary.io/           https://flatpak.elementary.io/icon.svg
    freedesktop Flathub   https://dl.flathub.org/repo/       -             -      /etc/flatpak/freedesktop.filter 1        system,filtered … … https://flathub.org/             https://dl.flathub.org/repo/logo.svg
    appcenter   AppCenter https://flatpak.elementary.io/repo -             -      -                               1        user            … … https://appcenter.elementary.io/ https://flatpak.elementary.io/icon.svg

From that list I could see which remotes were in use and which options applied (user, system). In addition, for one remote there was a filter set up. 

On a fresh installation, the following file exists `/etc/flatpak//freedesktop.filter`. It was missing on my upgraded system. So I copied it over. In addition, this file also existed in `/var/lib/flatpak/repo/`. Therefore, I had to reference it when adding the Flathub remote for Freedesktop. Here are the commands I used to add all missing remotes on my upgraded system:

    sudo flatpak remote-add --system --filter=/etc/flatpak/freedesktop.filter freedesktop https://dl.flathub.org/repo/flathub.flatpakrepo
    sudo flatpak remote-add --system appcenter https://flatpak.elementary.io/repo.flatpakrepo
    sudo flatpak remote-add --user appcenter https://flatpak.elementary.io/repo.flatpakrepo

### Applications

Now, with all remotes in place I was able to install the missing applications and runtimes. I created a nice one-liner that extracts all installed applications, their architecture and used branch. The output can then be fed to a simple installation loop:

    flatpak list --columns=application,arch,branch | tail -n +1 | sed 's/\s\+/\//g' > /path/to/app-list

After I ran the command above on a fresh Odin installation, my app-list file had the following content:

    io.elementary.Platform/x86_64/6
    io.elementary.calculator/x86_64/stable
    io.elementary.camera/x86_64/stable
    io.elementary.screenshot/x86_64/stable
    org.freedesktop.Platform.GL.default/x86_64/20.08
    org.gnome.Epiphany/x86_64/stable
    org.gnome.Evince/x86_64/stable

On my upgraded system I ran:

     while read -r e; do sudo flatpak install --system --noninteractive $([[ ! $e =~ freedesktop ]] && echo --no-auto-pin) $e; done < /path/to/app-list

To be on the save side I ran the following commands on my upgraded system and the fresh Odin installation.

    flatpak list --all
    flatpak remotes --show-details

If you followed along, make sure the commands output from both systems are equal.

Reboot and enjoy (again)!

# From Odin to Horus

So, finally Elementary OS 7, aka Horus, is here! This time, upgrading the system was the easiest, compared to the other two upgrade journeys.

Again, before I upgraded the system, I installed the new release from scratch in a VM and  extracted a list of installed packages, e.g. with `dpkg --get-selections | grep -v deinstall > fresh_install_list`.

Then, I saved `/etc/apt/sources.list`, `/etc/apt/sources.list.d/*` and replaced these files in my Odin installation

After that upgrading was as simple as:

    sudo apt update
    sudo apt upgrade
    sudo apt dist-upgrade #This will also remove packages!

Finally, I created a second list with installed packages and checked what packages from the fresh installation were missing:

    while read -r e; do package=$(echo $e | cut -d ' ' -f 1); grep -iq "$package" /path/to/upgraded_os_list || echo "$package"; done < /path/too/fresh_install_list

Two packages remained missing:

    libllvm13:amd64
    linux-image-generic-hwe-22.04

Nothing of interest. Upgrade done!

## A note on tray icons

If you run `sudo apt autoremove` it might happen that, among others, package `indicator-application` gets removed. Next time you log in, tray icons of third party applications, e.g. `owncloud` might not show. Simply install this package again, log out and back in.




