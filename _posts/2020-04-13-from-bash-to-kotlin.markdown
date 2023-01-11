---
layout: post
title: From Bash to Kotlin
date: 2020-04-13T13:28:46+02:00
tags: [linux, kotlin, bash]
---

I've already written multiple posts during my development of bcrm. I learned a lot new Linux skills. And of course I also improved my shell scripting skills in Bash. But, as much fun as it has been, the script is now at a development stage where I have to think about another technology.

Shell scripts are cool. But, they are not well suited for big programs. Debugging can quickly become almost impossible. I utilized clean code as much as possible. And so far, the script works fine. I even created a test pipeline so I could better cope with all the possible execution environments and combinations.

But having a script with almost 3 thousand lines of code now scratches the limits of a scripting language like Bash.

So, I started thinking of how to best convert it to another language. I know some python, but with its dynamic typing it fell off the list. I'd face the same issues I would with Bash. The holds true for Perl 5. Version 6 I did not check. It's simply too special, in my opinion.

So, I thought about JavaScript with NodeJS. That seemed to be a good candidate for a command-line tool. But when I checked for a simple arguments parser, I got a bit shocked that on the one hand NodeJS does not offer a core implementation and available npm packets always "suck in" a dozen dependencies (transitive dependencies unregarded). So, I put this thought aside quickly, too.

Go is another candidate. I kept it on the list. It has all I need (including comand-line parsing) in its core. 

Nex was Java. Of course, that is a big one and that was exactly why I did not like it. The idea of having to have a JVM for a simple command-line tool is something I do not feel comfortable with.

So, I thought about Kotlin. Again, it is a JVM language, but there is also Kotlin/Native which allows one to use Kotlin and create a compiled executable. That one was very appealing because I know Kotlin already quite well and have some ongoing development projects where I use it. One language that fits it all. So, I kept it on the list.

Ruby was the last candidate. I became a bit familiar with it during my work with Vagrant. It's a lean language, but performance is terrible. It seems to be fine for prototyping and has some elegance to it. But from what I saw (including documentation), it simply did not feel right.

# Connclusion

Ruby, Python, Perl and Bash, they all are fine for scripting and did not feel well suited for a bigger, growing code base. NodeJS is interesting, but I do not like all those npm dependencies. And the npm package repository hasn't received my full trust, either. In addition, the JavaScript community seems to not care about an abundance of dependencies very much.

So, I had to make a choice between Go and Kotlin. Both I know, though the former I haven't used that much for some time and the latter I use more or less frequently (apart from Java). The idea of using one language for the JVM and an compiled executable had a very nice touch to it.


