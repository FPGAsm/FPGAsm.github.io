---
layout: post
title:  "Welcome to FPGAsm!"
date:   2021-07-24 10:35:51 -0700
categories: 
---

To my future self.  I pity the fool.

### Restarting the FPGAsm project

My brain is an imperfect device to begin with, and COVID-19 has not improved it, to say the least.  I'be been spending a lot of time spelunking my old drives, looking at projects and notes, often trying to figure out exactly what I was trying to accomplish.  FPGAsm is reasonably straightforward, but I will try to put down what I remember and what I discover as I go.  

### A Brief History

Around 2010 I picked up a pen to cross off a bucket list item labeled 'FPGAs'.  My learning curve was (and is) a long story for another time.  By 2012 I was sufficiently ired with the tools and vowed to toss them and start over.  Within months, a Smalltalk prototype of FPGAsm was functional, soon to be followed by a C-- version.  Based on its mission statement - allow rapid turnaround - FPGAsm was a success.  It is possible to build circuts with it - although it is not necessarily easy.

Before burning out and switching to other projects, I made a feeble attempt to create some infrastructure for Spartan-3 architecture, and the reference platform - `Digilent Spartan-3 Starter Board` with an XC3S200.  I was able to build led flashers, counters, ALUs and apparently came close to rigging a small CPU (a crossover with a several computer architecture projects).  

### Taking stock of FPGAsm

FPGAsm definitely works, and was built reasonably well.  A few days ago I cleaned it up a little, eliminating a few memory leaks and enlarging hardwired string buffers.  To my surprise, it loaded the .xdlrc of a large Artix-7 chip, orders of magnitude larger than anything I've tried before.  CL-FPGASM-DEVICE loaded the .xdlrc without any changes (thank god for Lisp!).  The turnaround with Artix is not as fast -- about 45 seconds for a trivial circuit.  XC3S200 is 4 seconds.  But it's still better than minutes with Vivado.

I am not too happy with FPGAsm.  It is a little too hard to use.  The syntax is slightly hoaky.  My pre-LISP past self struggled with the concept of a DSL.  He did OK, all things considered, but there are many things that can be a lot better.

C++ (or C-- in my able hands) is clearly the wrong tool for the job.  FPGAsm wrangles a lot of very complicated data, and memory manipulation is a big part of it.  Lisp is clearly the tool for that.  Speed has not been an issue (compilation is sub-millisecond currently), and using C++ is a combination my ignorance of Lisp at the time, combined with premature optimization.  Althogh to be fair, the Smalltalk version was sluggish...

Dealing with sub-grid circuits proved problematic.  When an entire slice, a unit on the X-Y grid, is used, things are fine.  Using a single LUT and leaving the second one presents an issue; in 7-series architecture with 8 LUTS it is a very serious issue.  FPGAsm's rigor in making sure you can't place anything in a grid position already in use prevents sub-cell (I think they call it fractional?) placement of different circuit parts.  I see that I made an attempt to solve it with `quarks`, but it was definitely an afterthought.

Parametrization is rudimentary.  FPGAsm deals with modules and wiring, but it lacks expressions as we know them in software languages.  There is simple text macro substitution, which is helpful, but no expressions, or variables.  X-Y grid handling is therefore rudimentary - only simple, literal numbers are allowed.  There should be another layer of abstraction for variables.  Passing everything as parameters keeps things simple, but there is a need for scoped environments.

As an exploration tool FPGAsm could be much better in a dynamic environment.  Exploring the FPGA grid interactively, and perhaps visually, would be a boon.  Xilinx FPGA editor is dead - it only works in Windows (The Linux version has been uncompilable for decades due to dependencies on OpenLook or something equally dead).  

### New Open-Source Toolchains

A lot (yet not nearly enough) has taken place in the past decade.  

Xilinx and other vendors are still (idiotically) not sharing information with the very people who could make their hardware truly amazing.  The current Vivado toolchain is slow as dogmeat and takes up 40 Gigabytes.  Waiting several minutes (perhaps hours with a very large design) is simply unacceptable.  I can't imagine what they are thinking.  At least you can still get ISE 14.7, and it supports Artix 100 (for which you can get a Digilent board for a couple of hundred bucks).  

On the positive side, there are several project that address FPGAs.  The bitstreams are somewhat documented, and bitstream generation, and routing tools are almost usable.  This means that FPGAsm may get a new, opensourced back-end.  I am hopeful.

### Plan of action

I think that moving to Common Lisp is inevitable.  Nothing else fits the bill.

I already have code to load an .xdlrc; this can be a foundation for the Lisp FPGAsm.  I hope I remember enough Lisp to move quickly.


