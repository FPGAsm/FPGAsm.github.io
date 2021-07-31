---
layout: post
title:  "CL-FPGASM Syntax Rev.03"
date:   2021-07-31 12:37:00 -0700
categories: cl-fpgasm
---
This is a preliminary specification for CL-FPGASM module and instantiation syntax.

## Module Syntax
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

The **body** is a list of (almost) arbitrary Lisp expressions, and usually contains one or more **INFRASTRUCTURE-CLAUSES** such as `INST` and `WIRE` clauses.  Except for infrastructure clauses, the body is evaluated at the time the module is parametrized (see **Parametrization**). 

Modules **should** be defined in the top context, that is, at the top level and in a null lexical environment, unless you really know what you are doing.

During module definition, CL-FPGASM:
* Creates a MODULE structure of same name, with module information and source;
* Populates the structure with INST structures and WIRE information from the infrastructure clauses;
* Compiles a parametrization function from the body

### Instantiation syntax
```
(INST <inst-name> <type> <args>)
```
| inst-name | name of this instance |
| type | module being instantiated |
| args | instantiation arguments |

The name of the instance must not compete with any other in-scope names, module names, or reserved words `TO`, `AND`, `HIS`, `HER`, `THEIR`, `GND` and `VCC`.

An `INST` clause **must** be inside a module definition, and is an infrastructural element.  It is used during module definition to construct an INST structure in the containing module's instance array.  

The instantiation arguments (args) **MUST** match the mod-lambda-list, using normal Lisp rules.   Args are not evaluated until parametrization.   An argument may be an atom or a valid Lisp expression in the current scope.

### Wiring syntax

```
(WIRE <srr-pinholder> <src-pin-id> [/to/and] <sink-pinholder> <sink-pin-id> ...)
```  

| src-pinholder | name of the object containing the source pin or pin-bus |
| src-pin-id | pin identifier of the source pin or pin-bus |
| `to` `and`  | optional syntax sugar for readability |
| sink-pinholder | name of object containing the sink pin or pin-bus |
| sink-pin-id | pin identifier of the sink pin or pin-bus |

The body of the module definition with pins must contain a complete wiring network in which every mod-pin is terminated as a source or a sink, using WIRE instructions.  Each pin must have a single wire attached to it.  WIRE expressions are infrastructural, and are processed during the definition of the containing module.

Each wire has a single **source** and one or more **sink** endpoint.  The endpoints are represented by two sequential items: the pinholder and the pin-id within it.  If the source endpoint is a single pin, each sink endpoint must likewise be a single pin.  If the source endpoint is a group of pins, then each sink endpoint must be a group with the same number or pins.  

Group endpoints represent the wiring of buses, and each pin of a group source endpoint is wired to the corresponding pin in each group sink endpoint.  

#### PIN-HOLDER

The pin-holder must be an instance-name of an INST previously declared in this module.  The most-recently-defined instance may also be referred to by pronouns `HIS`, `HER`, `ITS` or `THEIR`.  Each instance has a set of pins declared in its the instance-type-module.  An instance's **in-pins** act as sinks, while its **out-pins** act as sources.

The pin-holder may be the module itself, identified by the module's name or the symbol `MY`.  A module's **in-pins** act as sources for the wiring of the module, while its **out-pins** act as sinks (opposite of instance pins!)

#### PIN-IDs of scalars and groups

A pin-id must a symbol or a list containing a symbol and one or more integer indices.

| name | a single pin |
| name | all pins of a bus in ascending order |
| ( name idx ) | an single bus pin at index |
| ( name idx1 ... ) | a grouping of indexed pins from a bus in the order specified |

In case of a scalar pin, pin-id is simply the name of the pin.

If the pin is a **pin-bus**, the entire bus may be wired using the bus name.  

The grouping syntax allows arbitrary ordering of the indexed pins of a single bus in entirety or any subset of bus pins.  

### A Simple example - a 2-bit counter
```
(module CTR2(loc) (&in CLK &out (OUT 2) COUT) ( 
"A simple 2-bit counter occupying a slice.  Note the OUT pin is a bus with two wires"
  (inst SLICEL sll :loc (0.0)  :CLKINV CLK :COUTUSED 0 :CYINIT CIN 
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

### NotesGASM Syntax V02.markdown

The FPGASM syntax fits naturally into Lisp. For instance, the `&in` and `&out` pin-list keywords act much like Lisp's own lambda-list-keywords. 



