---
layout: post
title: Running your own CA authority
date: 2020-02-22T19:33:49+01:00
tags: [Debian, Linux, security, nss]
---

At home, I have some services running that are using https and self-signed certificates. Normally this is not a problem. Whenever you open up a browser and it complains, you can simply add an exception. But with more services (and users), it can quickly become tiresome to always have to apply these exceptions. Especially during all sorts of tests where you reset your system or VM and simply want to try things out.

So, with the help of a very nice [tutorial (in German)](https://wiki.ubuntuusers.de/CA/#CAhinzufuegen) on ubuntuusers.de, I setup my own local demo CA and configured my servers accordingly.

Though I also copied the root certificate on all clients, not all applications will pick it up. Even when it is installed as a system-wide certificate. For instance Google Chrome uses a user-wide certificate database located at `~/.pki/nssdb` whereas Mozilla programs store this database in each profile directory, e.g. `~/.mozilla/firefox/XXXXXXX.default/`.

Fortunately, the way different applications implement their security with respect to SSL, S/MIME and other Internet security standards has been standardized with [Network Security Services (NSS)](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/NSS/Overview). 

With little effort I hacked together a [little script](https://gist.github.com/Jeansen/c6a72cd39d43e5208763d7d5271105ea) that you can use to add a certificate to any NSS certificate database (version 8 or 9).



