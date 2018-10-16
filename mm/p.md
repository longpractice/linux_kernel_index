# P

## __pa()
macro used by the kernel to convert linear-shifted-mapped virtual address to phycical memory address

## PAGE_ALIGN 
a standard macro that must be defined by each architecture(typically in page.h). It expects an address as parameter and "rounds" the address so that it is exactly at the start of the next page.


## pageblock_bits
for a block of pages(that is, pageblock_nr_pages pages) in the freelist, some bits are used to set the migration-type or whether-skipping, the enum here are the indices of the bits.


```c
/* Bit indices that affect a whole block of pages */
enum pageblock_bits {
	PB_migrate,
	PB_migrate_end = PB_migrate + 3 - 1,
			/* 3 bits required for migrate types */
	PB_migrate_skip,/* If set the block is skipped by compaction */

	/*
	 * Assume the bits will always align on a word. If this assumption
	 * changes then get/set pageblock needs updating.
	 */
	NR_PAGEBLOCK_BITS
};
```

## pageblock_flags
field in `struct zone`, it is an array of unsigned long. It has enough space to hold bit property of all page blocks. Each block needs NR_PAGEBLOCKBITS bits which is 4 bits. So one element of pageblock_flags in 64 bit machine could hold flags for 16 page blocks. 

## PF_MEMALLOC
Per process flag, this flag is used within the kernel to indicate that a thread is currently executing a memory allocation. Therefore it is allowed to recursively allocate any memory it requires ignoring watermarks and without being forced write out dirty pages.

This is to ensure that, if the code, who is attempting to free pages to satisfy an original allocation request, has to allocate a small amount of memory to proceed, that code won't then recursively try to free pages.

## pfn_to_bitidx
```c
static inline int pfn_to_bitidx(struct page *page, unsigned long pfn)
{
#ifdef CONFIG_SPARSEMEM
	pfn &= (PAGES_PER_SECTION-1);
	return (pfn >> pageblock_order) * NR_PAGEBLOCK_BITS;
#else
	pfn = pfn - round_down(page_zone(page)->zone_start_pfn, pageblock_nr_pages);
	return (pfn >> pageblock_order) * NR_PAGEBLOCK_BITS;
#endif /* CONFIG_SPARSEMEM */
}
```
given a page, get the bit index of page block flags in the zone::pageblock_flags array. The logic is clear. It first get the page frame number relative to the start of the first page of the zone. Then it gets the page block number it belongs to. Since each page block occupy `NR_PAGEBLOCK_BITS`, `(pfn >> pageblock_order) * NR_PAGEBLOCK_BITS` donates the start bit index in zone::pageblock_flags(an array of unsigned longs).



## PAGE_OFFSET
the virtual address at which the kernel portion starts, since the physical memory is mapped to the virtual memory space of the kernel. For a limited range, the physical memory plus PAGE_OFFSET gives you the virtual memory of the memory for the kernel. 
In x86-32, normally in 3:1 user-space/kernel-space division for 4GB virtual mem space, it is 0xC0000000. You could set it differently, say 0x80000000 when it requires a large amount of memory for the kernel but little for the user processes, for example, network routers.

In x86-64, this value is 0xffff 8100 0000 0000 which is 0x100 0000 0000 more than the available kernel area of 0xffff 80000 0000 0000. This additional hole before is to prevent incidental access on the non-canonical portion of the address space that would result in a general protection exception raised by the processor.




## PAGE_SHIFT
page size in terms of power of 2. So a 4k page size gives a PAGE_SHIFT of 12

## _PAGE_ACCESSED
a flag in superfluous bit in PTE entry. It is set automatically by the CPU each time the page is accessed. The kernel regularly checks the field to establish how actively the page is used(in frequently used pages are good swapping candidates). The bit is set after either read or write access.

## _PAGE_BIT_NX
a flag in superfluous bit in PTE entry. Only IA-32 and AMD64 provide this information. It labels that the page is not executable.

## _PAGE_DIRTY
a flag in superfluous bit in PTE entry. It indicates whether the page is "dirty", that is, whether the page contents have been modified.

## _PAGE_GLOBAL
a flag in superfluous bit in PTE entry for x86. The TLB entries of pages with a set _PAGE_GLOBAL bit are not flushed from the TLBs during context switches. If possible, kernel pages are provided with this page. Since the kernel is always present at the same location in the virtual address space, this enhances system performce, a welcome effect as kernel data must be made available as quickly as possible.

## page_group_by_mobility_disabled
the freelist of a zone for buddy system is normally seperated in different mobility to relieve fragmentation problem.
However, if the total memory is too small, smaller than HUGETLB_PAGE_ORDER or MAX_ORDER-1, `build_all_zonelists` will disable `page_group_by_mobility`.



## _PAGE_PRESENT
a flag in superfluous bit in PTE entry that specifies whether the virtual page is present in RAM memory. This need not necessary be the case because pages may be swapped out into a swap area.

## _PAGE_READ
a flag in superfluous bit in PTE entry. It specifies whether normal user process is allowed to read the page.

