# z

## zone_list
```c
struct zonelist {
	struct zoneref _zonerefs[MAX_ZONES_PER_ZONELIST + 1];
};
```

One allocation request operates on a zonelist. A zonelist is a list of zones, the first one is the 'goal' of the allocation, the other zones are fallback zones, in decreasing priority.

## ZONE_MOVABLE
a virtual zone used for marking range of memory that is movable. In contrast to all other zones in the system, ZONE_MOVABLE is not associated with any memory range that is of significance to the hardware. Indeed, the zone is filled with memory taken from either the hightmem or the regular zone.

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

