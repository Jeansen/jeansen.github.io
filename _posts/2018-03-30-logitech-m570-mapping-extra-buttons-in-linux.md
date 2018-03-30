---
layout: post
title: Logitech M570 - mapping extra buttons in Linux
tags: [linux]
date: 2018-03-30T13:21:32+02:00
---

I have always wanted to use the extra buttons on my Logitech M570 trackball with Linux. On Windows one can use a nice
GUI tool provided by Logitech that allows you to quickly change these settings. For instance I like to map the forward
key to 'Page_Up' and the backward key to 'Page_Down'.

On Linux there is no such tool. That is, there a other system tools that you can utilize, but nothing provided by
Logitech.

First, make sure you have the right packages installed:

    sudo apt-get install xbindkeys
    sudo apt-get install xautomation

You will not need the package 'xbindkeys-config'. Though it is a GUI for 'xbindkeys', I could not get it to identify my
trackball buttons. So, I did it by hand.

Next, you will need to find the correct button numbers. The following command should make this easy:

    xev | grep -i button

Her is an example of what the output should look like after you press the forward and backward buttons.

    ButtonPress event, serial 35, synthetic NO, window 0x1800001,
        state 0x10, button 9, same_screen YES
    ButtonRelease event, serial 35, synthetic NO, window 0x1800001,
        state 0x10, button 9, same_screen YES

    ButtonPress event, serial 35, synthetic NO, window 0x1800001,
        state 0x10, button 8, same_screen YES
    ButtonRelease event, serial 35, synthetic NO, window 0x1800001,
        state 0x10, button 8, same_screen YES

As you can see, the forward button has number 9 and the backward button number 8.

Finally you can use this information to configure 'xbindkeys'. Put (or append) the following in the file '.xbindkeysrc'
in your home directory (~/.xbindkeysrc) and adapt the numbers if need be.

    # Key bindings for Logitech M570 trackball
    "xte 'key Page_Up'"
    b:9
    "xte 'key Page_Down'"
    b:8

If you do not want to map to 'Page_Up' and 'Page_Down' you should check the man page for 'xte'. I should show you a list
similar to the following one with some common key names you can use:

    Home
    Left
    Up
    Right
    Down
    Page_Up
    Page_Down
    End
    Return
    BackSpace
    Tab
    Escape
    Delete
    Shift_L
    Shift_R
    Control_L
    Control_R
    Meta_L
    Meta_R
    Alt_L
    Alt_R
    Multi_key
    Super_L
    Super_R

To make thing work, you will have to restart X. Simply log out and back in to your desktop environment and see if things
work as expected.