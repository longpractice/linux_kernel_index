# M


## managed_zone
```c
/*
 * Returns true if a zone has pages managed by the buddy allocator.
 * All the reclaim decisions have to use this function rather than
 * populated_zone(). If the whole zone is reserved then we can easily
 * end up with populated_zone() && !managed_zone().
 */
static inline bool managed_zone(struct zone *zone)
{
	return zone->managed_pages;
}
```

## MAX_NUMNODES
defines maximum number of nodes

## MAX_NR_ZONES
defines maximum number of zones per node

## MAX_ZONES_PER_ZONELIST
MAX_NUMNODES * MAX_NR_ZONES

## memblock_add


## mem_init
routine used in start_kernel(), architecture-specific. It disables the bootmem allocator and perform the transition to the actual memory management functions.

## mk_pte
Must be implemented by all architectures.
creates a pte entry; a page instance and the desired page access permissions must be passed as parameters

## mem_map
mem_map is an global array holding all page frames `struct page`

## mm_init
routine used in start_kernel(). Setup kernel memory allocators.
It will call mem_init() which is architecture specific. In x86-32, it will call free_all_bootmem(() and x86_init.hyper.init_after_bootmem().
