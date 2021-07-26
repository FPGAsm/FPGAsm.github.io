---
layout: post
title:  "FPGAsm Basic Concepts!"
date:   2021-07-25 15:00:00 -0700
categories: 
---
# FPGAam Basic Concepts

## FPGASM Modules

FPGASm is a modular hardware description language.  Modules are used to hierarchically describe a circuit.  A module must first be **defined**, providing an external interface -- module name, input pin names, output pin names, and parameter names.  The definition also describes the internals of the module.  Each module is a black box to the outside.  Internally it may contain instances of other modules and wiring between these modules and its own pins.  The contained instances are locally named.

A the lowest level of the hierarchy are **primitive-defs**, the FPGA's basic hardware circuits such as IOBs, LUTs and memories.  These are provided by the device .xdlrc file.  These are wrapped by higher-level modules, providing a more convenient abstraction.  

The IOB primitive is configured using complicated strings that refer to its internal elements (inverters, flip-flops, etc), and offers many standards of voltage and current.  It is much easier to create a simple module, say INSIMPLE which instantiates an IOB and sets it to LVTTL 3.3V, exposing a single wire called OUT.  My FPGA board has 8 toggle switches and it is very convenient to create a module SWITCHES containing 8 INSIMPLE instances, one for each input pin, with a an output bus OUT[8].  This module can in turn be instantiated in a higher-level module.

SWITCHES module is a singleton, and may be instantiated only once, since it is bound to specific IOBs in the FPGA.  Other primitive-defs such as LUTs and flip-flops are abundant, and modules built from those may be instantiated many times.  A REGISTER8 module, for instance, may contain 8 instances of a flip-flop, an may itself be instantiated hundreds or even thousands of times at different locations.

## FPGAsm is not a 'programming language'

Thinking of FPGAsm (and othre Hardware Description Languages) as programming languages is OK up to a point.  There are syntactic and structural similarities.  In software we define named, parametrized subroutines which internally manipulate their arguments and call other subroutines.  In FPGAsm we define named, parametrized **modules** which internally **instantiate** other modules, and provide wiring between them.  We compile software to create an executable binary.  We build FPGAsm code to create an .xdl description of the circuit. 

We execute code many times, sequentially.  With FPGAsm we transfer the resultant circuit description to an FPGA board, once.

More importantly, FPGAsm modules are really more like **macros** that are expanded.  A module is defined once and may be instantiated in many other modules; each instantiation is a different copy of the described circuit, placed in a different location on the FPGA.  Software subroutines are compiled to machine code actively called from many places when executed; an FPGAsm module is used as a **prototype** for creating instances of a circuit fragment during expansion.  A module definition itself is not a circuit - it is internal fiction, a reference prototype used only during expansion to create a circuit.

Unlike software, there is no penalty for creating very deep hierarchies of modules.  FPGAsm flattens them very fast - in milliseconds.  

## Definition and Instances

We draw a sharp distinction between a module definition, the single prototype description of a module, and the instances of such module contained in higher-level module definitions.

A definition is a conceptual entity.  It itself does not occupy space on a device - it is a prototype which may be used to construct a circuit.   High-level instances are also fictional - they are simply containers for lower level instances.  Only lowest-level instances, onces that contain primitive-defs, are actually used to construct the circuits.  Everything else is an organizational convenience -- their only purpose is to help your feeble brain (which is thoroughly incapable of keeping track of more than a handful of items).

The FPGAsm build process is largely a 'flattening' of the tree which starts at the top node.

## TOP node

The top node is a curiosity.  It exists only as a definition (it is automatically instantiated).  It must be called **top** and have no parameters.

It is the highest-level node, at the highest level of abstraction - it contains your entire project.  It is also responsible for all low-level minutia.  It dictates the placement of all sub-units on the FPGA grid.  It must specify all the IO, assigning pinning (unless it is hardwired elsewhere).

## Placement

FPGAsm code is responsible for the placement of circuits in specific locations on the hardware grid.

Since modules describe circuits in terms of containment, and may be instantiated multiple times, we do not specify absolute locations of instances within a module; instead, we must specify relative offsets within the module being defined.  An 8-bit ALU8 with a carry chain must be organized vertically .  If we build it with four 2-bit ALU2 slices occupying a slice, we will place these at (0,0), (0,1), (0,2) and (0,3) in ALU8's definition.  When ALU8 is instantiated at some position, these offsets will be added to instantiate each of the components.

IOBs are specific to FPGA pins, and their position must be specified by providing the cell name.  To reuse the above INSIMPLE example, we must parametrize the location, and instantiate each instance of INSIMPLE with a parameter specifying the proper location.  Our SWITCHES module with its 8 instances of INSIMPLE is the perfect place to specify that information.

## Scoping rules

**Module names** have a global scope.  

**Module parameter names** are local to the definition, but are visible in the instantiation's argument expression, for the purpose of parametrizing the instance

**Pin names** declared in a module definition are local to the definition, to be used for internal wiring.  When such module is instantiated, its pins are also accessible by name in the context of the instance.  For example, module FOO declares an OUT pin, and later we create an instances of FOO called BAR and BAS.  Both BAR and BAS have an accessible OUT pin.  We can wire something to BAR's OUT as well as BAS's OUT.


