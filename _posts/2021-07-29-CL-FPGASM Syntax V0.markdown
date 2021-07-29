---
layout: post
title:  "CL-FPGASM Syntax v.02"
date:   2021-07-28 12:37:00 -0700
categories: cl-fpgasm
---
This is a preliminary specification for CL-FPGASM module and instantiation syntax.

### Module Syntax
```
(MODULE <mod-name> <mod-lambda-list> <pins> <body> )
```

| mod-name         | name of the module |
| mod-lambda-list  | parameters for instantiation |
| pins             | pin-list for wiring |
| body             | module definition body |

**name** is the symbolic name of the module.  Upon successful definition, the symbol-value will contain a MODULE structure with the details of this module, and symbol-function will contain an instantiation macro, which, when expanded inside another module, will create an instance of this module.

The **mods-lambda-list** is a Lambda-List containing the names of parameters to be used for the instantiation of this module.  FPGAsm supports required, optional and keyword parameters (for now, keywords must be used as argument names).  Optional and keyword parameters with default values are supported.  &allow-other-keys feature is supported.

The **pins** parameter is a list of zero or more pin names.  By default the pins are inputs; this may be changed by using `&in` and `&out` keywords.  All names following such a keyword are affected by it.  Any number of `&in` and `&out` groupings may be used.*

Each pin may be a scalar pin, represented by a single symbol, or a pin-bus of several pins sharing the same name.  In order to specify a pin-bus, a list of two elements provides a name and a width of the bus.  Internally, a scalar pin is treated as a pin-bus with the width of 1.
Note that it is possible to pluck individual wires out of a pin-bus, or specify a partial bus to match the width and sequence of another bus.  It is not possible to 'construct' a bus from disparate pins - such cases requre individual wiring.

The **body** is a list of (almost) arbitrary Lisp expressions evaluated at the time the module is defined.  The body **may** contain instantiation expressions, making the module a container of module instances.  If the module declares pins, the body **must** contain one or more wire expressions, providing internal wiring from each the module's in-pins and to each of the module's out-pins.

The body may contain strings, which will be treated as comments.

Modules **should** be defined in the top context, that is, at the top level and in a null lexical environment, unless you really know what you are doing.

Upon module definition, CL-FPGASM:
* Creates a MODULE structure of same name, with module information and the entire source SEXP expression for later editing;
* Creates an environment containing a dynamic variable `*THIS-MODULE*` with the partially-populated MODULE structure;
* Evaluates the body of the module
* Generates a macro of same name name and and module's lambda-list, to be used for instantiation of module's instances inside other modules.

During the evaluation of the body, instancing expressions add to `*THIS-MODULE*`'s instance array and wiring instructions expand its wiring data.  

### Instantiation syntax
```
(<mod-name> <inst-name> <args>)
```

| mod-name | name of the module being instantiated |
| inst-name | name of this instance |
| args | instantiation arguments

Instantiations occur during the macroexpansion of the containing module definition, and result in an INST structures being created and added to the module's instance array.  Each previously-defined module provides an instantiation macro of the same name, to be used exclusively inside later module definitions.  The macro has the same lambda list as the one provided to the module.  

The arguments **MUST** match the mod-lambda-list, using normal Lisp rules.  The arguments are evaluated and stored in the INST structure along with parameter names in a key-value database.  The INST structure keeps a reference the the module representing this instance.

### Wiring syntax

```
(WIRE <source-pinholder> <source-pin-id> [/to/and] <sink-pinholder> <sink-pin-id> ...)
```  

| source-pinholder | name of the object containing the source pin or pin-bus |
| source-pin-id | pin identifier of the source pin or pin-bus |
| to and | optional syntax sugar for readability |
| sink-pinholder | name of object containing the sink pin or pin-bus |
| sink-pin-id | pin identifier of the sink pin or pin-bus |

The body of the module definition with pins must contain a complete wiring network in which every mod-pin is terminated as a source or a sink, using WIRE instructions.  Each pin must have a single wire attached to it.

Each wire has a single **source** and one or more **sink**.  Each sink endpoint of a WIRE instruction must be the same width as the source endpoint, be it a single wire or a pinbus.

Each sink or source endpoint is represented by two sequential items: the pinholder and the pin-id within it.

#### PIN-HOLDER

The pin-holder may be any of the instances of this modules, identified by the instance-name.  The most-recently-defined instance may also be referred to by pronouns `HIS`, `HER`, `ITS` or `THEIR`.  Each instance has a set of pins declared in its respective module.  An instance's **in-pins** act as sinks, while its **out-pins** act as sources.

The pin-holder may be the module itself, identified by the module's name or the symbol `MY`.  A module's **in-pins** act as sources for the wiring of the module, while its **out-pins** act as sinks (opposite of instance pins!)

#### PIN-IDs of scalars, buses, and bus components

In case of a scalar pin, the pin-id is simply the name of the pin.

If the pin is a pin-bus, the name of the bus may be used to represent all the pins in the bus, in sequential ascending order.

It is possible to represent a single pin of a pin-bus or several pins of a pin bus in arbitrary order, by specifying a list containing the name of the pin-bus followed by one or more numeric identifiers of a bus.  For instance, given a pin-bus declared as `(DATA 8)`, the wiring pin-d for the four odd elements of the bus is in descending order is `(DATA 7 5 3 1)`.

### A Simple example - a 2-bit counter
```
(module CTR2(loc) (&in CLK &out (OUT 2) COUT) ( 
"A simple 2-bit counter occupying a slice.  Note the OUT pin is a bus with two wires"
  (SLICEL sll :loc (0.0)  :CLKINV CLK :COUTUSED 0 :CYINIT CIN 
    :CY0G 0 :CYSELG G :DYMUX 1 :GYMUX GXO  :FFX |#FF| :FFX_INIT_ATTR INIT0 :|G:LUT:D=| #xAAAA
    :CY0F 0 :CYSELF F :DXMUX 1 :FXMUX:FXOR :FFY |#FF| :FFY_INIT_ATTR INIT0 :|F:LUT:D=| #xFF00)

  (wire my CLK his CLK) "'my' is this module, 'his' is the most-recently-defined instance"
  (wire my CIN his CIN) "carry path via BX"

  (wire her COUT to my COUT)            "'to' is optional"
  (wire his XQ   his F4 and my (OUT 0)  "'and' is optional" 
  (wire his YQ   his G1 my (OUT 1)) 

  (wire my vcc   sll F1  sll F2  sll F3
                 sll G2  sll G3  sll G4))
```

### Discussion

This syntax fits naturally into Lisp, and allows us to use the Lisp engine to deal with nastiness of language develpment.  More imporantly we instantly inherit all the power of Lisp - scoped variables, flexible parameter passing, conditional constructs, loops, and most importantly, macros.

The Xilinx usage of # and : characters is worked around using `|` to enclose symbols.  The G:LUT:D=0xAA expression is iffier as the Xilinx configuration parameters seem to be double-tiered.  For now I will treat `G:LUT:D=` as a single keyword parameter in order to let the value be just the contents of the LUT...

* The `&in` and `&out` keywords act much like Lisp's own lambda-keywords such as `&optional`.  The ability to use any number of in and out groups may be helpful later - buses and specialized groupings of pins may be inserted by macros or other generators.




