# b

## batch size
batch size is the unit of filling in per cpu page set for a certain zone, it is normally one fourth of the 1000th of total number of pages contained by the zone


## bootmem_data, bootmem_data_t
per node information used by the bootmem allocator
```c
/**
 * struct bootmem_data - per-node information used by the bootmem allocator
 * @node_min_pfn: the starting physical address of the node's memory
 * @node_low_pfn: the end physical address of the directly addressable memory
 * @node_bootmem_map: is a bitmap pointer - the bits represent all physical
 *		      memory pages (including holes) on the node.
 * @last_end_off: the offset within the page of the end of the last allocation;
 *                if 0, the page used is full
 * @hint_idx: the PFN of the page used with the last allocation;
 *            together with using this with the @last_end_offset field,
 *            a test can be made to see if allocations can be merged
 *            with the page used for the last allocation rather than
 *            using up a full new page.
 * @list: list entry in the linked list ordered by the memory addresses
 */
typedef struct bootmem_data {
	unsigned long node_min_pfn;
	unsigned long node_low_pfn;
	void *node_bootmem_map;
	unsigned long last_end_off;
	unsigned long hint_idx;
	struct list_head list;
} bootmem_data_t;
```

`node_min_pfn` is the start physical address of the node's memory, note that in early kernel, it used to be called node_boot_start.

`node_low_pfn` is the end physical address of the directly addressable memory, it is the end of ZONE_NORMAL, bootmem allocation happens only in ZONE_DMA and ZONE_NORMAL

`node_bootmem_map` is a pointer to virtual memory of the memory area in which the allocation bitmap is stored. On IA-32 systems, the memory area immediately following the kernel image is used for this purpose. The corresponding address is held in the _end variable, which is automatically patched into the kernel image during linking.

`last_end_off` the offset within the page of the end of the last allocation; if 0, the page used is full. It used to be call `last_pos` in older kernels.

`hint_idx` the PFN of the page used with the last allocation; together with using this with the `last_end_offset` field, a test can be made to see if allocations can be merged with the page used for the last allocation rather than using up a full new page.

`list` system with discontinuous memory can require more than one bootmem allocator. This is typically the case on NUMA machines that register one bootmem allocator per node, but it would, for instance, also be possible to register one bootmem allocator for reach continuous memory region on systems where the physical address space is interspersed with holes.



## bootmem_node_data
a global array holding all the `bootmem_data_t` for all the nodes. For UMA, there is only one element in this array, and is assigned to contig_page_data.bdata





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