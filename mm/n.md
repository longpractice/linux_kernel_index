# n

## native_pagetable_init()
in x86 architecture, used by x86_init.paging.pagetable_init(); In x86-32, it is in arch/x86/mm/init_32.c. It first set up all the page tables, and then calls the static inline function called paging_init() in the same file.

paging_init() first calls **pagetable_init()** in the same file(**the naming is really confusing here**).


## NODE_DATA
must be implemented by all architectures.

After architecture specific code detects memory and establishes how it is distributed among nodes and zones, each architecture should hold one pglist_data for each node. NODE_DATA(nid) returns such node instance according to the node number.

## NR_PAGEBLOCK_BITS
the number of bits to donate the page block property. See enum `pageblock_bits`.

## node_states

in nodemask.h.

If more than one node can be present on the system, the kernel keeps a bitmap that provides state information for each node. The states are specified with a bitmask, and the following values are possible:

```c
/*
 * Bitmasks that are kept for all the nodes.
 */
enum node_states {
	N_POSSIBLE,		/* The node could become online at some point */
	N_ONLINE,		/* The node is online */
	N_NORMAL_MEMORY,	/* The node has regular memory */
#ifdef CONFIG_HIGHMEM
	N_HIGH_MEMORY,		/* The node has regular or high memory */
#else
	N_HIGH_MEMORY = N_NORMAL_MEMORY,
#endif
	N_MEMORY,		/* The node has memory(regular, high, movable) */
	N_CPU,		/* The node has one or more cpus */
	NR_NODE_STATES
};

```

note that the N_HIGH_MEMORY means that the node has regular or high memory.

N_NORMAL_MEMORY is only set if non-highmem is present on a node.

Note the for UMA, there is not bitmap for such states. The set, clear operation of the states(node_set_state and node_clear_state) does nothing. All the state of the first node will be 1.
