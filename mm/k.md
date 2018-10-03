# k

## kmap_init
kmap_init initializes the global variable kmap_pte. The kernel uses this variable to store the page table entry for the area later used to map pages from the highmem zone into kernel address space. 

## kmap_vstart
in x86. The address of the first fixmap area for highmem kernel mappings