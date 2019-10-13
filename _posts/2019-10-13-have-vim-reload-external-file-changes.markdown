---
layout: post
title: Have vim reload external file changes
date: 2019-10-13T01:34:19+02:00
tags: [vim, linux]
---

Usually, when doing some shell scripting (Python, Perl, Bash) I use vim. I would even use it for other languages, but when developing in languages like Kotlin or Java there is simply no way around a full-blown IDE. Most IDEs also have a lot of other nice features that can prove very useful. For instance IntelliJ IDEA is absolutely priceless for diffs in Git. So, whenever I can, I combine the power of multiple tools.

After having fixed a bug in a shell script I did the review in IDEA. During the review I made some minor changes. To verify the changes I ran the script again and found yet another issue. So I changed back to my vim session and made the adjustments needed there. But when I wanted to save the changes, I got a warning the file had changed on disk. Reloading would undo my latest changes done in vim whereas saving would undo my changes done in IDEA. Either way, I was confronted with the loss of some work.

Now, here comes the important part. I knew, I had already set `autoread` which should actually avoid such a situation. But it turned out, that wasn't enough. I also had to add `au CursorHold * checktime`.

So, to have vim automatically reload your files on external changes, put the following two lines in your `.vimrc`:
    
    set autoread                              
    au CursorHold * checktime
    
The only conflict situation that still can occur is when you start editing in vim (without saving) and continue in another program. Then vim will ask you again what to do: load the file from disk or keep the current buffer.
