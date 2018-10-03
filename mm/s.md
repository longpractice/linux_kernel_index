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
Related to memory management is that it initialize the boot allocator.

## setup_pageset
used inside build_all_zonelists() to set up the boot pageset.
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

## System.map
each time the kernel is compiled, a file named System.map is generated and stored in the source base directory. It shows the symbols and their addresses. The information during runtime could also be read from /proc/iomem.