---
layout: article
title: A Truffle/Graal High Performance Backend for JRuby
author: Chris Seaton
date: 6 January 2014
copyright: Copyright © 2014 Chris Seaton.
redirect_from: "/rubytruffle/announcement/"
---

<p style="text-align: center">
<img alt="Ruby Logo (Copyright (c) 2006, Yukihiro Matsumoto. Licensed
under the terms of Creative Commons Attribution-ShareAlike 2.5.)" src="../ruby.png" width="96" height="96">
<img alt="JRuby Logo (Copyright (c) 2011, Tony Price. Licensed
under the terms of Creative Commons Attribution-NoDerivs 3.0 Unported (CC BY-ND 3.0))." src="../jruby.png" width="96" height="96">
</p>

For the past year [Oracle Labs](https://labs.oracle.com/) have been working on an implementation of Ruby built upon two new JVM technologies - [the Truffle AST interpreter framework and the Graal JVM compiler](https://openjdk.java.net/projects/graal/). We believe that the new approach will lead to a faster and simpler Ruby implementation.

We’ve been talking to Charles Nutter and Thomas Enebo for several months and have given them early access to our code. We’ve also been talking to other Ruby implementors such as Alex Gaynor of Topaz, and presenting our results to the JVM language community at the [JVM language summit](https://www.youtube.com/watch?v=hDfYRjNynmQ). Today we are open sourcing our implementation and, with the consent of Charles and Thomas, pushing a patch that begins the integration as an optional backend in JRuby.

This blog post describes some of the background to this project. The [original code](https://hg.openjdk.java.net/graal/graal) is available in the Graal Mercurial repository and the patch to JRuby is available in a new `truffle` branch in the JRuby Git repository. There is documentation of how to use the code and an FAQ in the JRuby wiki, as well as pointers to more in-depth technical information such as peer-reviewed research publications.

## What does this new implementation do differently?

The new backend is an AST interpreter. We call it the Truffle backend because it’s written using the [Truffle framework for writing AST interpreters](https://openjdk.java.net/projects/graal/) from Oracle Labs.

Truffle is different to other AST interpreters in that it creates ASTs that specialize as they execute. The AST interpretation methods that are currently in JRuby are megamorphic, which means that they must handle all possible types and other dynamic conditions such as branches taken. In Truffle, AST nodes ideally handle a minimal set of conditions, allowing them to be simpler and more statically typed, which can lead to more efficient code. In the less common case that the node's conditions aren’t met the node is replaced with another node that can handle them.

AST interpreters are generally thought of as being slow. This is because every operation becomes something such as a virtual method call. MRI 1.8 used a simple AST interpreter, and JRuby still uses an AST interpreter by default for the first run of methods. To improve performance many language implementations convert the AST to bytecode. Python, Ruby 1.9 and above (via YARV) and PHP all do this. Normally the bytecode is still interpreted, but it’s often faster as the data structure is more compactly represented in memory. In the case of many JVM languages like JRuby and other Ruby implementations such as Topaz and Rubinius, this bytecode is eventually compiled into machine code by the JIT compiler.

Again, Truffle takes a different approach here. When running on JVM with the Graal JIT compiler, Truffle will take all of the methods involved in interpreting your AST and will combine them into a single method. The powerful optimisations that the JVM usually applies to single methods are applied across the combined AST methods and a single machine code function per Ruby method is emitted by Graal.

For more information about what Truffle and Graal do see the [JRuby wiki page](https://github.com/jruby/jruby/wiki), the a recent [project summary slide deck](https://www.slideshare.net/ThomasWuerthinger/graal-truffle-ethdec2013).

## Is this going to change with how you use JRuby today?

The Truffle backend, Truffle itself, and Graal are research projects and are certainly not ready for general use today. It is very unlikely that your application or gem will run right now, but if you are interested in the JRuby internals, or the JVM or compiler technology in general we’d encourage you to take a look at what we’re doing. The JRuby wiki page will give you a starting point.

For the foreseeable future this work is going to live on a separate `truffle` branch in the main JRuby Git repository. It is possible that this branch will be considered for merging into the master branch before the next major release, JRuby 9000, but this a decision for the wider JRuby community and its leadership.

We believe that in the further future the Truffle backend could be good enough to become the default backend for JRuby, but again this is a decision for the JRuby community.

## What are we going to do next?

We are going to continue to integrate Truffle into JRuby at the same time as continuing to implement more of the Ruby language. We already have very encouraging results with our initial implementation and with the excellent work already done by the JRuby community we think we can fill in the gaps to be a complete implementation.

{% include trufflerubylinks.html %}
