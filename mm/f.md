# f

## fallbacks
discribed as in the code
```c
/*
 * This array describes the order lists are fallen back to when
 * the free lists for the desirable migrate type are depleted
 */
static int fallbacks[MIGRATE_TYPES][4] = {
	[MIGRATE_UNMOVABLE]   = { MIGRATE_RECLAIMABLE, MIGRATE_MOVABLE,   MIGRATE_TYPES },
	[MIGRATE_RECLAIMABLE] = { MIGRATE_UNMOVABLE,   MIGRATE_MOVABLE,   MIGRATE_TYPES },
	[MIGRATE_MOVABLE]     = { MIGRATE_RECLAIMABLE, MIGRATE_UNMOVABLE, MIGRATE_TYPES },
#ifdef CONFIG_CMA
	[MIGRATE_CMA]         = { MIGRATE_TYPES }, /* Never used */
#endif
#ifdef CONFIG_MEMORY_ISOLATION
	[MIGRATE_ISOLATE]     = { MIGRATE_TYPES }, /* Never used */
#endif
};
```


## __flush_tlb_all()
flush the tlb entries. In contrast to TLB flushes during context swtiches, pages with a _PAGE_GLOBAL bit are also flushed.

## __FIXADDR_TOP
end of fixed kernel mapping virtual address

## fixed_address
in x86, an enum holding all fixed address in fixed kernel mapping area.

## fixed_to_virt
in x86, calculates the virtual address of a fixmap constant

## __fixed_to_virt
in x86, used by fixed_to_virt. 
```c
#define __fix_to_virt(x) (FIXADDR_TOP - ((x) << PAGE_SHIFT))
```
so the x is the number of pages counting down from the FIXADDR_TOP.

## Fixmaps
kernel virtual address space entries associated a fixed but freely selectable page in physical address space. In contrast to directly mapped pages that are associated with RAM memory by a fixed formula, the association between a virtual fixmap address and the position in RAM memory can be freely defined and is then always observed by the kernel.

The advantage of fixmap addresses is that at compilation time, the address acts like a constant whose physical address is assigned when the kernel is booted. The kernel also ensures that the page table entries of fixmaps are not flushed from the TLB during a context switch so that acess is always made via fast cache memory.

## free_all_bootmem 
diable the bootmem allocator and make the buddy system allocator assume responsibility of memory management. It relies on the function of __free_page. Th enable the pages to be incorporated in the data structures of the buddy system, where they are managed as free pages and are available for allocation. The memory for bitmap in bootmem is also then freed.

## free_area
```c
struct free_area {
	struct list_head	free_list[MIGRATE_TYPES];
	unsigned long		nr_free;
};
```
contained in each zone, donotes the free memory of the buddy system of a certain order
nr_free donates the number of 2^order pages.

free_list is the used to link the segments of memories. Only the first page of the memory area is linked.



## free_bootmem(unsigned long addr, unsigned long size)
Only whole pages can be freed because the bootmem allocator does not keep any information about page divisions. Delegate the work to __free_bootmem_core, which calculates the pages whose contents are fully held in the area to be freed. So the if one page is freed only partly once and it will not be freed when the remaining part is freed later. This is normally not a big problem because free_bootmem is very rarely used. Most memory areas allocated during system initialization are intended for basic data structures that are needed throughout kernel run time and are therefor never relinquished.

## free_initmem
responsible for freeing the memory area defined for initialization purposes and returning the pages to the buddy system. 
