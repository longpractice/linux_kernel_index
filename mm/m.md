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

## max_zone_pfn
an array showing maximum pfn for each zone, it is used by free_area_init_nodes to set up data structure for each node and zone.

## memblock

Good memblock overview doc from memblock.c:
It is also the name of the same global value of type `memblock` which is used for containing the memblock of the system.

Below are the comments from memblock.c.

---
Memblock is a method of managing memory regions during the early
boot period when the usual kernel memory allocators are not up and
running.
 
Memblock views the system memory as collections of contiguous
regions. There are several types of these collections:
 
* ``memory`` - describes the physical memory available to the
  kernel; this may differ from the actual physical memory installed
  in the system, for instance when the memory is restricted with
  ``mem=`` command line parameter
* ``reserved`` - describes the regions that were allocated
* ``physmap`` - describes the actual physical memory regardless of
  the possible restrictions; the ``physmap`` type is only available
  on some architectures.
 
Each region is represented by `struct memblock_region` that
defines the region extents, its attributes and NUMA node id on NUMA
systems. Every memory type is described by the `struct
memblock_type` which contains an array of memory regions along with
the allocator metadata. The memory types are nicely wrapped with
`struct memblock`. This structure is statically initialzed
at build time. The region arrays for the "memory" and "reserved"
types are initially sized to %INIT_MEMBLOCK_REGIONS and for the
"physmap" type to %INIT_PHYSMEM_REGIONS.
The `memblock_allow_resize` enables automatic resizing of
the region arrays during addition of new regions. This feature
should be used with care so that memory allocated for the region
array will not overlap with areas that should be reserved, for
example initrd.
 
The early architecture setup should tell memblock what the physical
memory layout is by using `memblock_add` or
`memblock_add_node` functions. The first function does not
assign the region to a NUMA node and it is appropriate for UMA
systems. Yet, it is possible to use it on NUMA systems as well and
assign the region to a NUMA node later in the setup process using
`memblock_set_node`. The `memblock_add_node`
performs such an assignment directly.
 
Once memblock is setup the memory can be allocated using either
memblock or bootmem APIs.
 
As the system boot progresses, the architecture specific
`mem_init` function frees all the memory to the buddy page
allocator.
 
If an architecure enables %CONFIG_ARCH_DISCARD_MEMBLOCK, the
memblock data structures will be discarded after the system
initialization compltes.

---

## memblock_add
add memory range to `memblock.memory` memory block type.

## memblock_add_range
in memblock.c
Add a new range to a memblock_type. Note that it does not affect existing regions(this is not true actually since merging may happen later) and if necessary it will double the allocated regions array until enough number of regions is enough. There might be multiple new regions added. The input region may overlap with one or more existing region in which case the new region will split to avoid that overlapping. Compatible neighbor regions will eventually be merged.

```c
/**
 * memblock_add_range - add new memblock region
 * @type: memblock type to add new region into
 * @base: base address of the new region
 * @size: size of the new region
 * @nid: nid of the new region
 * @flags: flags of the new region
 *
 * Add new memblock region [@base, @base + @size) into @type.  The new region
 * is allowed to overlap with existing ones - overlaps don't affect already
 * existing regions.  @type is guaranteed to be minimal (all neighbouring
 * compatible regions are merged) after the addition.
 *
 * Return:
 * 0 on success, -errno on failure.
 */
int __init_memblock memblock_add_range(struct memblock_type *type,
				phys_addr_t base, phys_addr_t size,
				int nid, enum memblock_flags flags)
```

## memblock_flags
a property of a certain memblock_region 
```c
/**
 * enum memblock_flags - definition of memory region attributes
 * @MEMBLOCK_NONE: no special request
 * @MEMBLOCK_HOTPLUG: hotpluggable region
 * @MEMBLOCK_MIRROR: mirrored region
 * @MEMBLOCK_NOMAP: don't add to kernel direct mapping
 */
enum memblock_flags {
	MEMBLOCK_NONE		= 0x0,	/* No special request */
	MEMBLOCK_HOTPLUG	= 0x1,	/* hotpluggable region */
	MEMBLOCK_MIRROR		= 0x2,	/* mirrored region */
	MEMBLOCK_NOMAP		= 0x4,	/* don't add to kernel direct mapping */
};
```

