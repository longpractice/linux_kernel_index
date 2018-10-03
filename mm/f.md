# f

## __FIXADDR_TOP
end of fixed kernel mapping virtual address

## fixed_address
in x86, an enum holding all fixed address in fixed kernel mapping area.

## fixed_to_virt
in x86, calculates the virtual address of a fixmap constant

## __fixed_to_virt
in x86, used by fixed_to_virt. 
```
#define __fix_to_virt(x) (FIXADDR_TOP - ((x) << PAGE_SHIFT))
```
so the x is the number of pages counting down from the FIXADDR_TOP.

## Fixmaps
kernel virtual address space entries associated a fixed but freely selectable page in physical address space. In contrast to directly mapped pages that are associated with RAM memory by a fixed formula, the association between a virtual fixmap address and the position in RAM memory can be freely defined and is then always observed by the kernel.

The advantage of fixmap addresses is that at compilation time, the address acts like a constant whose physical address is assigned when the kernel is booted. The kernel also ensures that the page table entries of fixmaps are not flushed from the TLB during a context switch so that acess is always made via fast cache memory.

