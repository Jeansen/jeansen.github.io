---
layout: post
title: Windows 11 with WSL2 in Qemu/Libvirt
date: 2022-12-17T23:32:59+01:00
tags: [kvm, libvirt, qemu, linux]
---

So, as I wrote in a [my previous post]({{ site.baseurl }}{% post_url 2022-12-13-mapping-copy-paste-in-xterm-and-rxvt-unicode %}), I now own an all new shiny power laptop and almost anything works as before. That's the beauty of running Linux on hardware that is intended for Linux systems. You take out the hard disk and simply put it into a new machine. Boot it up, and that's it. Well almost. Admittedly, I had to do some Desktop tweaking because the scaling of fonts made my Desktop environment use a magnifier.

Another thing that stopped working was passing through the CPU to my Windows 11 virtual machine. It worked on the older Laptop, with a generation 8 Intel CPU, but no more on the new one machine with a generation 12 Intel CPU. I am not sure, why. On the other hand, I found a working solution without using the host CPU directly.

At this point I expect you to know how to configure and install KVM, Qemu, Libvirt etc and I won't go into all the details. But if you came along because you haven't found any way to get Windows 11 working with WSL2 in a Qemu VM, then the snippet below might help.

Simply pick the XML snippet below and try it out in `virt-manager`:

    <domain type="kvm">
      <memory unit="KiB">8388608</memory>
      <currentMemory unit="KiB">8388608</currentMemory>
      <vcpu placement="static">8</vcpu>
      <os>
        <type arch="x86_64" machine="pc-q35-7.2">hvm</type>
        <loader readonly="yes" secure="no" type="pflash">/usr/share/OVMF/OVMF_CODE_4M.secboot.fd</loader>
        <nvram>/var/lib/libvirt/qemu/nvram/win10_VARS.fd</nvram>
        <boot dev="hd"/>
      </os>
      <features>
        <acpi/>
        <apic/>
        <hyperv mode="custom">
          <relaxed state="on"/>
          <vapic state="on"/>
          <spinlocks state="on" retries="8191"/>
        </hyperv>
        <vmport state="off"/>
        <smm state="on"/>
      </features>
      <cpu mode="custom" match="exact" check="partial">
        <model fallback="allow">Broadwell-noTSX-IBRS</model>
        <topology sockets="2" dies="1" cores="2" threads="2"/>
        <feature policy="disable" name="hypervisor"/>
        <feature policy="require" name="vmx"/>
      </cpu>
      <clock offset="utc"/>
      <on_poweroff>destroy</on_poweroff>
      <on_reboot>restart</on_reboot>
      <on_crash>destroy</on_crash>
    </domain>

I stripped away the `<devices>` and some metadata. Most relevant ist the `<cpu>` tag, anyway. I kept the `<features>` but actually they are identical to those when I first generated the machine in virt-manager. You might also want to play with the CPU topology to map more or less Cores from the thost to your VM.

With the settings above I successfully ran Windows 11 with active WSL2, Docker Desktop and other container stuff. I totally mapped 8 virtual CPUs which "heat up" 8 cores (from 20) on my host.

Enjoy!
