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

## find_suitable_fallback
check whether there is a suitable fallback freepage with requested order. If only_stealable is true, this function fallback_mt only if we can steal other freepages all together. This would help to reduce fragmentation due to mixed migratetype pages in one pageblock.

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

## free_area_init_nodes

void __init free_area_init_nodes(unsigned long *max_zone_pfn)

init all pg_data_t and zone data. @max_zone_pfn: an array of max PFNs for each zone

This will call free_area_init_node() for each active node in the system. Using the page ranges provided by memblock_set_node(), the size of each zone in each node and their holes is calculated. If the maximum PFN between two adjacent zones match, it is assumed that the zone is empty. For example, if arch_max_dma_pfn == arch_max_dma32_pfn, it is assumed that arch_max_dma32_pfn has no pages. It is also assumed that a zone starts where the previous one ended. For example, ZONE_DMA32 starts at arch_max_dma_pfn.

## free_bootmem(unsigned long addr, unsigned long size)
Only whole pages can be freed because the bootmem allocator does not keep any information about page divisions. Delegate the work to __free_bootmem_core, which calculates the pages whose contents are fully held in the area to be freed. So the if one page is freed only partly once and it will not be freed when the remaining part is freed later. This is normally not a big problem because free_bootmem is very rarely used. Most memory areas allocated during system initialization are intended for basic data structures that are needed throughout kernel run time and are therefor never relinquished.

## free_initmem
responsible for freeing the memory area defined for initialization purposes and returning the pages to the buddy system. 

## __free_one_page
the cornerstone of memory freeing. The relevant area is added to the appropriate free_area list of the buddy system. When buddy pairs are freed, the function coalesces them into a contiguous area that is then placed in the next higher free_area list. If this reunites a further buddy pair, it is also coalesced and moved to a higher list. This procedure is repeated until all possible buddy pairs have been coalesced and the changes have been propagated upwards as far as possible.


## __free_pages

the base function to implement all buddy kernel freeing API.
__free_pages first establishes whether a single page or a larger contiguous block is to be freed. If a single page is freed, it is not returned to the buddy system but is placed in the per-CPU cache -- in the warm list for all pages that are highly likely to reside in the CPU cache. For this purpose, the kernel provides the free_unref_page helper function.

## free_unref_page

kernel buddy system to free a 0-order page.



