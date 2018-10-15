# e

## e820_table

```c
/*
 * The whole array of E820 entries:
 */
struct e820_table {
	__u32 nr_entries;
	struct e820_entry entries[E820_MAX_ENTRIES];
};
```

## e820__memory_setup(void)
in x86 architecture, routine invoked inside setup_arch() which is inside start_kernel(). It creates a list with the memory regions occupied by the system and the free memory regions. It will call the e820__memory_setup_default() by default. It will also print out the memory information.

## e820__memory_setup_default(void)
int x86 architecture, the default function called by e820__memory_setup(). Pass the firmware (bootloader) E820 map to the kernel and process it. It pops the variable of e820_table.

## e820__register_active_regions
in latest kernel, replaced by e820__memblock_setup()

## _edata
the end of data section of the kernel binary. The data section starts from _etext, the end of the text section. It is also the start of data section that is no longer used after kernel initialization.

## _end
the end of the kernel executable binary in physical memory. It is also the end of the data section that is not needed after kernel initialization.

## _etext
end of the text section of contains the kernel code. It is also the start of the data section of the kernel binary loaded in memory

## expand
in buddy system, if the memory chunk to be allocated is smaller than the selected range of contiguous pages, that is, if the pages stem from a higher allocation order than required because no suitable smaller block was available, it must be split into smaller segments in accordance with the principles of the buddy system. This is done by the expand function.

```c
/*
 * The order of subdivision here is critical for the IO subsystem.
 * Please do not alter this order without good reasons and regression
 * testing. Specifically, as large blocks of memory are subdivided,
 * the order in which smaller blocks are delivered depends on the order
 * they're subdivided in this function. This is the primary factor
 * influencing the order in which pages are delivered to the IO
 * subsystem according to empirical testing, and this is also justified
 * by considering the behavior of a buddy system containing a single
 * large block of memory acted on by a series of small allocations.
 * This behavior is a critical factor in sglist merging's success.
 *
 * -- nyc
 */
static inline void expand(struct zone *zone, struct page *page,
	int low, int high, struct free_area *area,
	int migratetype)
{
	unsigned long size = 1 << high;

	while (high > low) {
		area--;
		high--;
		size >>= 1;
		VM_BUG_ON_PAGE(bad_range(zone, &page[size]), &page[size]);

		/*
		 * Mark as guard pages (or page), that will allow to
		 * merge back to allocator when buddy will be freed.
		 * Corresponding page table entries will not be touched,
		 * pages will stay not present in virtual address space
		 */
		if (set_page_guard(zone, &page[size], high, migratetype))
			continue;

		list_add(&page[size].lru, &area->free_list[migratetype]);
		area->nr_free++;
		set_page_order(&page[size], high);
	}
}
```

the function split the high order page list and put them in lower order(half given when going one order lower).