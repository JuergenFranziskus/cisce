# System Control
Exception and Interrupt handling, as well as virtual memory is controlled
by a set of control registers which may only be accessed in kernel mode.



- 0x0: RootPageTable

    Specifies the high bits of the physical address of the root of the page table tree used for virtual to physical address translation.  
    This register is 51 bits in size.  
    If this register is zero, virtual memory is disabled.  
    When written to, the low 13 bits of the 64-bit operand are ignored, and the high 51 are stored.
