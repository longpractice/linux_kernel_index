# e

## e820_add_kernel_range
used inside x86 setup_arch(), add the kernel code and data range between `_text` and `_end` to E820_TYPE_RAM range, which includes kernel text, kernel data and initialization data.

## e820_end_pfn
Find the highest page frame number no larger than limit_pfn that is in a range of a certain type.
```c
/*
 * Find the highest page frame number we have available
 */
static unsigned long __init e820_end_pfn(unsigned long limit_pfn, enum e820_type type)
{
	int i;
	unsigned long last_pfn = 0;
	unsigned long max_arch_pfn = MAX_ARCH_PFN;

	for (i = 0; i < e820_table->nr_entries; i++) {
		struct e820_entry *entry = &e820_table->entries[i];
		unsigned long start_pfn;
		unsigned long end_pfn;

		if (entry->type != type)
			continue;

		start_pfn = entry->addr >> PAGE_SHIFT;
		end_pfn = (entry->addr + entry->size) >> PAGE_SHIFT;

		if (start_pfn >= limit_pfn)
			continue;
		if (end_pfn > limit_pfn) {
			last_pfn = limit_pfn;
			break;
		}
		if (end_pfn > last_pfn)
			last_pfn = end_pfn;
	}

	if (last_pfn > max_arch_pfn)
		last_pfn = max_arch_pfn;

	pr_info("last_pfn = %#lx max_arch_pfn = %#lx\n",
		last_pfn, max_arch_pfn);
	return last_pfn;
}
```

## e820_entry

```c
/*
 * A single E820 map entry, describing a memory range of [addr...addr+size-1],
 * of 'type' memory type:
 *
 * (We pack it because there can be thousands of them on large systems.)
 */
struct e820_entry {
	u64			addr;
	u64			size;
	enum e820_type		type;
} __attribute__((packed));

```

## e820_memblock_setup
setup memblocks according to e820_table. If the entry in the e820_table is not of type E820_TYPE_RAM or E820_TYPE_RESERVED_KERN, that entry will go into memblock type reserved. The holes between the entries will also go to the memblock reserved. Everything else will be treated as memblock type memory.


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

## e820_type
```c
/*
 * These are the E820 types known to the kernel:
 */
enum e820_type {
	E820_TYPE_RAM		= 1,
	E820_TYPE_RESERVED	= 2,
	E820_TYPE_ACPI		= 3,
	E820_TYPE_NVS		= 4,
	E820_TYPE_UNUSABLE	= 5,
	E820_TYPE_PMEM		= 7,

	/*
	 * This is a non-standardized way to represent ADR or
	 * NVDIMM regions that persist over a reboot.
	 *
	 * The kernel will ignore their special capabilities
	 * unless the CONFIG_X86_PMEM_LEGACY=y option is set.
	 *
	 * ( Note that older platforms also used 6 for the same
	 *   type of memory, but newer versions switched to 12 as
	 *   6 was assigned differently. Some time they will learn... )
	 */
	E820_TYPE_PRAM		= 12,

	/*
	 * Reserved RAM used by the kernel itself if
	 * CONFIG_INTEL_TXT=y is enabled, memory of this type
	 * will be included in the S3 integrity calculation
	 * and so should not include any memory that the BIOS
	 * might alter over the S3 transition:
	 */
	E820_TYPE_RESERVED_KERN	= 128,
};
```

## e820__memory_setup(void)
in x86 architecture, routine invoked inside setup_arch() which is inside start_kernel(). It creates a list with the memory regions occupied by the system and the free memory regions. It will call x86_init.resources.memory_setup(), which is e820__memory_setup_default() by default(see `x86_init` for details). This function will copy boot_params.e820_table to `e820_table`. It then also copy `e820_table` to `e820_table_kexec` and `e820_table_firmware`. It will also print out the memory information.

## e820__memory_setup_default(void)
int x86 architecture, the default function called by e820__memory_setup(). It calls basically append_e820_table (pops the variable `e820_table`) and e820__update_table (sanitize the table `e820_table`).

## e820__range_add
```c
void __init e820__range_add(u64 start, u64 size, enum e820_type type)
{
	__e820__range_add(e820_table, start, size, type);
}
```

this function just add the range to the e820_table global variable.


## e820__register_active_regions
in latest kernel, replaced by e820__memblock_setup()

## e820__update_table
```c
int __init e820__update_table(struct e820_table *table)
```
sanitize the e820_table. 
TODO: analyse this.

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