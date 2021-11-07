---
layout: post
title: PipeWire with Pulseaudio - Troubleshooting
date: 2021-11-07T19:19:00+01:00
tags: [debian, linux, pulseaudio]
---

Recent updates on the unstable release branch of Debian (which is the version of Debian I use) broke my sound settings. That is, I could no longer connect my bluetooth headset. And even when I got it connected, it wasn't offered as an additional output device.

After half a night of investigating and smashing my head on the keyboard, I finally found a workaround and learned:

- PipeWire is a new technology that is going to replace Pulseaudio.
- I can have both installed.
- I cannot uninstall any of the packages like `pulseaudio` or `pipewire` because Gnome depends on them
- Package `pipewire-pulse `overwrites pulseaudio.
- Systemd service `pipewire.service` currently has access problems with `rtkit`.
- Restarting `pipewire.service` resolves the access problems but `pulseaudio.service` will have trouble loading modules.

To get my sound system up and running again, I did the following:

    systemctl stop --user pipwire.service
    systemctl stop --user pipwire.socket
    systemctl stop --user pipwire-pulse.service
    systemctl stop --user pipwire-pulse.socket

    systemctl restart --user pulseaudio.service

Of course, on a next reboot, I'll have to do it again. I could use `disable` instead of `stop`. But so far I hope it will be fixed soon, and I do not have to reboot my system that often.

