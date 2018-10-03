# b

## batch size
batch size is the unit of filling in per cpu page set for a certain zone, it is normally one fourth of the 1000th of total number of pages contained by the zone

## build_all_zonelists
used in start_kernel(), called after setup_arch() or used in memory hot plug.


if the system state is not in SYSTEM_BOOTING(eg, memory hotplug), by calling __build_all_zonelist(), it sets up the node and zone data structures. It determines the order that kernel allocates memory and frees memory. Generally speaking the highmem should be firstly allocated and then the normal memory and eventually the DMA part.

if the system state is in SYSTEM_BOOTING, it will first call __build_all_zonelists() and then it initializes the boot_pagesets for the per cpu pageset.

```c
/*
 * Initialize the boot_pagesets that are going to be used
 * for bootstrapping processors. The real pagesets for
 * each zone will be allocated later when the per cpu
 * allocator is available.
 *
 * boot_pagesets are used also for bootstrapping offline
 * cpus if the system is already booted because the pagesets
 * are needed to initialize allocators on a specific cpu too.
 * F.e. the percpu allocator needs the page allocator which
 * needs the percpu allocator in order to allocate its pagesets
 * (a chicken-egg dilemma).
 */
for_each_possible_cpu(cpu)
    setup_pageset(&per_cpu(boot_pageset, cpu), 0);
```

Note that later in start_kernel(), setup_per_cpu_pageset() will be called to do some final pageset initialization.

## __build_all_zonelists()
it sets up the node and zone data structures. It determines the order that kernel allocates memory and frees memory. Generally speaking the highmem should be firstly allocated and then the normal memory and eventually the DMA part.

## build_zonelists(pg_data_t *pgdat) 
for UMA
used inside __build_all_zonelists. build zonelists ordered by zone and nodes within zones. This results conserving DMA zones until all Normal memory is exhausted, but results in overflowing to remote node while memory may still exist in local DMA zone.

Say current node is ith node. It first add all the zones on the current node(ith), then add all the zones from i+1 to  MAX_NUMNODES and then adds the zones from 0 to i-1. So it is added in a round-robin manner.

## build_zonerefs_node(pg_data_t *pgdat, struct zoneref *zonerefs)
used inside build_zonelists(). fill in zonerefs[] with all zones in fallback order in pgdat, only zones with managed_pages are added(using function managed_zone()). Returns the number of zones added.