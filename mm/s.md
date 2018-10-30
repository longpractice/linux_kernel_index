# S

## set_pgd
Must be implemented by all architectures.
Set the value of a page global directory

## set_pud
Must be implemented by all architectures.
set the value of a page upper directory

## set_pmd
Must be implemented by all architectures.
set the value of a page middle directory

## set_pte
Must be implemented by all architectures.
set ent value of a page table entry

## __set_fixmap
in x86, associate the fixmap to the physical addresses

## start_kernel
global starte routine that is executed after kernel loading to start various subsystems

## setup_arch
routine used in start_kernel(). Contains architecture specific code that is responsible for boot-time initializations.
Related to memory management are


1. 	`memblock_reserve(__pa_symbol(_text), (unsigned long)__bss_stop - (unsigned long)_text);`
   Reserve the bss area is reserved.

2. ```c
	/*
	 * Make sure page 0 is always reserved because on systems with
	 * L1TF its contents can be leaked to user processes.
	 */
	memblock_reserve(0, PAGE_SIZE);
	```
	Reserve the first page.
3. e820__memory_setup();
4. e820_add_kernel_range();
5. trim_bios_range();
6. as below, find the last pfn to max_pfn
```c
	/*
	* partially used pages are not usable - thus
	* we are rounding upwards:
	*/
	max_pfn = e820__end_of_ram_pfn();
```

5. as below 1UL << (32 - PAGE_SHIFT) shows the maximum pfn with 32 bit address space. This function clamp the max_pfn to what 32bit physical address space could donate. Note that we could not directly clamp max_pfn to 1UL << (32 - PAGE_SHIFT) in the if branch since it can have empty areas between e820__end_of_low_ram_pfn() and (1UL << (32 - PAGE_SHIFT)).
```c
   	/* How many end-of-memory variables you have, grandma! */
	/* need this before calling reserve_initrd */
	if (max_pfn > (1UL<<(32 - PAGE_SHIFT)))
		max_low_pfn = e820__end_of_low_ram_pfn();
	else
		max_low_pfn = max_pfn;
```
6. high_memory = (void *)__va(max_pfn * PAGE_SIZE - 1) + 1; set the high_memory to be the virtual address corresponding to the max_pfn.
7. e820__memblock_setup();
8. reserve_bios_regions();
9. reserve_real_mode();
10. trim_platform_memory_ranges();
11. trim_low_memory_range();
12. init_mem_mapping();
13. initmem_init(); //just set node number 0 for UMA
## setup_bootmem_allocator
arch specific routine to setup bootmem allocator, no longer used by x86 though. In x86, it only does printing.

## setup_pageset
used inside build_all_zonelists() to init the boot pageset(set up the high value, set count=0, etc...).
```c
static void setup_pageset(struct per_cpu_pageset *p, unsigned long batch)
{
	pageset_init(p);
	pageset_set_batch(p, batch);
}
```
the high of the pageset is set to 6 times the batch size here.
## setup_per_cpu_areas
routine used in start_kernel(). Contains architecture specific code. For SMP systems, it initialize per-CPU variables defined statically in the source code and of which there is a seperate copy for each CPU in the system. Variables of this kind are stored in a seperate section of the kernel binaries. The purpose of setup_per_cpu_areas is to create a copy of these data for each system CPU. For non-SMP systems, it does nothing.

## setup_per_cpu_pageset
routine used in start_kernel(), before this call, only the boot pagesets are available. Set up the pageset arrays from each struct zone. 
calling sequence:

setup_per_cpu_pageset-->
setup_zone_pageset()-->
zone_pageset_init()-->
pageset_init()  and  pageset_set_high_and_batch() 



It is called after the architecture specific routine of setup_arch()
inside which there is a calling sequence of:
paging_init()-->
free_area_init_nodes()-->
free_area_init_node()-->
free_area_init_core()-->
zone_init_internals()-->
zone_pcp_init().

It is also called after the build_all_zonelists() inside start_kernel(). The build_all_zonelists(), when called during booting, fill the pageset with boot-pageset.


## sign extension approach
The approach to use virtual address for x86-64 system to limit to only 47 effective bits.

the first 47 bits of a virtual address, that is [0, 46], can be arbitrarily set. Bits in the range [47, 63], however, always need to have the same value: Either all are 0, or all are 1. Such addresses are called canonical. They divide the total address space into three parts, a lower half, a higher half, and a forbidden region in between. 

The virtual address of the lower half is [0x0, 0x0000 7fff ffff ffff], while the subset for the top half is [0xFFFF 8000 0000 0000, 0xFFFF FFFF FFFF FFFF]. 

The lower half is used for user space. The higher space is used for kernel space.

## split_mem_range

split_mem_range() is a helper function that
takes a range (for example "starting at the 0M address line and ending at the
512M address line") and returns a list of suitable ranges for mappings, while
applying some rules like "Don't use a hugepage for the first 4M on 32-bit".  So
what comes back is an array that basically says something like "Use a 4K page
at 0, a 4K page at 4K, a 4K at 8/12/... up to "4K at (2M-4))", a 2M hugepage at
2M, 4M... 508M. Or something else suitable for the kernel config (32/64 bit,
etc, look at the #ifdefs to figure out exactly what it does for a given address
location)

## steal_suitable_fallback
This function implements actual steal behavior. If order is large enough, we can steal whole pageblock. If not, we first move freepages in this pageblock to our migratetype and determine how many already-allocated pages are there in the pageblock with a compatible migratetype. If at least half of the pages are free or compatible, we can change migratetype of the pageblock itself, so pages freed in the future will be put in the correct free list.

## __START_KERNEL_MAP
in x86-64, the kernel text is mapped into the region starting from __START_KERNEL, which is __START_KERNEL_MAP plus a compile-time configurable offset given by CONFIG_PHYSICAL_START. 

## __START_KERNEL
## System.map
each time the kernel is compiled, a file named System.map is generated and stored in the source base directory. It shows the symbols and their addresses. The information during runtime could also be read from /proc/iomem.