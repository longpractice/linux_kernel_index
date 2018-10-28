## r

## real mode
physical address mode in x86. When in real mode, processors do not support virtual addressing, and Linux, which is designed and implemented for systems with protected mode, requires virtual addressing to enable process protection and isolation, a crucial abstraction provided by the kernel.

## real mode kernel
With the kernel image in memory and control given from the stage 2 boot loader, the kernel stage begins. The kernel iamge isn't so much an executable kernel, but a compressed kernel image. Typically this is a zImage (compressed image, less than 512KB) or a bzImage(big compressed image, greater than 512KB), that has been previously compressed with zlib. At the head of the this kernel image is a routine that does some minimal amount of hardware setup and then decompresses the kernel contained within the kernel image and places it into memory. If an initial RAM disk image is present, this routine moves it into memory and notes it for later use. The routine then calls the kernel and the kernel boot begins. 

When the bzImage is invoked, you begin at arch/x86/boot/header.S _start routine.
It then jumps to start_of_setup label in header.S and then eventually jump to arch/x86/boot/main.c :: main(). main() in arch/x86/boot/main.c eventually calls go_to_protected_mode() to jump to protected mode.


## reserve_bios_regions
reserve bios region which is guaranteed to start between BIOS_START_MIN and BIOS_START_MAX(128k and 640k) and end to 1mb. It also covers the range of EBDA(Extended BIOS Data Area).

## reserve_bootmem
arch-irrelavant code to mark none-free pages in boot allocator, arch-specific code will call this code to mark according to its only need. No longer used by x86 though.

## reserve_real_mode
in x86, reserve memory in memblock_type reserved. It must be from memory under 1M. It is used for trampoline. 

## reset vector
In most architectures, on reset, processor is initialized in normal or physical address mode (also called real mode in x86) and begins executing the platform's firmware instructions found at the reset vector.

## required_kernelcore
the administrator could specify this value for the non-movable memory. The non-movable memory is spread evenly across the different nodes. 

## required_movablecore
the administrator could specify the value for the movable memory. Note that the non-movable memory is spread evenly across the different nodes and the movable memory is only from highest zone(in 32bit systems, ZONE_HIGHMEM, for 64bit systems, ZONE_NORMAL or ZONE_DMA32).

## rmqueue
allocate a page from the given zone. Use pcplists for order-0 allocations. It used by get_page_from_freelist function to remove the page from freelist once the kernel has found a suitable zone with sufficient free pages for the allocation.

## __rmqueue
do the hard work of removing an element from the buddy allocator. Call me with the zone->lock already held.

The kernel uses the __rmqueue function which acts as a gatekeeper to penetrate into the innermost core of the buddy system.

## __rmqueue_cma_fallback
not useful unless CONFIG_CMA is set. If CONFIG_CMA is not set, returns null.

## __rmqueue_fallback
used inside __rmqueue, try finding a free buddy page on the fallback list and put it on the free list of requested migratetype, possibly along with other pages from the same block, depending on the fragmentation avoidance heuristics. Returns true if fallback was found so that __rmqueue_smallest() can grab it.

The use of signed ints for order and current_order is a deliberate deviation from the rest of the file, to make the for loop condition simpler.

## __rmqueue_smallest
used inside __rmqueue, by reference to the desired allocation order, the zone from which the pages are to be removed, and the migrate type, __rmqueue_smallest scans the page list until it finds a suitable contiguous chunk of memory. If such a finding failed, the other migrate lists are tried as an emergency measure in __rmqueue_cma_fallback if migratetype == MIGRATE_MOVABLE and __rmqueue_fallback otherwise.

```c
/*
 * Go through the free lists for the given migratetype and remove
 * the smallest available page from the freelists
 */
static __always_inline
struct page *__rmqueue_smallest(struct zone *zone, unsigned int order,
						int migratetype)
{
	unsigned int current_order;
	struct free_area *area;
	struct page *page;

	/* Find a page of the appropriate size in the preferred list */
	for (current_order = order; current_order < MAX_ORDER; ++current_order) {
		area = &(zone->free_area[current_order]);
		page = list_first_entry_or_null(&area->free_list[migratetype],
							struct page, lru);
		if (!page)
			continue;
		list_del(&page->lru);
		rmv_page_order(page);
		area->nr_free--;
		expand(zone, page, order, current_order, area, migratetype);
		set_pcppage_migratetype(page, migratetype);
		return page;
	}

	return NULL;
}

```

The function is simple. It scans the required order or above to find smallest contiguous memory in current migrate type.

## rmv_page_order
a helper function that deletes the PG_buddy bit from the page flags -- the page is not contained in the buddy system anymore -- and sets the private element of struct page to 0. The private data should not be contained any more.