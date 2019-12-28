---
layout: post
title: Gnomde GDM Authentication With U2F
date: 2017-11-11T19:09:33+01:00
tags: [u2f, Debian, Linux]
---

I have always wanted to add physical authentication to my desktop authentication. For some time now I have been using a 
U2F USB dongle from Yubico. So, why not use it for two-factor authentication with GDM?

And it turns out, it is quite simple!

Add the necessary packages: 

    pamu2fcfg libpam-u2f
    
Make sure your FIDO U2F device is plugged in and run:
    
    sudo sh -c "echo $(pamu2fcfg -u ${USERNAME}) >> /etc/security/u2f_keys"

Then touch your Yubikey and check that you have a new line added to `/etc/security/u2f_keys`.
 
Finally, prepend the following line to `/etc/pam.d/gdm-password` to setup PAM for GDM:

    auth required pam_u2f.so cue authfile=/etc/security/u2f_keys
    
If you also want to secure the virtual terminals, you should add the same line above to `/etc/pam.d/login`. Also check the other files available in `/etc/pam.d`. Maybe there are other places you would like to add two-factor authentication to.
    
Now, logout! If everything went OK, you will now first have to touch your U2F device and only then you will be able to type in the password for your user.
