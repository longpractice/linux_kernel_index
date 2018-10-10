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

## MAX_ORDER
defines the maximum number of orders. The order could be from 0 to `MAX_ORDER-1`

it normally 11 but can be overwriten by CONFIG_FORCE_MAX_ZONEORDER.

the order 
```c
#ifndef CONFIG_FORCE_MAX_ZONEORDER
#define MAX_ORDER 11
#else
#define MAX_ORDER CONFIG_FORCE_MAX_ZONEORDER
#endif
```

## MAX_ZONES_PER_ZONELIST
MAX_NUMNODES * MAX_NR_ZONES

## memblock_add


## mem_init
routine used in start_kernel(), architecture-specific. It disables the bootmem allocator and perform the transition to the actual memory management functions.

## migratetype

a enum to donate the type of free area according to mobility. The free list is divided according to mobility so that fragmentation problem is releaved. 

enum migratetype {
	MIGRATE_UNMOVABLE,
	MIGRATE_MOVABLE,
	MIGRATE_RECLAIMABLE,
	MIGRATE_PCPTYPES,	/* the number of types on the pcp lists */
	MIGRATE_HIGHATOMIC = MIGRATE_PCPTYPES,
#ifdef CONFIG_CMA
	MIGRATE_CMA,
#endif
#ifdef CONFIG_MEMORY_ISOLATION
	MIGRATE_ISOLATE,	/* can't allocate from here */
#endif
	MIGRATE_TYPES
};


note MIGRATE_ISOLATE is a special virtual zone that is required to move physical pages across NUMA nodes. On large systems, it can be beneficial to bring physical pages closer to the CPUs that use them most. 


## mk_pte
Must be implemented by all architectures.
creates a pte entry; a page instance and the desired page access permissions must be passed as parameters

## mem_map
mem_map is an global array holding all page frames `struct page`

## mm_init
routine used in start_kernel(). Setup kernel memory allocators.
It will call mem_init() which is architecture specific. In x86-32, it will call free_all_bootmem(() and x86_init.hyper.init_after_bootmem().

