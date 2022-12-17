---
layout: post
title: Mapping Copy/Paste in Xterm and RXVT Unicode
date: 2022-12-13T01:02:57+01:00
tags: [xterm, xrvt, urxvt, X11, xresources, linux]
---

Bought myself a new laptop (TUXEDO InfinityBook Pro Gen7). Unfortunately, the `INSERT` key does not exist. Well, it does, but it's not a stand-alone key. Yes... it's complicated. But well, where would all the fun be if that would not be a challenge?! ;-)

After all, I always wanted to switch some key mappings, anyway. At work, I have to use a MacBook. There, I already mapped `Shift-Backspace` for pasting clipboard contents.

This mapping is, however, not globally controlled in X11. At least, I could not find any setting, though in any X11 application one can use `Shift-Insert`. What I had to do was to review my `~/.Xresources` and do some tinkering there. I haven't touched that file in years. And I recall, it took me years to have it twisted to my likings.

Yes. Back to school! Here is, what you need:

### Xterm
To map the clipboard to `Shift-Backspace`, in Xterm put the following in your `.Xresources`.

    *VT100.translations: #override \n\
    Shift <Key>BackSpace: insert-selection(RIMARY, CLIPBOARD, CUT_BUFFER0)\n\
    /*Ctrl Shift <Key>c: copy-selection(CLIPBOARD)\n\*/

The `#override` part is crucial. It makes sure that already defined mappings will be overwritten!

### RXVT-Unicode
To map the clipboard to `Shift-Backspace`, in RXVT Unicode put the following in your `.Xresources`.

    URxvt.keysym.Shift-BackSpace:eval:paste_clipboard
    /* URxvt.keysym.Shift-Control-C: eval:selection_to_clipboard */

### The problem is choice
Because I always run in Tmux, I do not care for the copy part. Everything I copy from other X11 applications, VIM or even within Tmux is available in the clipboard. I just need to be able to paste it either in any other X11 application, e.g. GEdit (using Ctrl-V) or VIM (using 'p'). Yes, I could also paste via Tmux, but that would need more strokes.

With the mappings above, I have the choice. `Shift-Backspace`, `Ctrl-\ p` (Tmux) or simply `p` in VIM. Of course, when using VIM, I could also hit `i` and while in insert mode use `Shift-Backspace` or `Ctrl-\ p` (for Tmux). If not using VIM, I could stick to Tmux all the time. Then and when it might happen I do not even have a Tmux session at hand and am too lazy to create one. But what I always have is either Xterm or XRVT-Unicode terminals.

### Coming next ...
Nex post will be about my transition from an older Dell Latitude to a modern Tuxedo InfinityBook Pro. I bet, I'll have to fix some bugs in [cdmn](https://github.com/Jeansen/cdmn). After all, my CPU count went from 4 to 16! The project could get some more attention, after all. But for the moment, I've been too busy with [bcrm](https://github.com/Jeansen/bcrm). As it turns out, if you don't test it, it WILL break. That's what I had to deal with lately. Now, I am ready to clone over my encrypted system to a new encrypted clone. And when I am fine with my new setup, a lot more bugs await fixing and new features to be added.

Ah well, I'll hit the hay. I don't complain. Challenge accepted! ;-)

