# n

## native_pagetable_init()
in x86 architecture, used by x86_init.paging.pagetable_init(); In x86-32, it is in arch/x86/mm/init_32.c. It first set up all the page tables, and then calls the static inline function called paging_init() in the same file.

paging_init() again calls **pagetable_init()** in the same file(**the naming is really confusing here**).

## NODE_DATA
must be implemented by all architectures
After architecture specific code detects memory and establishes how it is distributed among nodes and zones, each architecture should hold one pglist_data for each node. NODE_DATA(nid) returns such node instance according to the node number.