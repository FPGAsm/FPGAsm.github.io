---
layout: post
title:  "CL-FPGASM Syntax, first attempt"
date:   2021-07-27 12:35:00 -0700
categories: cl-fpgasm
---

## CL-FPGASM Syntax

This is a preliminary specification for CL-FPGASM module and instantiation syntax.

### Module Syntax
```
(MODULE <name> <mod-lambda-list> <inwires> <outwires> <body> )
```

| parameter | description |
| :-- | :-- |
| `<name>`             | name of the module |
| `<mod-lambda-list>`  | parameters for instantiation |
| `<inpins>`           | list of sinks for wiring |
| `<outpins>`          | list of sources for wiring |
| `<body>`             | module definition |

The pins parameters consist of a list of zero or more symbolic names of pins.  Buses are expressed as a cons pair containing a pin name symbol in car and the width of the bus in cdr.

The body is a list of arbitrary Lisp expressions evaluated at the time the module is defined.  The body **may** contain instantiation expressions, making the module a container of module instances.  The body **must** contain one or more wire expressions, providing internal wiring from the module's inpins to internal circuitry and module's outputs.

The body may contain strings, which will be treated as comments.

Modules **should** be defined in the top context, that is a null lexical environment, unless you really know what you are doing.

Upon module definition, CL-FPGASM:
* Creates a MODULE structure of same name, with module information and the entire source SEXP expression for later editing;

* Generates a macro of same name name and and module's lambda-list, to be used for instantiation of module's instances inside other modules.

### A Simple example - a 2-bit counter
```
(module CTR2(loc) (CLK CIN) ((OUT.2) COUT) (
  (SLICEL sll :loc (0.0)  :CLKINV CLK :COUTUSED 0 :CYINIT CIN 
    :CY0G 0 :CYSELG G :DYMUX 1 :GYMUX GXO  :FFX |#FF| :FFX_INIT_ATTR INIT0 :|G:LUT:D=| #xAAAA
    :CY0F 0 :CYSELF F :DXMUX 1 :FXMUX:FXOR :FFY |#FF| :FFY_INIT_ATTR INIT0 :|F:LUT:D=| #xFF00)

  (wire my CLK  his CLK)
  (wire my CIN  his CIN) "carry path via BX"

  (wire his COUT  my COUT)
  (wire his XQ    his F4  my (OUT 0) "close counter loop"
  (wire his YQ    his G1, my (OUT 1)) 

  (wire my vcc   sll F1  sll F2  sll F3
                 sll G2  sll G3  sll G4))
```

### Discussion

This syntax fits naturally into Lisp, and allows us to use the Lisp engine to deal with nastiness of language develpment.  More imporantly we instantly inherit all the power of Lisp - scoped variables, flexible parameter passing, conditional constructs, loops, and most importantly, macros.

The Xilinx usage of # and : characters is worked around using `|` to enclose symbols.  The G:LUT:D=0xAA expression is iffier as the Xilinx configuration parameters seem to be double-tiered.  For now I will treat `G:LUT:D=` as a single parameter in order to let the value be just the contents of the LUT...