## _PAGE_RW
a flag in superfluous bit in PTE entry. It is used for architectures that lacks a finer grain of controlling _PAGE_READ and _PAGE_WRITE. It specifies whether the page is allowed to read or write from a normal user process.

## _PAGE_USER
a flag in superfluous bit in PTE entry. If _PAGE_USER is set, userspace code is allowed to access the page. Otherwise, only the kernel is allowed to do this.

## _PAGE_WRITE
a flag in superfluous bit in PTE entry. Specify whether a normal user process is allowed to write to the page

## _PAGE_EXECUTE
a flag in superfluous bit in PTE entry. Specify whether a normal user process is allowed to execute the machine code in the page

## paging_init()
in x86, called by setup_arch(). For x86-32, paging_init() is responsible for setting up the page tables that can be used only by the kernel and are not accessible in user space. 

In both x86-64 and x86-32, the calling sequence is
paging_init()-->
zone_sizes_init()-->
free_area_init_nodes()-->
free_area_init_node()-->
free_area_init_core()-->
zone_init_internals()-->
zone_pcp_init()
 

## pagetable_init()
in x86-32, called by paging_init() 

## parse_early_param()
routine used in setup_arch() in start_kernel(). It concentrates on arguments like mem=XXX, highmen=XXX, or memmap=XXX arguments. The administrator can overwrite the size of available memory or manualy define memory areas if the kernel calculates an incorrect value or is proved with a wrong value by the BIOS. This option is only of relevance on older computers. highmem= permits overwriting of the highmen size value detected. It can be used on machines with a very large RAM configuration to limit available RAM size - as it sometimes yields performance gains.



## per_cpu_pages
contained by per_cpu_pageset which forms an array in struct zone.

```c
struct per_cpu_pages {
	int count;		/* number of pages in the list */
	int high;		/* high watermark, emptying needed */
	int batch;		/* chunk size for buddy add/remove */

	/* Lists of pages, one per migrate type stored on the pcp-lists */
	struct list_head lists[MIGRATE_PCPTYPES];
};
```

note that MIGRATE_PCPTYPES just marks the number of types on the pcp lists.
So it contains MIGRATE_UNMOVABLE, MIGRATE_MOVABLE and MIGRATE_RECLAIMABLE.

## per_cpu_pageset
hold the hot pages for one cpu. there used to be two element for hot-n-cold pages. 

But this is no longer true.
https://lwn.net/Articles/173215/

Having a hot and a cold pcp list means that:

- cold pages are overlooked when a hot page is needed but none available.
- when the hot list spills, it doesn't fill the cold list if it is low.

Use a single pcp list to solve both these problems. Disallow cold page
allocation from taking hot pages though.


## persistent mapping 



## __pgprot
this data type holds addtional bits in an PTE entry

## pgd_alloc
reserve and initialize memory to hold a complete page global directory table

## pgd_bad
check whether entries of the global page directory is valid. The semantics is fuzzy across different architectures. They are used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pgd_clear
delete the passed page table entry. This is usually done by setting it to zero

## pgd_free
Must be implemented by all architectures.
free the memory occupied by the page global directory

## pgd_present
check whether the PAGE_VALID bit of the corresponding entry is set. This is the case when the page or page table addressed is in RAM memory

## pgd_t

in include/asm-generic/page.h, defined as

```c
typedef struct {
	unsigned long pgd;
} pgd_t;
```

it is the global directory of the four level page tables, it is the highest level.

## pgd_index
`pgd_index(address)` finds an global directory entry according to an virtual address

## pgd_val
convert a variable of type pte_t to an unsigned long number

## pgd_present
check whether the PAGE_VALID bit of the entry is set

## pg_data_t

typedef of struct pglist_data

## pglist_data

a struct type holding memory statistics and page replacement data of a zone.
struct pglist_data is typedef-ed as pg_data_t. It is called a pglist since it holds a pointer to an array of all struct page's with in all the zones of the node. That is the member of pglist_data->node_mem_map.

```c
typedef struct pglist_data {
	struct zone node_zones[MAX_NR_ZONES];
	struct zonelist node_zonelists[MAX_ZONELISTS];
	int nr_zones;
	struct page *node_mem_map;
	struct bootmem_data *bdata;

	unsigned long node_start_pfn;
	unsigned long node_present_pages; /* total number of physical pages */
	unsigned long node_spanned_pages; /* total size of physical page range, including holes */
	int node_id;
	wait_queue_head_t kswapd_wait;
	wait_queue_head_t pfmemalloc_wait;
	struct task_struct *kswapd;	/* Protected by
					   mem_hotplug_begin/end() */
	int kswapd_order;
	enum zone_type kswapd_classzone_idx;

	int kswapd_failures;		/* Number of 'reclaimed == 0' runs */
	/*
	 * This is a per-node reserve of pages that are not available
	 * to userspace allocations.
	 */
	unsigned long		totalreserve_pages;

	/* Write-intensive fields used by page reclaim */
	ZONE_PADDING(_pad1_)
	spinlock_t		lru_lock;

	/* Fields commonly accessed by the page reclaim scanner */
	struct lruvec		lruvec;
	unsigned long		flags;
	ZONE_PADDING(_pad2_)

	/* Per-node vmstats */
	struct per_cpu_nodestat __percpu *per_cpu_nodestats;
	atomic_long_t		vm_stat[NR_VM_NODE_STAT_ITEMS];
} pg_data_t;
```
`node_zones` is an array that holds the data structures of the zones in the node.


