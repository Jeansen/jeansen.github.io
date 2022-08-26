---
layout: post
title: Volume aliases with vagrant-libvirt
date: 2022-08-27T00:03:31+02:00
tags: [linux, vagrant, libvirt, vagrant-libvirt]
---

## Quick note to the world and myself

If you happen to detach an attach different disks when createing a Vagrant machine by means of the plugin `vagrant-libvir`, you might encounter an error regarding a fallback because `vagrant-libvirt` thinks the disk just attached was created in a very early version of `vagrant-libvirt`.

The solution to circumvent that warning and misguiding stack trace is to add `--alias ua-box-volume-0` when attaching a disk. If you have more disk, just increase the number.[^1]

In my case I have a trigger that now looks like the following:

    override.trigger.before :"VagrantPlugins::ProviderLibvirt::Action::StartDomain", type: :action do |t|
    t.info = "Setup pool for Test #{name}."

        t.run = {
          inline: "bash -c 'export LIBVIRT_DEFAULT_URI=#{LIBVIRT_DEFAULT_URI};
          virt-xml bcrm_test_#{name}-test --edit model=lsilogic --controller model=virtio-scsi;
          virsh detach-disk bcrm_test_#{name}-test sda --config;
          virsh attach-disk bcrm_test_#{name}-test #{Dir.pwd}/disks/libvirt/#{name}/#{cloned_disk}.qcow2 sda --targetbus scsi --driver qemu --subdriver qcow2 --type disk --config --alias ua-box-volume-0'"
        }
        t.exit_codes = [0, 1]
    end

[^1]: https://github.com/vagrant-libvirt/vagrant-libvirt/discussions/1552#discussioncomment-3455330

