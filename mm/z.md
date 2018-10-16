# z

## ZONE
a notion to describe physical memory. Each node is divided up in to a number of blocks called zones which represent ranges within the memory. See ZONE_DMA, ZONE_DMA32, ZONE_MOVABLE, ZONE_HIGHMEM for references.

ZONE is described by `struct zone`.

## zone
`struct zone` in <mmzone.h>

```c

struct zone {
	/* Read-mostly fields */

	/* zone watermarks, access with *_wmark_pages(zone) macros */
	unsigned long watermark[NR_WMARK];

	unsigned long nr_reserved_highatomic;

	/*
	 * We don't know if the memory that we're going to allocate will be
	 * freeable or/and it will be released eventually, so to avoid totally
	 * wasting several GB of ram we must reserve some of the lower zone
	 * memory (otherwise we risk to run OOM on the lower zones despite
	 * there being tons of freeable ram on the higher zones).  This array is
	 * recalculated at runtime if the sysctl_lowmem_reserve_ratio sysctl
	 * changes.
	 */
	long lowmem_reserve[MAX_NR_ZONES];

#ifdef CONFIG_NUMA
	int node;
#endif
	struct pglist_data	*zone_pgdat;
	struct per_cpu_pageset __percpu *pageset;

	/* zone_start_pfn == zone_start_paddr >> PAGE_SHIFT */
	unsigned long		zone_start_pfn;

	/*
	 * spanned_pages is the total pages spanned by the zone, including
	 * holes, which is calculated as:
	 * 	spanned_pages = zone_end_pfn - zone_start_pfn;
	 *
	 * present_pages is physical pages existing within the zone, which
	 * is calculated as:
	 *	present_pages = spanned_pages - absent_pages(pages in holes);
	 *
	 * managed_pages is present pages managed by the buddy system, which
	 * is calculated as (reserved_pages includes pages allocated by the
	 * bootmem allocator):
	 *	managed_pages = present_pages - reserved_pages;
	 *
	 * So present_pages may be used by memory hotplug or memory power
	 * management logic to figure out unmanaged pages by checking
	 * (present_pages - managed_pages). And managed_pages should be used
	 * by page allocator and vm scanner to calculate all kinds of watermarks
	 * and thresholds.
	 *
	 * Locking rules:
	 *
	 * zone_start_pfn and spanned_pages are protected by span_seqlock.
	 * It is a seqlock because it has to be read outside of zone->lock,
	 * and it is done in the main allocator path.  But, it is written
	 * quite infrequently.
	 *
	 * The span_seq lock is declared along with zone->lock because it is
	 * frequently read in proximity to zone->lock.  It's good to
	 * give them a chance of being in the same cacheline.
	 *
	 * Write access to present_pages at runtime should be protected by
	 * mem_hotplug_begin/end(). Any reader who can't tolerant drift of
	 * present_pages should get_online_mems() to get a stable value.
	 *
	 * Read access to managed_pages should be safe because it's unsigned
	 * long. Write access to zone->managed_pages and totalram_pages are
	 * protected by managed_page_count_lock at runtime. Idealy only
	 * adjust_managed_page_count() should be used instead of directly
	 * touching zone->managed_pages and totalram_pages.
	 */
	unsigned long		managed_pages;
	unsigned long		spanned_pages;
	unsigned long		present_pages;

	const char		*name;

	int initialized;

	/* Write-intensive fields used from the page allocator */
	ZONE_PADDING(_pad1_)

	/* free areas of different sizes */
	struct free_area	free_area[MAX_ORDER];

	/* zone flags, see below */
	unsigned long		flags;

	/* Primarily protects free_area */
	spinlock_t		lock;

	/* Write-intensive fields used by compaction and vmstats. */
	ZONE_PADDING(_pad2_)

	/*
	 * When free pages are below this point, additional steps are taken
	 * when reading the number of free pages to avoid per-cpu counter
	 * drift allowing watermarks to be breached
	 */
	unsigned long percpu_drift_mark;
	bool			contiguous;

	ZONE_PADDING(_pad3_)
	/* Zone statistics */
	atomic_long_t		vm_stat[NR_VM_ZONE_STAT_ITEMS];
	atomic_long_t		vm_numa_stat[NR_VM_NUMA_STAT_ITEMS];
} ____cacheline_internodealigned_in_smp;

```
---
`unsigned long watermark[NR_WMARK]` is an array holding all the watermarks, see enum zone_watermarks.

---

`long lowmem_reserve[MAX_NR_ZONES]` acts as a disclaimer to all higher ranking zones(eg, ZONE_HIGHMEM is higher ranking zone than ZONE_DMA32). It tells all the higher ranking zones(each element in the lowmem_reserve[] array tells each another zone, but for not-higher ranking zones, these values are just zero) how many pages should be reserved for this zone when that higher ranking zone allocation falls back to this zone. It is set by setup_per_zone_lowmem_reserve according to sysctl_lowmem_reserve_ratio(see in page_alloc.c).
This function ensures that in lowmem_reserve[] for this zone, the disclaimed reserved memory is larger for higher ranking zones. For example, for a zone of ZONE_DMA, its lowmem_reserve[ZONE_HIGHMEM] is bigger than its lowmem_reserve[ZONE_NORMAL]. This ensures that the some memory in this zone_dma is still available for falling back from zone_normal when after zone_highmem reserves lowmem_reserve[ZONE_HIGHMEM] pages.

