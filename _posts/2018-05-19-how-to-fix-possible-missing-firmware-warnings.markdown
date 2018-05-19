---
layout: post
title: How to fix possible missing firmware warnings
date: 2018-05-19T23:23:42+02:00
tags: [Debian, Linux]
---

While running 'update-initramfs -u' after [adding a module to initramfs](2018-05-19-how-to-add-kernel-modules)
I got the following warnings:

    W: Possible missing firmware /lib/firmware/i915/skl_dmc_ver1_27.bin for module i915
    W: Possible missing firmware /lib/firmware/i915/kbl_dmc_ver1_04.bin for module i915
    W: Possible missing firmware /lib/firmware/i915/kbl_guc_ver9_39.bin for module i915
    W: Possible missing firmware /lib/firmware/i915/bxt_guc_ver9_29.bin for module i915
    W: Possible missing firmware /lib/firmware/i915/skl_guc_ver9_33.bin for module i915

I wondered how to get rid of these and finally found a solution.

First, I cloned the linux firmware repository from kerrnel.org:

    git clone https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/

Then I copied over all the missing binaries from the i915 folder to '/lib/firmware/i915'. So, in my case:

    sudo cp skl_dmc_ver1_27.bin kbl_dmc_ver1_04.bin kbl_guc_ver9_39.bin bxt_guc_ver9_29.bin skl_guc_ver9_33.bin /lib/firmware/i915

Finally, I had to run 'update-initramfs -u' again. Et Volià. All warnings gone!




