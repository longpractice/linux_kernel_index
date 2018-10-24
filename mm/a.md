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

## __alloc_bootmem
calls __alloc_bootmem_nopanic which calls ___alloc_bootmem_nopanic.

## ___alloc_bootmem_nopanic
calls alloc_bootmem_core on a specific goal, if it fails the, set goal to zero and then tries all the nodes.

## __alloc_pages_nodemask

allocate pages for buddy zoned buddy allocator. This is the 'heart' of the zoned buddy allocator.

It first tries to call the get_page_from_freelist. If not succeeded, it will go into a slower method, __alloc_pages_slowpath, which contains much more details.

## __alloc_pages_slowpath
If the first attemp of memory allocation of get_page_from_freelist in __alloc_pages_nodemask fails, we will start to go into this routine. The blow are the goto marks discussed seperately.

retry_cpuset



## ALLOC_NO_WATERMARKS
allocation flag for buddy system, if this flag is set, the allocation does not check the watermark at all

## ALLOC_WMARK_MIN
allocation flag for buddy system, if this flag is set, only allocate page if the zone still contains at least zone->pages_min pages.

## ALLOC_WMARK_HIGH
default allocation flag for buddy system, if this flag is set, only allocate page if the zone still contains at least zone->pages_high pages.

## ALLOC_WMARK_LOW
allocation flag for buddy system, if this flag is set, only allocate page if the zone still contains at least zone->pages_low pages.

## append_e820_table
called inside e820_memory_setup_default(), it basically copy entries to e820_table variable.

```c
/*
 * Copy the BIOS E820 map into a safe place.
 *
 * Sanity-check it while we're at it..
 *
 * If we're lucky and live on a modern system, the setup code
 * will have given us a memory map that we can use to properly
 * set up memory.  If we aren't, we'll fake a memory map.
 */
static int __init append_e820_table(struct boot_e820_entry *entries, u32 nr_entries)
{
	/* Only one memory region (or negative)? Ignore it */
	if (nr_entries < 2)
		return -1;

	return __append_e820_table(entries, nr_entries);
}
```

## __append_e820_table
called inside append_e820_table(), it basically copy entries to e820_table variable.

```c
static int __init __append_e820_table(struct boot_e820_entry *entries, u32 nr_entries)
{
	struct boot_e820_entry *entry = entries;

	while (nr_entries) {
		u64 start = entry->addr;
		u64 size = entry->size;
		u64 end = start + size - 1;
		u32 type = entry->type;

		/* Ignore the entry on 64-bit overflow: */
		if (start > end && likely(size))
			return -1;

		e820__range_add(start, size, type);

		entry++;
		nr_entries--;
	}
	return 0;
}
```


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