The purpose for this is that a lower zone short of memory sometimes is much more severe than a higher zone short of larger chunks of memory. The OOM killer may be triggered if a lower zone has shortage of memory even though there are still much memory in higher zones.

---
`struct pglist_data *zone_pgdat` points to node that holds this zone

---
`struct per_cpu_pageset __percpu *pageset`
pointer to an array(one element each cpu) of per-CPU hot page lists(there used to be cold pages, but no longer in newer kernels).
`per_cpu_pageset` holds one `per_cpu_pages`.







## zone_list
```c
struct zonelist {
	struct zoneref _zonerefs[MAX_ZONES_PER_ZONELIST + 1];
};
```

One allocation request operates on a zonelist. A zonelist is a list of zones, the first one is the 'goal' of the allocation, the other zones are fallback zones, in decreasing priority.

For UMA, there is actually only one zonelist in a node which is the fallback zonelist. But for a NUMA, there are two zonelists, one for fallback across all the nodes. Another one for only falling back within the node. It is because that we need zonelists that restrict the allocation to a single node for __GFP_THISNODE.

```c
enum {
	ZONELIST_FALLBACK,	/* zonelist with fallback */
#ifdef CONFIG_NUMA
	/*
	 * The NUMA zonelists are doubled because we need zonelists that
	 * restrict the allocations to a single node for __GFP_THISNODE.
	 */
	ZONELIST_NOFALLBACK,	/* zonelist without fallback (__GFP_THISNODE) */
#endif
	MAX_ZONELISTS
};
```

To speed the reading of the zonelist, the zonerefs contain the zone index of the entry being read. Helper functions to access information given a struct zoneref are

`zonelist_zone()` - Return the struct zone* for an entry in a zoneref


## ZONE_DMA

ZONE_DMA is used when there are devices that are not able to do DMA to all of addressable memory (ZONE_NORMAL). Then we carve out the portion of memory that is needed for these devices. 

It is configurable by `CONFIG_ZONE_DMA`, ZONE_DMA is unnecessary for a large number of machines that do not require addressing in the lower 16MB of memory because they do not use ISA devices with 16-bit address registers(plus one page byte register). Disabling ZONE_DMA prevents the VM from unnecessarily reserving a ratio of memory(defaulting to 1/256th of system capacity) with lowmem_reserve_ratio for such allocations when it will never be used.


## ZONE_DMA32
64 bits architectures needs two ZONE_DMAs because it supports devices that are only able to do DMA to the lower 16M but also 32 bit devices that can only do DMA areas below 4G.

It is not useful for 32 bits architecture.

## zonelist_node_idx
get the node index of the zone
```c
static inline int zonelist_node_idx(struct zoneref *zoneref)
{
	return zone_to_nid(zoneref->zone);
}
```

## zonelist_zone
get the struct zone* from the zoneref

```c
static inline struct zone *zonelist_zone(struct zoneref *zoneref)
{
	return zoneref->zone;
}
```

## zonelist_zone_idx
get the zone index of from the zoneref
```c
static inline int zonelist_zone_idx(struct zoneref *zoneref)
{
	return zoneref->zone_idx;
}
```


## ZONE_MOVABLE
a virtual zone used for marking range of memory that is movable. In contrast to all other zones in the system, ZONE_MOVABLE is not associated with any memory range that is of significance to the hardware. Indeed, the zone is filled with memory taken from either the hightmem or the regular zone.

## ZONE_NORMAL
Normal addressable memory is in ZONE_NORMAL. DMA operations can be performed on pages in ZONE_NORMAL if the DMA devices support transfers to all addressable memory. This is currently the only zone guaranteed to be possible present on all architectures. It is, however, not guaranteed that the zone must be equipped with memory. If, for instance, an AMD64 system has 2GiB of RAM, then all of it will belong to ZONE_DMA32, and ZONE_NORMAL will be empty.



## zone_pcp_init
set up the zone pageset with the boot pageset.

## zoneref
This struct contains information about a zone in a zonelist. It is stored here to avoid dereferences into large structures and lookups of tables.
```c
struct zoneref {
	struct zone *zone;	/* Pointer to actual zone */
	int zone_idx;		/* zone_idx(zoneref->zone) */
};
```

## zoneref_set_zone(struct zone *zone, struct zoneref *zoneref)
set a zoneref from a zone
```c
static void zoneref_set_zone(struct zone *zone, struct zoneref *zoneref)
{
	zoneref->zone = zone;
	zoneref->zone_idx = zone_idx(zone);
}
```

## zone_watermarks
<mmzone.h>
```c
enum zone_watermarks {
	WMARK_MIN,
	WMARK_LOW,
	WMARK_HIGH,
	NR_WMARK
};
```
these enums are indices inside an array of zone->watermark[NR_WMARK]. 

If more than watermark[WMAKR_HIGH] are free, the state of the zone is ideal.

If the number of free pages falls below pages_low, the kernel begins to swap pages out onto the hard disk.

If the number of free pages falls below pages_min, the pressure to reclaim pages is increased because free pages are urgently needed in the zone. 

## zone_watermark_ok
```c
int zone_watermark_ok(struct zone *z, int order, unsigned long mark, int classzone_idx, int alloc_flags)
```
function used for buddy memory allocator to see if we could allocate memory from a certain zone.

It basically checks if free pages contains at least wanted memory plus memory with lower order(they are fragmented and not fit current memory order allocation) and the lowmem_reserve.

