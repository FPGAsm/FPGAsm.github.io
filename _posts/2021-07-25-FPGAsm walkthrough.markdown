---
layout: post
title:  "FPGAsm compilation walkthrough"
date:   2021-07-25 16:00:00 -0700
categories: 
---

# FPGAsm Internals Overview

FPGAsm converts user's top module, expanding it using other user module definitions, and eventually reaching the bottom primitive-defs provided by the hardware-description XDLRC file.  The process consists of several distinct sequential tasks, roughly described below.  This is not an accurate description but an overview of the tasks at hand and strategies available.

## Construct a model of the FPGA hardware.

The XDLRC consists of two major sections: tile grid and primitive-defs.

### Tile Grid

The tile grid is a rectangular array consisting of tiles.  Each tile has a list of primitives contained within.  

### Primitive-defs

Primitive-defs appear as lowest-level module, complete with in and out pins, and may be instantiated in user modules.  They may be placed and hold configuration parameters.

## Process user module definitions.

We now create module definitions from user's source.  

Each module definition declares:
* In and Out pins;
* Parameters;
The module also keeps track of:
* Instance names, types, relative locations, and arguments
* Wiring connecting its own pins with instance pins

## Build the Circuit

### Expand into an Instance-based Model 

The module definitions form a hierarchy of containment, with the top module being the root of the tree and lowest-level modules containing only primitive-defs as leaf nodes.  This description is downward-only - module definitions specify what other modules instances are to be included, but not which modules contain it itself.

The first step is to traverse the definition tree from the top module, recursively following each requested instance type reference into its module definition, depth-first.  For each visited definition node we construct a corresponding instance structure, and link it into a mirroring tree of instances.  Instances keep a reference to the module definition and instance-local data.  

The instance-model mirrors the definition tree hierarchy, with one major difference: the module hierarchy describes each module contents as references to their respective module definitions (the 'type' of each requested instance in FPGAsm source).  There may be many references to the same module definition.  To contrast that, our instance model creates a unique **instance** datastructure for each visited node.  

Since each node in our tree is unique, each node can be linked to its container parent, allowing arbitrary traversal up or down the tree. 

So far each instance has the following fields:
* parent
* children
* definition
* instance-data

In practice, we need another piece of data for each node: the place in the container's definition where the node is instantiated.  This allows us to access its name and instantiation parameters, including location data.  The instantiation point also provides a reference to the definition (as instance 'type').  So we track that instead:

* parent
* children
* instantiation(name,definition)
* instance-data

The name of each instance is provided by the instantiation line in the containing module's definition.  However, these names are not unique.  In order to uniquely identify each instance, we use instance-pathnames (ipaths).  These look much like filenames in a filesystem, and are constructed by travelling up to the root node, collecting names along the way in reverse order.

All further operations will be performed by walking the instance-tree.

### Construct a target cell model

Using the cell description of the entire FPGA from the xdlrc file, we create an equivalent grid datastructure for tracking which FPGA features are occupied.  This allows us to detect overlaps in placement, and to verify that we are placing primitives that actually exist at requested locatons in the FPGA.

### Map and Place 

We now walk the instance-tree, keeping track of the locations.  The top instance is implicitly located at (0,0).  Each instantiation in the top definition provides an x-y offset which we apply and store as the node instance-data.  We do so recursively to the leaf nodes.  As we hit primitive-defs, we can use the grid location to verify that the primitive can be placed on the grid.  If so, we mark the appropriate cell in the cell model as occupied with the primitive.

Some primitive-defs (such as IOBs) don't use the grid, and are located at a specific pin locationthe FPGA.  The location for these is provided somewhere along the line, and the end result is similar.

In addition to the XY grid position, we recover the name of the cell containing the primitive.  For instance, on xc3s200ft256 pin T9 is at the cell named BIOIC11.

### Process the parameters

As we walk the instance tree, we check each instantiation point for parameters.  Parameters are stored as key-value pairs, and we accumulate them as we drill down, storing each in its instance.

When the primitive-def is reached, we process the accumulated parameters in order to construct the required configuration string for the primitive.  The exact mechanism will be discussed elsewhere.

At this point, all the low-level primitive-defs had been identified, placed and parametrized, we can output the XDL insts that represent them.  


### Construct a wiring netlist

It is a basic requirement that every pin of every placed primitive is connected to something.  The wiring process therefore consists of traversing the list of all placed primitives and wiring each wire, one by one, constructing nets.  We can walk up and down the instance-tree, tracing the wiring information provided to construct nets.  Each wire instruction has a single source and one or more sinks.  As we trace a wire, we have to record the position of fan-out and return to process the remaining sinks.  

A net consists of a source and one or more sinks.  These endpoints must be appropriately-directed pins of primitives, or special endpoints such as 'vcc'.  We are not interested in any objects along the way -- only primitives.

To record a net we start with a primitive outpin and trace it (recursively backing up at fanout points) until we reach a primitive sink.  We keep track of instance-path and pin of each endpoint.

At this point we can keep track of primitive pins to catch a number of problems:
* Unconnected pins;
* multiple connections to a sink pin;

We can write the netlist into the xdl file.

We are done!

## Debugging

The resultant XDL file may be further processed to create a bitmap.  Xilinx tools may catch problems and output error messages.  Debugging is facilitated by the fact that every inst in the XDL file is named using the instance-path of the primitive.  If there is an instantiation problem, it is very easy to look up the module definitions from the bottom up and isolate the problem.

If there is a wiring problem, the XDL nets are likewise identified by the instance-path of the instance where the net originates.  Each sink is identified with the path of the target instance.

## Future directions

This is where the current C implementation ends.

An interactive Lisp implementation can do this and much more.  For instance:
* The technology xdlrc can be loaded once and used many times throughout the session;
* Modules may be edited interactively and verified for correctness;
* Reports of all kinds may be generated using live circuits;
* Simulation is possible
* A graphical interface may be constructed for interactive viewing and manipulation of the source modules, the FPGA substrate, the instantiated circuit, mplacing or mapping of the circuit (including back-annotation), etc.
* Syntax improvements;
* Variables, expressions, and arbitrary Lisp code execution at any point of the process;
and ultimately,
* Routing
* Bitstream generation

