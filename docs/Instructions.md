# Instruction Set

Instructions have a complex variable-length encoding.  
An instruction consists of one or more atomic 'Components'.  
Each 'Component' fundamentally provides some information about the semantics of the instruction,
such as an opcode or a register reference.  
Some 'Components' may (optionally) require other 'Components' to be present in the instruction.
For instance, an 'Operand Specifier Component' whose value indicates that one operand is an immediate
value then requires an 'Immediate Component' to be present.


## Memory Access Sizes
All operations involving the general-purpose and floating-point registers act upon the whole 8-byte value.  
There are no 'add byte', 'add short' or 'add int' instructions.  
For this reason, when an instruction needs to load a value from memory, the amount of bytes to load
is an aspect of the memory operand, not of the operation being performed. All memory operands are always extended to 8 bytes
before being used in the operation (sign or zero-extended for integers, NaN-boxed for floats).

## Components

### Opcode
The 'Opcode' is the always the first and only required Component in any instruction.  
It is a single byte in size.

### BinOp Operand Specifier
The BinOp Operand Specifier ('BinOp') specifies the operands for a binary operation.  
It is 2 bytes in size, and may indicate either a Destination and two Source registers,
or a Destination and Source register plus the addressing mode for a Memory Operand.

It has two forms:

#### Three-Register Form

| Bits | Meaning |
| ---- | ------- |
| 0-4 | Destination Register |
| 5-9 | First Source Register |
| 10 - 14 | Second Source Register |
| 15 | Always 0 |

#### Two-Register Memory Form

| Bits | Meaning |
| ---- | ------- |
| 0-4 | Destination Register |
| 5-9 | Source Register |
| 10 | Operand Flip |
| 11 - 14 | Addressing Mode |
| 15 | Always 1 |

If bit #10 (Operand Flip) is clear, the Source Register is the first (left) operand and the Memory Operand the second (right).
If bit #10 is set, the reverse.  
If the order is irrelevant for the operation, it shall be clear.

The following addressing modes are possible:

| Addressing Mode | Symbolic | Requires |
| --------------- | -------- | -------- |
| 0x0 | | |
| 0x1 | Imm8 | 'Imm8' |
| 0x2 | Imm16 | 'Imm16' |
| 0x3 | Imm32 | 'Imm32' |
| 0x4 | [ Base ] | 'Base' |
| 0x5 | [ Base + Imm8 ] | 'Base', 'Imm8' |
| 0x6 | [ Base + Imm16 ] | 'Base', 'Imm16' |
| 0x7 | [ Base + Imm32 ] | 'Base', 'Imm32' |
| 0x8 | [ Base + Index + Imm2 ] | 'BaseIndex2' |
| 0x9 | [ Base + Index + Imm10 ] | 'BaseIndex2', 'Imm8' |
| 0xA | [ Base + Index + Imm18 ] | 'BaseIndex2', 'Imm16' |
| 0xB | [ Base + Index + Imm34 ] | 'BaseIndex2', 'Imm32' |
| 0xC | 1 | |
| 0xD | -1 | |


Where 'BaseIndex2' is combined with an immediate Component, the actual offset is calculated as 'sext(Imm2 + (ImmN << 2))'.

### Base
The 'Base' Component specifies a base address for a Memory Operand, which might be either a general register, the program counter,
or their sum.  
It also specifies the access size so that either 1, 2, 4 or 8 bytes are loaded.  
If less than 8 bytes are loaded, the value is zero-extended to 8 bytes.

The 'Base' Component has the following form:

| Bits | Meaning |
| ---- | ------- |
| 0-4 | Base Register |
| 5 | Rip-Relative |
| 6-7 | Access Size |

If bit #5 is set, the program counter is added to the Base register to form the address.
Note that r0 is always equal to zero, thereby using only the program counter as a base.

Bits 6-7 (Access Size) are interpreted as follows:

| Value | Access Size |
| ----- | ----------- |
| 0 | 1 Byte |
| 1 | 2 Bytes |
| 2 | 4 Bytes |
| 3 | 8 Bytes |

### BaseIndex2
The 'BaseIndex2' Component is an extension of the 'Base' Component which also specifies an index register and a 2-bit immediate value.  
Unlike the 'Base' Component, it also allows to specify the extension method for access sizes less than 8.

