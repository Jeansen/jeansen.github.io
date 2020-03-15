---
layout: post
title: LVM on LUKS with hibernation
date: 2020-03-15T13:09:02+01:00
tags: [linux, power, the real thing]
---

Oh boy, yesterday it had to happen. I knew it! It was on my todo list for ages and my laziness no earned its price. What am I talking about? Simple! I use a laptop and like any other laptop, it has a battery. Most of the time I have it attached to a power source, but then and when I don't. Yesterday I started a task, put the laptop aside and forgot about it. Later on that day I was confronted with a dead laptop. No energy, battery drained totally empty ;-(

So, today I decided to finally check this point on my todo list. The point that says: Activate suspend and hibernate on laptop.

Because I have a totally encrypted system, including the `boot` and `swap` partition, I thought it would be harder. As it turns out, it is quite simple. Here's what I had to do:

- Tell GRUB what partition device to use for hibernation.
- Tell systemd about what sleep mode to use and when.

## GRUB

You tell GRUB to resume from suspend or hibernation by adding the following to `/etc/default/grub`.

    GRUB_CMDLINE_LINUX_DEFAULT="resume=/dev/mapper/<swap-partition>"
    
You can also specify the swap partition by its UUID:
    
    GRUB_CMDLINE_LINUX_DEFAULT="resume=UUID=uuid-of-swap-partition"
    
    
## Systemd

For systemd I changed two files. The first file was `/etc/systemd/sleep.conf`. This file allows you to enable (or disable) different sleep modes. Here are my settings:

    [Sleep]
    AllowSuspend=yes
    AllowHibernation=yes
    AllowSuspendThenHibernate=yes
    AllowHybridSleep=yes
    #SuspendMode=
    #SuspendState=mem standby freeze
    #HibernateMode=platform shutdown
    #HibernateState=disk
    #HybridSleepMode=suspend platform shutdown
    #HybridSleepState=disk
    HibernateDelaySec=90min

As you can see, I simply uncommented The first four entries and therefore enabled all available sleep modes.

Then, I also changed the file `/etc/systemd/logind.conf`. Here are my settings:

    [Login]
    #NAutoVTs=6
    #ReserveVT=6
    #KillUserProcesses=no
    #KillOnlyUsers=
    #KillExcludeUsers=root
    #InhibitDelayMaxSec=5
    #HandlePowerKey=poweroff
    #HandleSuspendKey=suspend
    #HandleHibernateKey=hibernate
    
    HandleLidSwitch=suspend-then-hibernate
    
    #HandleLidSwitchExternalPower=suspend
    #HandleLidSwitchDocked=ignore
    #PowerKeyIgnoreInhibited=no
    #SuspendKeyIgnoreInhibited=no
    #HibernateKeyIgnoreInhibited=no
    #LidSwitchIgnoreInhibited=yes
    #HoldoffTimeoutSec=30s
    #IdleAction=ignore
    #IdleActionSec=30min
    #RuntimeDirectorySize=10%
    #RemoveIPC=yes
    #InhibitorsMax=8192
    #SessionsMax=8192

The only setting of interest to me was `HandleLidSwitch`, which I uncommented and set to `=suspend-then-hibernate`. Check the man page of `logind.conf`, to see all the other available settings and values!

## GNOME

I also tinkered a bit with the GNOME power settings. For instance, I wanted to have my system to hibernate when I press the power button and not to simply enter stand-by mode.

## UPower

Up to this stage the settings above were fine. Whenever I press the power button ot simply close the lid, my system will write its current state back on disk and shutdown without consuming any more power.

But what happens if I put the system aside for some longer running task and forget about it? It would drain the battery until it is empty. So, I configured [UPower](https://upower.freedesktop.org/) accordingly. With GNOME you should already have it installed. I do not know about other desktop environments, so maybe you'll have to install it separately. 

This tool allows you to define actions with respect to the laptop battery state. For instance you could tell the system to got to sleep if the estimated power supply will remain for about 15 min or if the battery is at 10% of it's capacity. The man page will tell you more about the different settings.

Here is an excerpt of my settings in `/etc/UPower/UPower.conf` that deviate from from the default:

    PercentageLow=15
    PercentageCritical=8
    PercentageAction=5
    CriticalPowerAction=Hibernate

## Additional tools

When I started my tests, hibernation worked fine. Whenever I closed the lid or pressed the power button, my laptop went into hibernation. I could resume from it and continue where I had left. Only the automatic hibernation with UPower did not work perfectly fine. When the battery reached the a critical point UPower kicked in, but it did not hibernate. My display went black and something happened on the disk but then .... nothing.

Only after I installed [tlp](https://linrunner.de/en/tlp/tlp.html) hibernation also worked as expected with UPower. In addition I also installed [thermald](https://github.com/intel/thermal_daemon).

TLP is an advanced power management tool and thermald is developed by Intel to make sure your CPU does not overheat. Even when you do not experience the same symptoms that I had to deal with, I still think one should have these two tools installed. 

Of course, there are others but these are the tools of my choice.



