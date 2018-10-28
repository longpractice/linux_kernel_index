# i

## init_bootmem(usigned long start, unsigned long pages)
register boot memory for UMA, `start` is pfn where the bitmap is to be placed, is the page numgber corresponding to the address of bitmap of the boot allocator.  `pages` is number of available physical pages.
return he number of bytes needed to hold the bitmap.

## init_bootmem_core(bootmem_data_t *bdata, unsigned long mapstart, unsigned long start, unsigned long end)
given a certain bootmem_data_t, the start page of bitmap of boot, the start page to put in bootmem and end page to put in bootmem. Initialize all pages inside the bootmem allocator.

`mapstart` after translating to virtual address, goes to bdata->node_bootmem_map
`start` is assigned to bdata->node_min_pfn
`end` is assigned to bdata->node_low_pfn

## init_mem_mapping
in x86, used by setup_arch(). 
the ISA range(0-1Mb) is mapped
```c
	/* the ISA range is always mapped regardless of memory holes */
	init_memory_mapping(0, ISA_END_ADDRESS);
```
then according to bottom up or not, memblocks will be mapped.
```c
	/*
	 * If the allocation is in bottom-up direction, we setup direct mapping
	 * in bottom-up, otherwise we setup direct mapping in top-down.
	 */
	if (memblock_bottom_up()) {
		unsigned long kernel_end = __pa_symbol(_end);

		/*
		 * we need two separate calls here. This is because we want to
		 * allocate page tables above the kernel. So we first map
		 * [kernel_end, end) to make memory above the kernel be mapped
		 * as soon as possible. And then use page tables allocated above
		 * the kernel to map [ISA_END_ADDRESS, kernel_end).
		 */
		memory_map_bottom_up(kernel_end, end);
		memory_map_bottom_up(ISA_END_ADDRESS, kernel_end);
	} else {
		memory_map_top_down(ISA_END_ADDRESS, end);
	}
```
the memory_map_bottom_up or memory_map_top_down will call init_range_memory_mapping to go through all the nodes. All the regions in a certain memblock-type memory will be taken to run init_memory_mapping on.

## init_memory_mapping
in x86-64, init_memory_mapping directly map the available physical memory into the virtual address space portion of the kernel starting from PAGE_OFFSET. This runs before bootmem is initialized and gets pages directly from the physical memory. To access them they are temporarily mapped. Inside, it calls kernel_physical_mapping_init for the ranges.

## initmem_init
in x86, a function called inside setup_arch() to set up the bootmem(the boot memory allocator). It will call setup_bootmem_allocator().

## __init 
```c
#define __init __attribute__ ((__section__ (".init.text"))) __cold
```
label function calls to be put in section of ".init.text", so they will be released after the init process is finished.
The information of the size is patche in kernel during link time. The variables `__init_begin` and `__init_end` records that information.

The free is done by `free_initmem`.

## __initdata
```c
#define __initdata __attribute__ ((__section__ (".init.data")))
```
label function calls to be put in section of ".init.data", so they will be released after the init process is finished.
`__init_begin` and `__init_end` records that information.

The free is done by `free_initmem`.