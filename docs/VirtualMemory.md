# Virtual Memory
CiscE uses a page-table based virtual memory system.  
All memory addresses used by programs are virtual addresses, and need to be translated into physical addresses
before a memory access can happen.


# Form of Physical and Virtual Addresses
Virtual Memory is divided into pages 16KiB in size. The low 14 bits of a virtual address give an offset into a page,
called the PageOffset, whereas the high 50 bits select the page, called the PageIndex.  
This arrangement is mirrored with physical addresses:  
Physical pages are instead called frames, and a physical address therefor consists of a 14-bit FrameOffset
and a 50-bit FrameIndex.  
Address translation derives a FrameIndex from a PageIndex. The FrameOffset is always equal to the PageOffset.

# Enabling and Disabling
Address translation may be enabled and disabled by writing to the RootPageTable control register.  
If address translation is disabled, the FrameIndex is simply equal to the PageIndex.


# Page Tables
A page table is a datastructure that describes the mapping of a part of a virtual address to a physical one.  
The process of address translation usually involves more than one page tables:  
Each page table maps 10 bits from the PageIndex to 10 bits for the FrameIndex, and also specifies
the address of the page table to use for the next 10 bits.  
This process starts at the Root Page Table for the highest 10 bits.  
Since a page table is indexed by 10 bits, it has 1024 entries. Each entry is 8 bytes in size, making a whole
page table 8KiB in size. Page tables must be aligned to their size.  
Since a PageIndex is 50 bits wide, and each page table maps 10 of those bits, address translation
may involve up to 5 page tables.

| Bits | Meaning |
| ---- | ------- |
| 0 - 9 | FrameIndex Bits |
| 10 | Present |
| 11 | Write |
| 12 | Terminate Early |
| 13 - 63 | Next Page Table Address |

If the Present bit (#10) is clear, the address is not mapped and translation throws a fault.  
If the Write bit (#11) is clear in any of the Page Table Entries used for translation of an address,
the memory area is read-only and writes to it throw a fault.  
If the Terminate Early bit (#12) is set, address translation finishes early and the remaining parts of the PageIndex are identity-mapped.  
In this case, the low 4 bits of the Next Page Table Address (bits 13 - 16) are instead used to indicate one of 10 bits masks.  
Where a bit in that mask is clear, the corresponding bit of the FrameIndex part mapped by this entry is taken as usual from the entry's FrameIndex Bits (0-9).  
Where a bit in the mask is set however, the corresponding address bit is instead directly taken from the original virtual address, ie it is identity-mapped.  
The provided bit-masks only allow a bit to be set when all lower bits are also set.  Using this scheme, any power of two larger than 16KiB can be used as a Large Page size.  

| Bits 13-16 Value | Bit Mask |
| ---------------- | -------- |
| 0 | 0000000000 |
| 1 | 0000000001 |
| 2 | 0000000011 |
| 3 | 0000000111 |
| 4 | 0000001111 |
| 5 | 0000011111 |
| 6 | 0000111111 |
| 7 | 0001111111 |
| 8 | 0011111111 |
| 9 | 0111111111 |
| 10 - 15 | Invalid, Throw Fault |