It has the following form:

| Bits | Meaning |
| ---- | ------- |
| 0-7 | Same layout as 'Base' Component |
| 8 - 12 | Index Register |
| 13 | Extension Method |
| 14 - 15 | 2-bit Immediate Value |


If bit #13 is clear, accesses of size less than 8 are zero-extended, same as the regular 'Base' Component.  
If bit #13 is set, they are instead sign-extended.  
If the access size is 8, bit #13 shall be clear.

### LoadStore
The 'Store' Component specifies both Source/Destination and addressing mode for store and load instructions.  
It is one byte in size and has the following form:

| Bits | Meaning |
| ---- | ------- |
| 0-4 | Source Register |
| 5 - 7 | Addressing Mode |

The addressing modes 0-7 correspond to the 'BinOp' addressing modes 0x4-0xB.

## Instruction List

- 0x0: Illegal

    Permanently illegal/reserved to make accidentally executing all-zero memory panic

- 0x4: No Operation

- 0x5: Add Integers

    Add the two 64-bit operands.
    All register operands refer to the integer registers.

- 0x6: Subtract Integers

    Subtract the second operand from the first.
    All register operands refer to the integer registers.

- 0x7: Store Integer

    Requires: 'LoadStore'.  
    Stores part of the specified integer source register to memory.  
    The access size is indicated by the 'Base' or 'BaseIndex2' Component, indirectly required by 'Store'.

- 0x8: Load Integer

    Requires: 'LoadStore'.  
    Load a value from memory into the specified integer register.  
    The access size is indicated by the 'Base' or 'BaseIndex2' Component, indirectly required by 'Store'.

- 0x10: And

    Requires: 'BinOp'.  
    Bitwise-And the two Source operands and store result into the Destination register.  
    All register operands refer to the integer registers.

- 0x11: Or

    Requires: 'BinOp'.  
    Bitwise-Or the two Source operands and store result into the Destination register.  
    All register operands refer to the integer registers.

- 0x12: Xor

    Requires: 'BinOp'.  
    Bitwise-Xor the two Source operands and store result into the Destination register.  
    All register operands refer to the integer registers.

- 0x13: Multiply Words

    Requires: 'BinOp'.
    Multiply the first integer operand by the second and store the low 8 bytes of the 16-byte result.

- 0x14: Secondary Integer Operation

    Requires: 'Func'.  
    Perform an integer operation based on the 'Func' field.  

    - 0x00 thru 0x1F: DivMod

        Requires: 'BinOp'.  
        Divide or modulo the first operand by the second.  
        Details of operation are given by the low 4 bits of the 'Func' field.

        Bits 0-1 give the size of operation, values of 0 thru 3 standing for operation sizes of one, two, four and eight bytes respectively.  
        If the size is less than 8 bytes, the high bits of the operands are ignored, and the result is extended to 8 bytes based on the signedness of the operation.  
        The operation's signedness is given by bit #2 of 'Func'. If the bit is set, the operands are interpreted as signed integers. Otherwise, they are unsigned.  
        Bit #3 selects the modulo operation if set, division if cleared.
    
    - 0x20 thru 0x2F: Multiply

        Requires: 'BinOp'.  
        Multiply the first operand by the second.  
        Details of the operation are given by the low 4 bits of the 'Func' field.

        Bits 0-1 give the size of operation, values of 0 thru 3 standing for sizes of one, two, four and eight bytes respectively.  
        If the size is less than 8 bytes, the result is twice the operand's size. If the result's size is less than 8 bytes, it is extended based on the signedness
        of the operation.  
        If the size is 8 bytes, the result is actually the high 8 bytes of the 16-byte result that is gotten from an 8 by 8 byte multiplication.  
        The signedness op the operation is given by bits 2-3 with the following values:
        | Value | Signedness | Extension of Result |
        | ----- | ---------- | ------------------- |
        | 0 | Unsigned | Zero |
        | 1 | Signed | Sign |
        | 2 | Signed x Unsigned | Sign |
        | 3 | Invalid | N/A |


## Comments

- (Early in instruction set definition): On seperate math sizes

    I am not sure whether the many different multiplication and division instructions are necessary.  
    They may be removed at some point or something.
