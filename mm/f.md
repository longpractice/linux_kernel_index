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

## for_each_memblock_type
```c
#define for_each_memblock_type(i, memblock_type, rgn)			\
	for (i = 0, rgn = &memblock_type->regions[0];			\
	     i < memblock_type->cnt;					\
	     i++, rgn = &memblock_type->regions[i])
```
actually goes through all the regions inside a certain memory type.

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

## free_area_init_core
page_alloc.c used by free_area_init_node

Set up the zone data structures:
- mark all pages reserved
- mark all memory queues empty
- clear the memory bitmaps

 NOTE: pgdat should get zeroed by caller.

 NOTE: this function is only called during early init.


calling sequence:

pgdat_init_internals(pgdat):
init several spinlocks, wait queues, lruvec and some configuration specific fields of pgdat

	
pgdat->per_cpu_nodestats = &boot_nodestats;

it then goes through all the zones, for each zone:



it firstly needs to calculate the freesize(inited as zone->present_pages). This affects the watermark and per-cpu initialisations. Firstly the memory_map will take some space, it is firstly substracted. Then it substract the size of the dma_reserve range. 

Then zone_init_internal to init the zone. The important part of zone_init_internal is that it calls the zone_pcp_init.

Then setup_usemap(pgdat, zone, zone_start_pfn, size) to allocate pageblock_flags of the zone

Then init_currently_empty_zone which calls the zone_init_free_lists(zone, zone_start_pfn, size) which inits buddy allocation related zone->free_area and zone->free_area->fee_list fields(zone->free_area[]->nr_free is set to zero). 

Then memmap_init




 
## free_area_init_node
called by free_area_init_nodes to init the pg_data_t of one node with a passed-param of pgdat. 
It firstly get the pg_data_t from nid using NODE_DATA which need to be provided by all architectures.

Then it calculates the start pfn and total number of pages using get_pfn_range_for_nid and calculate_node_totalpages. Note that calculate_node_totalpages sets zone->spanned_pages, zone->present_pages and zone->zone_start_pfn as well as pgdat->node_spanned_pages and pgdat->node_present_pages.


## free_area_init_nodes
quote mm.h
```c
/*
 * With CONFIG_HAVE_MEMBLOCK_NODE_MAP set, an architecture may initialise its
 * zones, allocate the backing mem_map and account for memory holes in a more
 * architecture independent manner. This is a substitute for creating the
 * zone_sizes[] and zholes_size[] arrays and passing them to
 * free_area_init_node()
 *
 * An architecture is expected to register range of page frames backed by
 * physical memory with memblock_add[_node]() before calling
 * free_area_init_nodes() passing in the PFN each zone ends at. At a basic
 * usage, an architecture is expected to do something like
 *
 * unsigned long max_zone_pfns[MAX_NR_ZONES] = {max_dma, max_normal_pfn,
 * 							 max_highmem_pfn};
 * for_each_valid_physical_page_range()
 * 	memblock_add_node(base, size, nid)
 * free_area_init_nodes(max_zone_pfns);
 *
 * free_bootmem_with_active_regions() calls free_bootmem_node() for each
 * registered physical page range.  Similarly
 * sparse_memory_present_with_active_regions() calls memory_present() for
 * each range when SPARSEMEM is enabled.
 *
 * See mm/page_alloc.c for more information on each function exposed by
 * CONFIG_HAVE_MEMBLOCK_NODE_MAP.
 */
```
Note that x86 should have CONFIG_HAVE_MEMBLOCK_NODE_MAP set.

void __init free_area_init_nodes(unsigned long *max_zone_pfn)
defined in /mm/page_alloc.c

init all pg_data_t and zone data. @max_zone_pfn: an array of max PFNs for each zone

This will call free_area_init_node() for each active node in the system. Using the page ranges provided by memblock_set_node(), the size of each zone in each node and their holes is calculated. If the maximum PFN between two adjacent zones match, it is assumed that the zone is empty. For example, if arch_max_dma_pfn == arch_max_dma32_pfn, it is assumed that arch_max_dma32_pfn has no pages. It is also assumed that a zone starts where the previous one ended. For example, ZONE_DMA32 starts at arch_max_dma_pfn.

## free_bootmem(unsigned long addr, unsigned long size)
Only whole pages can be freed because the bootmem allocator does not keep any information about page divisions. Delegate the work to __free_bootmem_core, which calculates the pages whose contents are fully held in the area to be freed. So the if one page is freed only partly once and it will not be freed when the remaining part is freed later. This is normally not a big problem because free_bootmem is very rarely used. Most memory areas allocated during system initialization are intended for basic data structures that are needed throughout kernel run time and are therefor never relinquished.

## free_initmem
responsible for freeing the memory area defined for initialization purposes and returning the pages to the buddy system. 


## freelist (slab)
A freelist for a slab in a slab allocator is an array(not a linked list!) of elements of type `freelist_idx_t` with each element holding one free object index. When a slab is firstly allocated, the freelist should be initialized to hold indices of all the objects(it might be shuffled when initializing). We could use (colour + freelist_id * size) to get the free object location inside the slab(note the `size` here is the stride, the difference of addresses of adjacent objects, not the object_size itself). 

In the beginning, when we have 0 active objects in the slab and we want to use one object, we take the first element of the freelist array. When we want to use another element, we use the second element in the freelist array, etc...  We therefore could easily find another free object by indexing the freelist with number of active objects(this is stored in page->active). 

When we free an object, we do --page->active and put the id of the newly freed object to our freelist[page->active]. This way, when we use it next time, it tends to be still hot.

The memory location is found or allocated in routine alloc_slabmgmt during refilling of new slabs. 

## __free_one_page
the cornerstone of memory freeing. The relevant area is added to the appropriate free_area list of the buddy system. When buddy pairs are freed, the function coalesces them into a contiguous area that is then placed in the next higher free_area list. If this reunites a further buddy pair, it is also coalesced and moved to a higher list. This procedure is repeated until all possible buddy pairs have been coalesced and the changes have been propagated upwards as far as possible.


## __free_pages

the base function to implement all buddy kernel freeing API.
__free_pages first establishes whether a single page or a larger contiguous block is to be freed. If a single page is freed, it is not returned to the buddy system but is placed in the per-CPU cache -- in the warm list for all pages that are highly likely to reside in the CPU cache. For this purpose, the kernel provides the free_unref_page helper function.

## free_unref_page

kernel buddy system to free a 0-order page.



