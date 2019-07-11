---
layout: post
title: Using apt-cacher-ng with https
date: 2019-07-11T18:22:54+01:00
tags: [linux, debian]
---

For some while now I had the caching capabilities with apt-cacher-ng disabled because of problems with HTTPS. Finally I found some time to get it working.

## Installation
I already had apt-cacher-ng installed. If you haven't, check out the [Debian wiki](https://wiki.debian.org/AptCacherNg). To summarize an get you started, here is what you need to install on the server:

    sudo apt-get install apt-cacher-ng
    sudo apt-get install avahi-daemon
    
And on each client you would install:

    sudo apt-get install squid-deb-proxy-client
    
## Configuring AVAHI

On my client avahi-daemon is a dependency of GNOME. But avahi-daemon conflicted with my local name resolution. I could still use `nslookup` but a simple `ping <some-local-hostname>` would fail.

The fix to my DNS resolving issues was quite easy. Instead of fiddling around with `/etc/nsswitch.conf` I was better off by simply uncommenting `#domain-name=local` in `/etc/avahi/avahi-daemon.conf`

## Configuring apt-cacher-ng for HTTPS

Without any configuration apt-cacher-ng will fail with packages that use HTTPS. So, any `apt update` will complain.

To have apt-cacher-ng work with HTTPS I rewrote all entries in `/etc/apt/sources.list` and `/etc/apt/sources.list.d/*` from e.g. `deb http://dl.bintray.com/sbt/debian /` to `deb http://HTTPS///dl.bintray.com/sbt/debian /` [as suggested by the author of apt-cacher-ng.](https://www.unix-ag.uni-kl.de/~bloch/acng/html/howtos.html#ssluse).

## Set the UserAgent

I've experienced that setting the `UserAgent` explicitly can avoid a lot of strange effects, e.g. 403 access denied codes. Simply add the following to your `/etc/apt-cacher-ng/acng.conf` file:
  
       Debian APT-HTTP/1.3 (1.6.10)

This will overwrite the default setting, which is `Yet Another HTTP Client/1.2.3p4`

## Configuring APT

And just to be sure there should be a file /etc/apt/apt.conf.d/30autoproxy containing the following sting:

    Acquire::http::ProxyAutoDetect "/usr/share/squid-deb-proxy-client/apt-avahi-discover";
    
A manual entry without auto detection would be:

    Acquire::http { Proxy "http://<server-ip-or-name>:3142"; }

## Conclusion

Caching packages with apt-cacher-ng is almost as easy as just installing and get going. You only have to pay attention that AVAHI does not interfere with your normal DNS resolution.
    

