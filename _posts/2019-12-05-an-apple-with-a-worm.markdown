---
layout: post
title: An apple with a worm
date: 2019-12-05T21:49:37+01:00
tags: [linux, mac, the real thing]
---

For some days now I've had the "honor" to use an all new and shiny Mac Book pro (and will have to use it for a while longer ...). Though I've become fond of Linux, I always try to be open to new things. And after all, it's Unix under the hood. But after a couple of days, I cannot really say I became friends with MacOS. To be fair, that is not to blame Apple or Unix. It's a compound of orchestrated errors and incompatibilities. Some in hardware, some in the OS, others in the lack of application features or simply of manufactures not providing the needed support.

What follows is my little journey of pain to get things to work the way I am used to working (as close as possible).

## The escape plan

So, with the new system at hand I knew it would take me some time to get used to it. Unfortunately I will not have the luxury to play around for weeks and months until I am satisfied and "production ready". That said, I wanted to have an "escape plan". My idea: I clone my beloved Debian system to a virtual disk, prepare everything in Virtual Box and then put it an an external Disk. With USB 3.1 and an enclosure capable of utilizing Nvme, I would have all the performance I need without filling up the internal disk.

But, I needed to be able to mount the external disk on Linux and MacOS. And that's where the hassle began. `extFat` was the obvious choice to go for regarding the file system. Unfortunately (thanks to some patent issues) it is not part of the Linux Kernel and only runs in user land. That makes it insanely slow! `HFS+` on the other hand can be used with `hfsprogs`. But you'll have to disable journalling. That is, I had to connect the external disk to my Mac Book and use 'disk util' to format it with HFS+. Afterwords I ran `diskutil disableJournal /Volumes/<name>` in the terminal and disabled journaling.

If you do not disable journaling you'll be able to mount the disk, but whatever you do it will only be mounted read-only. The same is true when the disk wasn't properly unmounted. In the latter case you'll have to run `sudo fsck.hfsplus -f /dev/sdX` first.

With journaling disabled and the disk properly dismounted, I was then able to mount the disk on Linux with `sudo mount -t hfsplus -o force,rw /dev/sdX /mnt/<some-name>` and copy over my virtual disk image.

Unfortunately (for reasons I could not figure out, yet) the external disk was sporadically ejected. I've tried all sorts of settings, cables, ports, adapters but nothing seemed to help. My external disk always got "spat out". Luckily I had about 500GiB available on the Mac Book's internal drive and therefore decided to copy over the virtual image to the Mac Book's drive. It's not a solution on the long run, but a workaround for the moment.

## Virtual crash dummy

With the virtual disk in place it was Virtual Box that made me pull my hairs next. After I had installed the latest guest additions, I could no longer log in. The virtual would start but only show me a grey GDM screen. Even sending ACPI shutdown did not have any effect nor did  my attempts with different display settings  and uninstalling the guest additions did not work, either.

From previous experiences and reports I read on the Internet I knew that GDM can be "picky" and problematic. So, I booted into recovery mode, installed `lightdm` and reconfigured it with `dpkg-reconfigre lightdm` to be the default. After that I was able to se a login screen again and could login. 

Next, the ALT key did not work. For instance I could not use ATL-TAB to switch windows in the VM. For this, I had to tell [XQuartz](https://de.wikipedia.org/wiki/XQuartz) to send ALT keys, instead of the Modifier keys. I achieved this when I ran `xev` in the terminal (on the host). You will then also have a window for XQuartz and can change the settings via the main menu bar.

Depending on the display (internal or external), I also had to try different scale modes from the view menu. The internal display seems to behave very differently than external displays ...

Thinking that now everything should be fine I was quickly taught otherwise. In the middle of a session my Mac Book just restarted with a kernel panic. This happened again and again, every time I started up a virtual machine and worked in it for some minutes. The solution - as it seems - is to make sure that VirtualBox has [full disk access and revoked permission from input monitoring and accessibility](https://carleton.ca/scs/2019/virtualbox-crashing-on-mac-after-update/)

## Human Input Devices 

Now, with the software issues resolved, there are the hardware issues. My older Logitech Trackball M570 does not seem to be supported by Logitech's own products, namely [Options](https://www.logitech.com/en-roeu/product/options) and [Control Center](https://support.logi.com/hc/de/articles/360025297833-Logitech-Control-Center). I wasn't able to assign any function key. My newer model, Ergo MX, on the other hand is recognized just fine. 

My Logitech Keyboard K800 wasn't recognized at all and I've not been able to use the F-keys properly. It's a pity, but after a while I gave up and connected an original external Apple keyboard. I then switched the Option and Command keys and activated the F-keys as default (you can do both easily in the keyboard settings).

## Conclusion

Don't get me wrong, I do not want to put all - if any - blame on Apple. It's just that all things that should work simply did not work or have some annoying constraints. It feels like in my early days with Linux when everything in Windows just worked out of the box and in Linux you had to dis- and reassemble everything first. 






    
