# a

## add_active_range()
replaced by memblock_add() in newest kernel


## alloc_bootmem(size)
reserve the memory in bootmem allocator in ZONE_NORMAL, data is aligned at the start of L1 cache

calls `__alloc_bootmem` with alignment of `SMP_CACHE_BYTES` and goal of `BOOTMEM_LOW_LIMIT`

## alloc_bootmem_bdata
try to allocate memory of bootmem,  on a certain goal address, may fail.

The process goes:
starting at goal, the page bitmap is scanned for a free area to satisfy the allocation request.

If the page found immediately follows the last allocated page held in `bootmem_data_t::hint_idx` whether the required alignment could be allocated in the last page or can at least start there.

The bits of the newly allocated pages in the block bitmap are set to 1. The number of the last page allocated is also stored in `bootmem_data_t::hint_idx` and the offset is stored in 

## alloc_bootmem_pages(size)
reserve the memory in bootmem allocator in ZONE_NORMAL, data is aligned at the start of a page.

calls `__alloc_bootmem` with alignment of `PAGE_SIZE` and goal of `BOOTMEM_LOW_LIMIT`.


## alloc_bootmem_low(size)
reserve the memory in bootmem allocator in ZONE_DMA, data is aligned at the start of L1 cache

calls `__alloc_bootmem` with alignment of `SMP_CACHE_BYTES` and goal of 0.


## alloc_bootmem_pages_low(size)
reserve the memory in bootmem allocator in ZONE_DMA, data is aligned at the start of a page.

calls `__alloc_bootmem` with alignment of `PAGE_SIZE` and goal of 0.

## _ _alloc_bootmem
calls __alloc_bootmem_nopanic which calls ___alloc_bootmem_nopanic.

## _ _ _alloc_bootmem_nopanic
calls alloc_bootmem_core on a specific goal, if it fails the, set goal to zero and then tries all the nodes.

## ARCH_POPULATES_NODE_MAP
removed in latest kernel 
it is replaced by HAVE_MEMBLOCK_NODE_MAP.

Commit 0ee332c1 
```
Now all ARCH_POPULATES_NODE_MAP archs select HAVE_MEBLOCK_NODE_MAP -
there's no user of early_node_map[] left.  Kill early_node_map[] and
replace ARCH_POPULATES_NODE_MAP with HAVE_MEMBLOCK_NODE_MAP.  Also,
relocate for_each_mem_pfn_range() and helper from mm.h to memblock.h
as page_alloc.c would no longer host an alternative implementation.

This change is ultimately one to one mapping and shouldn't cause any
observable difference; however, after the recent changes, there are
some functions which now would fit memblock.c better than page_alloc.c
and dependency on HAVE_MEMBLOCK_NODE_MAP instead of HAVE_MEMBLOCK
doesn't make much sense on some of them.  Further cleanups for
functions inside HAVE_MEMBLOCK_NODE_MAP in mm.h would be nice.
```