---
layout: post
title: Going unstable
date: 2018-12-30T20:39:49+01:00
tags: [linux, debian]
---

So, finally I decided to go unstable with Debian. After switching from the rock solid (but partly old) stable to testing, I can only say that it works very well. Admittedly, you should do regular backups and updates. And if you do not update for months, then you might run into problems because a lot will have changed and you will have to do updates in chunks.

That said, if you are like me and run updates every day, you will rarely see any problems. And if some exist, then it is more because of changed configuration files introduces with new versions of some packages. And even then APT will tell you that there are changes and what they are.

For the last 2 years I must say the only real problems that showed up were missing packages. For instance if a new GNOME version was introduced then it happened that some packages appeared later in the testing repositories than others. The only thing I had to do was wait and try again a couple days later.

So, all in all I am quite happy with testing. Why then use unstable you may ask? The answer is simple and you can find it even in the [debian documentation](https://www.debian.org/doc/manuals/debian-faq/ch-choosing.en.html#s3.1):

*If you are a new user installing to a desktop machine, start with stable. Some of the software is quite old, but it's the least buggy environment to work in. You can easily switch to the more modern unstable (or testing) once you are a little more confident.*

*If you are a desktop user with a lot of experience in the operating system and do not mind facing the odd bug now and then, or even full system breakage, use unstable. It has all the latest and greatest software, and bugs are usually fixed swiftly.*

So, there it is. I could not describe it better :-)

# Backups, backups, backups

Even if you are (very) experienced, you should always have backups in place. Actually this holds true for every user on every system! But even more, if you use a system that is supposed to be unstable (though I think the term unstable is a uncalled for).

Anyway, backups are essential! But I prefer to have them run transparently. I do not always want to check my schedule if it is time to do a backup or have a reminder annoy me about a pending backup. I want it to run without bugging me. And when I need id, I know it is there. That's why I wrote [bcrm](https://github.com/jeansen/bcrm).

On the other hand backups are time consuming. Even if you have a backup and things go wrong you will have to restore it. So, while it is nice to have automatic backups (running in the middle of the night) it is still cumbersome when it comes to the point where you have to restore a backup. Imagine you want to try things that you know will break your system and after every try you have to restore your system ...

That's where LVM comes into the game. I grew quite fond of LVM because it allows me to create snapshots. Whenever I want to try things out, I create a snapshot. If things break, I reboot the system (thereby restoring the snapshot) and try again. That's a lot simpler than doing a full restore. It is actually as easy as you would do snapshots in VirtualBox. Because it is that easy, I always run on a snapshot. So I am sure that whenever I do a system update or upgrade, I will not loose much.

# The transition to unstable

Although I have the aforementioned setup in place, I wanted to be extra careful. So I cloned my current system into a VM and did the first test run in there. Doing things in a virtual environment is still even more comfortable, especially when it comes to multiple snapshots.

So, If you have the possibility to try things in a VM, do it. It will save you a lot of pain (and time).

Then, all I had to do was change `/etc/apt/sources.list` so it only has entries for unstable. It is only one line (or two, if you want sources, too):
    
    deb http://deb.debian.org/debian/ unstable main contrib non-free
    deb-src http://deb.debian.org/debian/ unstable main contrib non-free

As a last step I changed the running systemd configuration (runlevel) to `multi-user.target`. Here is the full command:

    systemcl isolate multi-user.target
    
That is a bit more than the `basic-target`. You may also like to use `rescue.target`. But in general `multi-user.target` should be fine.

While being root it was then nothing more than doing:

    apt udpate
    apt upgrade
    apt dist-upgrade
    
 
