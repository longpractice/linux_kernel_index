# v

## __va()
macro used by kernel to translate physical memory address(linear-shift-mappable) to virtual address using linear shifting(by PAGE_OFFSET)

## vmalloc
in x86, a virtual memory area in kernel virtual memory space. 

In x86-32, Virtually contiguous memory areas that are not contiguous in physical memory can be reserved in the vmalloc area. While this mechanism is commonly used with user processes, the kernel itself tries to avoid non-contiguous physical addresses as best it can. It usually succeeds because mosf of the large memory blocks are allocated for the kernel at boot time when RAM is not yet fragmented. However, on systems that ahve been running for longer periods, situations can arise in which the kernel requires physical memory but the space available is not contiguous. A primary example of such a situation is when modules are loaded dynamically.

In x86-64, the area is only useful with sparce memory, the page tables in this area are set up such that all struct page instances located in the physical memory are mapped into the area without any holes. This provides a virtually contiguous area in which only the active memory regions are included. The MMU therefore automatically aids the translation between virtual and physical number that does not needed to concernted with holes anymore. This acceration the operation considerably.

## VMALLOC_END
the end of VMALLOC area of X86. In x86-32:
```c
#ifdef CONFIG_HIGHMEM
# define VMALLOC_END	(PKMAP_BASE - 2 * PAGE_SIZE)
#else
# define VMALLOC_END	(LDT_BASE_ADDR - 2 * PAGE_SIZE)
#endif
```

in x86-64, it is defined as 0xffff e1ff ffff ffff.



## VMALLOC_OFFSET
macro in x86 defined as 8 * 1024 * 1024.

It is just any arbitrary offset to the start of the vmalloc VM area: the current 8MB value just means that there will be a 8MB "hole" after the physical memory until the kernel virtual memory starts. That means that
any out-of-bounds memory accesses will hopefully be caught. The vmalloc() routines leaves a hole of 4kB between each vmalloced area for the same reason. 

## __VMALLOC_RESERVE
macro defining the size of vmalloc area

## VMALLOC_START
in x86, the start of the virtual address of VMALLOC area. 

In x86-32 it is defined as ` high_memory + VMALLOC_OFFSET `

in x86-64, it is defined as 0xffff c200 0000 0000