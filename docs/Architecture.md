# Architecture

- little endian
- 64 bit flat addressing
- optional page-table based virtual memory
- kernel and user modes
- single-address-space architecture, with seperate permission-based memory protection


# Registers
## General Purpose
There are 32 general-purpose integer registers, each 64 bits in size.  
Register #0 is always 0 and ignores writes.

## Floating Point
There are 32 floating-point registers, each 64 bits in size.
Each register also has 5 status bits to indicate the standard IEEE floating point exceptions.
