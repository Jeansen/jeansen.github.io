---
layout: post
title: Gnomde GDM Authentication With U2F
date: 2017-11-11T19:09:33+01:00
tags: [u2f, Debian, Linux]
---

I have always wanted to add physical authentication to my desktop authentication. For some time now I have been using a 
U2F USB dongle from Yubico. So, why not use it for two-factor authentification with GDM?

And it turns out, it is quite simple! Well, at least on the current release of Debian Stretch.

Add the necessary packages: 

    pamu2fcfg libpam-u2f
    
Make sure your FIDO U2F device is plugged in and run:
    
    sudo pamu2fcfg -u ${USERNAME} >> /etc/security/u2f_keys
 
Finally, prepend the following line to `/etc/pam.d/gdm-password` to setup PAM for GDM:

    auth required pam_u2f.so cue authfile=/etc/security/u2f_keys
    
Now, logout. If everything went OK, you will now first have to touch your U2F device and only then you will be able to type
in the password for your user.