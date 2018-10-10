# z

## zone_list
```c
struct zonelist {
	struct zoneref _zonerefs[MAX_ZONES_PER_ZONELIST + 1];
};
```

One allocation request operates on a zonelist. A zonelist is a list of zones, the first one is the 'goal' of the allocation, the other zones are fallback zones, in decreasing priority.

## ZONE_MOVABLE
a virtual zone used for marking range of memory that is movable.

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