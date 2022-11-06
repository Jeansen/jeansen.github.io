---
layout: post
title: Setup athens proxy for GO
date: 2022-11-05T20:57:28+01:00
tags: [go, raspberry pi, linux]
---

During the automated setup of my local Kubernetes cluster, I decided to cache most of my requests to APT. This works in a similar fashion e.g. Artifactory, which I use to proxy Java/Kotlin dependencies. I soon faced the need to do the same for GO.

The easiest way would probably have been to continue with Artifactory and use it for APT, GO, NPM and so on. Unfortunately, only the Maven part is free for individual applications.

An alternative would have been Nexus. The open-source version offers all the repository types needed, e.g. APT or GO (and more). But there are some downsides. For example, there is no .deb package available. Installations must be done by hand. Configuration is another point of critic. Compared with Artifactory, it feels cumbersome to me. And finally, it still needs Java 8! The folks behind Nexus haven't been able to port it to Java 9 and later, yet.

Either way, Artifactory and Nexus are quite "heavy". Not something I would run on a Raspberry Pi 3. So, I found Athens. It's still in development, but exactly what I needed: Lightweight, simple setup. Only drawback here: No .deb file, either. But they provide an install script that builds Athens and installs a SystemD service, fully automated. There are some things to consider, though. What follows is a little step-by-step instruction of my journey installing Athens on a Raspberry Pi 3.

Note: In the following sections I will use  `host` and `proxy` to denote my laptop and my Raspberry Pi (which I baptized 'proxy'). In former times, `pi` was the default user on any Raspberry Pi. Nowadays when you install Raspberry Pi OS, you will be prompted to name a first user. To make things simple, I'll stick wih the user `pi`.

## Pre-Flight

Before I could install Athens by means of compiling it, I heeded a GO versions >= 1.17. Raspberry Pi OS is based on Debian Bullseye (stable), which currently only ships with GO 1.15. But one can add the Debian backports repository to have newer versions of GO available. 

First, I added the backports repo on the Raspi:

    echo "deb http://deb.debian.org/debian bullseye-backports main" | sudo tee -a /etc/apt/sources.list

Then I needed to get the repository keys. For that, I spun up a Debian container on my local machine with Docker and copied over the public keys:

    mkdir -p /tmp/transfer
    docker run -it --rm debian:bullseye -v /tmp/transfer:/transfer

    #Within the container
    cp -r  /etc/apt/trusted.gpg.d/ /transfer

Next, I copied the keys over to my Raspi:

    scp -r /tmp/transfer/trusted.gpg.d/ pi@proxy:

After I logged in on the Raspi, I finally copied the keys so APT would find them:

    sudo cp ~/trusted.gpg.d/*  /etc/apt/trusted.gpg.d/ 

Now, I was able to install GO 1.19 from the backports repository:

    sudo apt update && sudo apt install golang -t bullseye-backports

## Athens installation

Installing Athens was quite simple. First, I had to get the code:

    git clone https://github.com/gomods/athens.git
    
Then, I had to copy over the template config and adjust is to my liking. So far, I only made sure that instead of storing everything in memory, Athens would store all cached data on disk:

    cd athens
    #systemd.sh expect a file config.toml at the root of the repository folder
    cp config.dev.toml config.toml

In `config.toml` I changed `StorageType = "memory"` to `StorageType = "disk"` and in the '[Storage]' tree I set the path to where I wanted to have all files saved by replacing `RootPath = "/path/on/disk"` with `RootPath = "/var/cache/athens"`.

I also needed to create that 'RootPath' (with the right permissions). When invoked by SystemD, Athens will be executed with the user `www-data`. So, I had to take that into account:

    sudo mkdir -p /var/cache/athens
    sudo chown -R www-data:www-data /var/cache/athens

In `scripts/systemd.sh` I had to comment out the following four lines in `doInstallConfig()`:

    ATHENS_DISK_STORAGE_ROOT=/var/run/athens
    sudo mkdir -p $ATHENS_DISK_STORAGE_ROOT
    sudo chown www-data $ATHENS_DISK_STORAGE_ROOT
    sudo chgrp www-data $ATHENS_DISK_STORAGE_ROOT

Otherwise, my customized 'RootPath' would have been overwritten.

Now, with the right GO version installed, a customized config file and a little fix applied, I was fully prepared to install Athens. So, from the root of the repository folder I only had to execute:

    scripts/systemd.sh install

That execution compiled Athens and installed a SystemD service.

## Test

Back on my host machine, I was then able to install for instance yq (which is not yet in the Debian repositories):

    export GOPROXY="http://proxy:3000
    go install github.com/mikefarah/yq/v4@latest
    
After that I had a look on the Raspi. There should now be some data in `/var/cache/athens`, and it was. So, from now on, Athens will intercept each call, download any available updates or directly serve any cached version, in case there are no updates available upstream.

## Conclusion

Athens is simple and easy to setup. The only thing that cost me some time was the hard-coded `ATHENS_DISK_STORAGE_ROOT` variable. Unfortunately this installation procedure is not promoted by the project. Neither on the official homepage nor on the README. You can find it in [DEVELOPMENT.md](https://github.com/gomods/athens/blob/main/DEVELOPMENT.md) file. Athens offers an abundance of configuration settings. You should definitely check out the full `config.dev.toml` file. Hint: If you execute `scripts/systemd.sh` without any parameters, it will tell you what it can do!
