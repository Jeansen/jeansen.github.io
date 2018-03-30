---
layout: post
title: My VIM is slow - how to boost it up
tags: [vim, linux]
date: 2018-03-11T19:11:38+01:00
---

For some time now I ran into the problem that editing larger files in VIM became quite slow when moving the cursor. 
Holding 'j' was very sluggish. I first suspected a plugin to be the culprit but my examination was to no avail.

So, I searched the Internet and found all sorts of tips that might help. Most of them suggested turning of the cursor line,
setting 'ttyfast' or experiment with a buch of other settings. But none helped except turning off syntax
highlight completely (for the current file type).

Finally, I found [this post on stackoverflow.com](https://stackoverflow.com/questions/16902317/vim-slow-with-ruby-syntax-highlighting).

There, the author of the accepted answer states that any version up to Vim 7.3.969 uses an older regex engine than 
e.g. VIM 7.4 or 8.0. 

So, I followed the authors answer and added `set re=1` to my .vimrc file. This will set the regexp engine to the old
engine. (see `:help regexpengine`). And that did the trick. Finally!

Here is what `syntime` gives me before and after setting the regexp engine to use the old engine.

    TOTAL      COUNT  MATCH   SLOWEST     AVERAGE   NAME               PATTERN

    1.439144   8286   0       0.002236    0.000174  perlMatch          \%([$@%&*]\@<!\%(\<split\|\<while\|\<if\|\<unless\|\.\.\|[-+*!~(\[{=]\)\s*\)\@<=/\%(/=\)\@!
    0.712566   10847  2561    0.000394    0.000066  perlStatementProc  \<\%(after\|any\|before\|before_template\|cookies\|cookie\|config\|content_type\|dance\|dancer_version\|debug\|dirname\|engine\|error\|false\|forward\|from_dumper\|from_json\|from_yaml\
    0.507931   8821   535     0.000332    0.000058  perlString         \I\@<!-\?\I\i*\%(\s*=>\)\@=


    TOTAL      COUNT  MATCH   SLOWEST     AVERAGE   NAME               PATTERN

    0.244700   8330   320     0.000189    0.000029  perlString         \I\@<!-\?\I\i*\%(\s*=>\)\@=
    0.155624   8010   0       0.000477    0.000019  perlMatch          \%([$@%&*]\@<!\%(\<split\|\<while\|\<if\|\<unless\|\.\.\|[-+*!~(\[{=]\)\s*\)\@<=/\%(/=\)\@!
    0.122767   10762  2752    0.000110    0.000011  perlStatementProc  \<\%(after\|any\|before\|before_template\|cookies\|cookie\|config\|content_type\|dance\|dancer_version\|debug\|dirname\|engine\|error\|false\|forward\|from_dumper\|from_json\|from_yaml\
    
As you can see, forcing the old regexp engine results in a about three to nine times faster parsing! In addition the initial
loading of a file is almost instant. Pay attention to the call of `perlMatch`. If you compare the count and total columns,
you will see the impact.

Surprisingly this problem not only seems to exist in VIM 7.4, but even 8.0.