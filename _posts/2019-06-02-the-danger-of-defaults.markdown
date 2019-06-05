---
layout: post
title: The danger of defaults
date: 2019-06-02T23:20:47+02:00
tags: [linux]
---

I am a developer by profession. And like most developers, I am lazy. So yes, I get paid for doing less! More precisely it is not about doing more or less but about keeping things simple. And that's actually harder than one can imagine.

A very simple example that already splits the community is about declaring or defining variables. When you declare a variable you give a name and a type to a variable, e.g. in Java: `int myVar;`. When you define a variable, you assign it a value in addition: `int myVar = 0;`. Usually you go with the first one because declaring a variable of a simple type automatically initializes it with a default value, which is 0 in this example.

But some people argue one should always define a variable to make sure - in case defaults change - everything works as expected. Well, I honestly do not believe that the makers of a programming language will change defaults for simple types. But, for complex types, it might happen.

While this example seems fairly easy and obvious, there might be situations where you think the default is fine but the truth is the opposite.

Here is my example that made me write these lines and taught me a lesson (again!). If you use Linux, then you might be familiar with the Logical Volume Manager (LVM). If you do not know LVM, don't worry. Anyway, there are several commands to check for different parts related to LVM. These tools show - among a lot of other things - the size of used and available disk space. You can even define what columns you want to see and in what unit (with or without suffix) they should be printed.

In my concrete example I had the following command:

    lvs --noheadings --units m --nosuffix -o lv_name,vg_name,lv_size,vg_size,vg_free,lv_role,lv_dm_path
    
Which printed something like this:
  
      root   debian-vg 409600.00 487628.00 10484.00 public,origin,thickorigin     /dev/mapper/debian--vg-root  
      snap1  debian-vg  51200.00 487628.00 10484.00 public,snapshot,thicksnapshot /dev/mapper/debian--vg-snap1 
      swap_1 debian-vg  16344.00 487628.00 10484.00 public                        /dev/mapper/debian--vg-swap_1

Perfect. I could use this output, pipe it to `awk`, extract say column 3 (lv_size) and add them together. Something similar worked perfectly for a long time until it suddenly failed on one system. Now the search began and I found out that the system in question used different locale setting than expected. In this case:

        LC_NUMERIC=de_DE.UTF-8
        LANG=de_DE.UTF-8

And with this setting every decimal point in the above example was replaced by a comma. In general this should not be harmful if every operation adhered to the system locales. Only if you cross system boundaries the result could be hard to track down.

But even in the case where you stay within system boundaries some tools might fail. In my case it was `bc` that failed because it expects a dot as a decimal separator.

And there you have it. Some tools honor the system defaults, others do not.

So, to make everything work - even across system boards - I added the following line in my script:

    LC_ALL=en_US.UTF-8
    
This way it does not matter where the script runs. Every execution will be evaluated in a temporary locale of `en_US.UTF-8`. Of course the input from other script, like a file from another batch script, has to uses the same setting!

Therefore, whenever you find yourself in a heterogenous system environment it is obviously better to go by the rule:

    Expect nothing, assume everything!
    
So long, and thanks for all the fish :-)

