---
layout: post
title:  "LuaTruffle"
date:   2015-01-22 11:00:00
---

Since September of 2014, I have been working in a personal open source project called [LuaTruffle][luatruffle].
This project is an implementation of the [Lua language][lua] built on top of JVM.
It uses two new JVM technologies: [Truffle AST and GraalVM][graalvm].

The language is not completely implemented yet, however it already shows good results:

<img src='/images/mandelbrot-luatruffle.png'/>

We're around 13x faster than [Lua][lua] and about as fast as [Luajit][luajit].

[luatruffle]: http://en.wikipedia.org/wiki/Pcap
[lua]: http://www.lua.org/
[graalvm]: http://openjdk.java.net/projects/graal/
[luajit]: http://luajit.org/
