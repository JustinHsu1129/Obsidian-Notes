# Design Specifications

A document that defines an explicit set of requirements to be satisfied by a product
+ Idea begins with specs, and can be represented with graphs, text, etc.

# HDL

Contains constructs to define hardware concurrency and timing
+ Modular, can be reused across projects-good for scale, design time, error rate, etc.

## Multiple levels of abstraction

![[Pasted image 20260909124316.png]]

Switch level = physical implementation details like physical location, etc.

Abstraction levels help manage complexity by defining the system with RTL (how data moves between registers and what logic controls it)

# RTL Coding

Method of describing synthesizable synchronous digital circuits using HDLs where the flow of data between hardware registers is controlled by clock signals
+ Define logic to transfer data between register elements in your design

![[Pasted image 20260909124828.png]]

At RTL level we describe how the operation itself is performed in hardware

# Functional Simulation

# Coverage Analysis

# Synthesis Stage

Process of transforming HDL to logic circuit based on a specific library-you can provide constraints in terms of optimizing the circuit for power, performance, area, etc.
+ Translation, mapping and optimization of HDL into a circuit

## Stylus CLI

A user interface that is common across all Cadence tools-easier to work across many Cadence products
+ `set_db` and `get_db` are common across all tools
+ common gui across all tools

## Genus

![[Pasted image 20260909130353.png]]

![[Pasted image 20260909131209.png]]

Turn on Genus gui by using `gui_show`. 