`node_zonelists` specifies alternative nodes and their zones in the order in which they are used for memory allocation if no more space is available in the current zone. For UMA, there is only one fallback list, and for NUMA, there are two fallback list(one for falling back across all the nodes), one for falling back within the current node.


`nr_zones` hold the number of zones.

`node_mem_map` is a pointer to an array of page instances used to describe all physical pages of the node. It includes the pages of all zones in the node.

During system boot, the kernel needs memory even before memory management has been initialized (memory must also be reserved to initialize memory management). To resolve this problem, the kernel uses the boot memory allocator described by bootmem_data.

`node_start_pfn` is the logical number of the first page frame of the NUMA node. The page frames of all nodes in the system are numbered consecutively, and each frame is given a number that is globally unique(not just unique to the node).

node_start_pfn is always 0 in a UMA system because there is only one node whose first page frame is therefore 0.

`node_present_pages` is the total number of physical pages. Note that this may not be the total number of pages that the node spans since there might be holes. When taking holes into account, see node_spanned_pages.

`node_spanned_pages` is total number of page spanned by the node, this takes the holes (if any) into account and must be no smaller than `node_present_pages`.

`node_id` is a global node identifier. All the nodes in the kernel are numbered consecutively starting from zero.

`kswapd_wait` wait queue of kswapd kernel thread

`kswapd` pointer to task structure of kswapd kernel thread

`kswapd_order` ??

`totalreserve_pages` this is a per-node reserve of pages that are not available to userspace allocations.



## __pgd
convert an unsigned long to pgd_t

## PHYSICAL_ALIGN
a kernel configuration to set the alignment requirement for loading the kernel onto

## PHYSICAL_START
it is a kernel configuration to set the start physical address to load the kernel onto. The default value is 0x1000000 which is 16 megabytes.

This field is ignored if config RELOCATABLE is enabled. 

## PKMAP_BASE
in x86, defines the virtual address start of the permanent kernel mapping.
```
#define PKMAP_BASE ((LDT_BASE_ADDR - PAGE_SIZE) & PMD_MASK)
```

## pmd_alloc
Must be implemented by all architectures.
reserve and initialize memory to hold a complete page middle directory table

## pmd_bad
check whether the page middle directory entry is valid. The semantics is somewhat fuzzy across different architecture. Used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pmd_clear
delete the passed page table entry. This is usually done by setting it to zero

## pmd_free
Must be implemented by all architectures.
free the memory occupied by the page middle directory

## pmd_offset
given an global page directory and a virtual address, return a page middle entry containing the address 

## pmd_page
return the address of the page holding the page middle directory

## pmd_t 
??used to be for entries for the page middle directory??

## pmd_val 
convert a variable of type pmd_t to an unsigned long number

## __pmd
convert an unsigned long to pmd_t

## prep_new_page
prep_new_page has to prepare the pages for life in the kernel (note that the function returns a positive value if something is wrong with the selected pages; in this case, the allocation is restarted from the beginning).

prep_new_page performs several checks on the pages to ensure that they leave the allocator in a perfect state -- this means, in particular, that the page must not be in use in existing mappings and no incorrect flags like PG_locked or PG_buddy may be set because this would imply that the page is in use somewhere else and should not be on the free list. Normally, however, no error should occur because this would imply a kernel error elsewhere. The function also sets the following default flags used for each new page. 


## pte_alloc 
Must be implemented by all architectures.
allocate and initialize memory to hold a whole page table

## pte_clear
delete the passed page table entry. This is usually done by setting it to zero

## pte_free
Must be implemented by all architectures.
free the memory occupied by a page table entry

## pte_modify
function provided all architecture to modify the flags of a page entry

## pte_page
Must be implemented by all architectures.
return the address of the page holding the page entry

## pte_present
check whether the PAGE_VALID bit of the entry is set. It means whether the page is present or swapped out.

## pte_read
function that checks whether the page table entry is readable from user space

## pte_t
?? direct entries of page tables

## pte_write
function that checks whether the page table entry is writable from user space

## pte_val
convert a variable of pte_t to unsigned long

## __pte
convert an unsigned long to pte_t

## pud_alloc
reserve and initialize memory to hold a complete page up directory table

## pud_bad
checks whether the page upper directory is valid. The semantics is fuzzy across different archs. They are used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pud_clear
delete the passed page table entry. This is usually done by setting it to zero

## pud_free
Must be implemented by all architectures.
free the memory occupied by a page upper directory

## pud_page
return the address of page that holds the current page upper directory

## pud_present
check whether the PAGE_VALID bit of the entry is set

## pud_t
??used to be for entries of the page upper directory, not sure now??

## pud_val
convert a variable of type pud_t to unsigned long

## __pud
convert a variable of type unsigned long to pud_t






