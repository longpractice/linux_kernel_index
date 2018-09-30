# b

## build_all_zonelists
used in start_kernel(), it sets up the node and zone data structures. It determines the order that kernel allocates memory and frees memory. Generally speaking the highmem should be firstly allocated and then the normal memory and eventually the DMA part.

## build_zonelists(pg_data_t *pgdat) 
for UMA
used inside build_all_zonelists. build zonelists ordered by zone and nodes within zones. This results conserving DMA zones until all Normal memory is exhausted, but results in overflowing to remote node while memory may still exist in local DMA zone.

Say current node is ith node. It first add all the zones on the current node(ith), then add all the zones from i+1 to  MAX_NUMNODES and then adds the zones from 0 to i-1. So it is added in a round-robin manner.

## build_zonerefs_node(pg_data_t *pgdat, struct zoneref *zonerefs)
used inside build_zonelists(). fill in zonerefs[] with all zones in fallback order in pgdat, only zones with managed_pages are added(using function managed_zone()). Returns the number of zones added.