# v

## __va()
macro used by kernel to translate physical memory address(linear-shift-mappable) to virtual address using linear shifting(by PAGE_OFFSET)

## vmalloc
a virtual memory area in kernel virtual memory space. Virtually contiguous memory areas that are not contiguous in physical memory can be reserved in the vmalloc area. While this mechanism is commonly used with user processes, the kernel itself tries to avoid non-contiguous physical addresses as best it can. It usually succeeds because mosf of the large memory blocks are allocated for the kernel at boot time when RAM is not yet fragmented. However, on systems that ahve been running for longer periods, situations can arise in which the kernel requires physical memory but the space available is not contiguous. A primary example of such a situation is when modules are loaded dynamically.

## VMALLOC_END
the end of VMALLOC area of X86. In x86-32:
```
#ifdef CONFIG_HIGHMEM
# define VMALLOC_END	(PKMAP_BASE - 2 * PAGE_SIZE)
#else
# define VMALLOC_END	(LDT_BASE_ADDR - 2 * PAGE_SIZE)
#endif
```

## VMALLOC_OFFSET
macro in x86 defined as 8 * 1024 * 1024.

It is just any arbitrary offset to the start of the vmalloc VM area: the current 8MB value just means that there will be a 8MB "hole" after the physical memory until the kernel virtual memory starts. That means that
any out-of-bounds memory accesses will hopefully be caught. The vmalloc() routines leaves a hole of 4kB between each vmalloced area for the same reason. 

## __VMALLOC_RESERVE
macro defining the size of vmalloc area

## VMALLOC_START
in x86, the start of the virtual address of VMALLOC area. In x86-32 it is defined as ` high_memory + VMALLOC_OFFSET `
