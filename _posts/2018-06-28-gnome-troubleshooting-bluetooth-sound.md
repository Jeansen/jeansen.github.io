---
layout: post
title: GNOME troubleshooting bluetooth sound
tags: [gnome, pulseaudio, bluetooth, linux, debian]
date: 2018-06-28T23:03:51+02:00
---

This is going to be short but hopefully helpful. Today I tried connecting my sound bar and headphones via bluetooth. With GNOME this should be just a matter of some clicks. And it almost was. Pairing and connecting worked like a charm. But I was not able to select the new audio devices for playback. Restarting pulse audio did not work either.

Obviously with GDM another instance of PulseAudio is started, which "captures" bluetooth device connections. [But I found a solution in the archlinux wiki that worked for me](https://wiki.archlinux.org/index.php/Bluetooth_headset#Connecting_works.2C_but_I_cannot_play_sound). It was a mere matter of creating a symlink that simply masks the pulseaudio socket for the GDM user:

    sudo -uDebian-gdm mkdir -p  /var/lib/gdm3/.config/systemd/user
    sudo -uDebian-gdm ln -s /dev/null /var/lib/gdm3/.config/systemd/user/pulseaudio.socke

I then restarted my system and was now able to select the new bluetooth devices for output.

Note, that you might need to change the user. On Debian it is 'Debian-gdm' but on other distributions it might be something else. In addition the part of '.../gdm3/...' in the paths above might be simply 'gdm'.

Good luck!