## memblock_insert_region
put a certain region at a certain index after shifting the following region by one region. 

## memblock_isolate_range
```c
/**
 * memblock_isolate_range - isolate given range into disjoint memblocks
 * @type: memblock type to isolate range for
 * @base: base of range to isolate
 * @size: size of range to isolate
 * @start_rgn: out parameter for the start of isolated region
 * @end_rgn: out parameter for the end of isolated region
 *
 * Walk @type and ensure that regions don't cross the boundaries defined by
 * [@base, @base + @size).  Crossing regions are split at the boundaries,
 * which may create at most two more regions.  The index of the first
 * region inside the range is returned in *@start_rgn and end in *@end_rgn.
 *
 * Return:
 * 0 on success, -errno on failure.
 */
static int __init_memblock memblock_isolate_range(struct memblock_type *type,
					phys_addr_t base, phys_addr_t size,
					int *start_rgn, int *end_rgn)
```

## memblock_region
one memblock_region is a segment of contiguous memory which is used for composing one memblock_type(which is not a type enum, instead it is an array of memory regions with same type).
```c
/**
 * struct memblock_region - represents a memory region
 * @base: physical address of the region
 * @size: size of the region
 * @flags: memory region attributes
 * @nid: NUMA node id
 */
struct memblock_region {
	phys_addr_t base;
	phys_addr_t size;
	enum memblock_flags flags;
#ifdef CONFIG_HAVE_MEMBLOCK_NODE_MAP
	int nid;
#endif
};
```

## memblock_reserve
add a certain memory range to memblock.reserved memory type.

## memblock_set_node
set node to a certain range of memory in a memblock type. Note that since the memblock type regions may cover the pased-in range boundaries, memblock_isolate_range has to be called to split these regions so that the node could be set for the splited regions
From memblock.c:
```
The early architecture setup should tell memblock what the physical memory layout is by using :c:func:`memblock_add` or :c:func:`memblock_add_node` functions. The first function does not assign the region to a NUMA node and it is appropriate for UMA systems. Yet, it is possible to use it on NUMA systems as well and assign the region to a NUMA node later in the setup process using :c:func:`memblock_set_node`. The :c:func:`memblock_add_node` performs such an assignment directly.
```

## memblock_type
Note that a memblock_type is not a certain "type". It is a collection of regions with same type(see `memblock` for certain types).

The number of regions allocated is always power of 2(see `memblock_double_array`) and does not necessarily equal to the number of regions that actually exists. Therefore two fields, max and cnt are needed.
```c
/**
 * struct memblock_type - collection of memory regions of certain type
 * @cnt: number of regions
 * @max: size of the allocated array
 * @total_size: size of all regions
 * @regions: array of regions
 * @name: the memory type symbolic name
 */
struct memblock_type {
	unsigned long cnt;
	unsigned long max;
	phys_addr_t total_size;
	struct memblock_region *regions;
	char *name;
};
```
## mem_init
routine used in start_kernel(), architecture-specific. It disables the bootmem allocator and perform the transition to the actual memory management functions.

## migratetype

a enum to donate the type of free area according to mobility. The free list is divided according to mobility so that fragmentation problem is releaved. 
```c
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
```

note MIGRATE_ISOLATE is a special virtual zone that is required to move physical pages across NUMA nodes. On large systems, it can be beneficial to bring physical pages closer to the CPUs that use them most. 


## mk_pte
Must be implemented by all architectures.
creates a pte entry; a page instance and the desired page access permissions must be passed as parameters

## mem_map
mem_map is an global array holding all page frames `struct page`

## mm_init
routine used in start_kernel(). Setup kernel memory allocators.
It will call mem_init() which is architecture specific. In x86-32, it will call free_all_bootmem(() and x86_init.hyper.init_after_bootmem().

## movable_zone
a global variable showing the physical zone from which pages for the virtual ZONE_MOVABLE are taken is stored in the global variable movable_zone.

