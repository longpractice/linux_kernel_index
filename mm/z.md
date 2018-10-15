# z

## ZONE
a notion to describe physical memory. Each node is divided up in to a number of blocks called zones which represent ranges within the memory. See ZONE_DMA, ZONE_DMA32, ZONE_MOVABLE, ZONE_HIGHMEM for references.

ZONE is described by `struct zone`.
## zone_list
```c
struct zonelist {
	struct zoneref _zonerefs[MAX_ZONES_PER_ZONELIST + 1];
};
```

One allocation request operates on a zonelist. A zonelist is a list of zones, the first one is the 'goal' of the allocation, the other zones are fallback zones, in decreasing priority.


## ZONE_DMA

ZONE_DMA is used when there are devices that are not able to do DMA to all of addressable memory (ZONE_NORMAL). Then we carve out the portion of memory that is needed for these devices. 

It is configurable by `CONFIG_ZONE_DMA`, ZONE_DMA is unnecessary for a large number of machines that do not require addressing in the lower 16MB of memory because they do not use ISA devices with 16-bit address registers(plus one page byte register). Disabling ZONE_DMA prevents the VM from unnecessarily reserving a ratio of memory(defaulting to 1/256th of system capacity) with lowmem_reserve_ratio for such allocations when it will never be used.


## ZONE_DMA32
64 bits architectures needs two ZONE_DMAs because it supports devices that are only able to do DMA to the lower 16M but also 32 bit devices that can only do DMA areas below 4G.

It is not useful for 32 bits architecture.


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

## zone_watermark_ok
```c
int zone_watermark_ok(struct zone *z, int order, unsigned long mark, int classzone_idx, int alloc_flags)
```
function used for buddy memory allocator to see if we could allocate memory from a certain zone.

It basically checks if free pages contains at least wanted memory plus memory with lower order(they are fragmented and not fit current memory order allocation) and the lowmem_reserve.

