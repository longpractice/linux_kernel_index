# k

## kernel_physical_mapping_init()
routine used by init_memory_mapping, defined in arch/x86/mm/init_64.c and init32_c. 
For init_64.c:
Create page table(only reach pte level, not struct page inited yet) mapping for the physical memory for specific physical addresses. The virtual and physical address have to be aligned on PMD level down. It returns the last physical address mapped.



## KERNEL_TEXT_SIZE 
in x86-64, donates the kernel text size.

## KERNEL_TEXT_START
in x86-64, donates the kernel text region start virtual address.

## kmap_get_fixmap_pte
from a virtual address, construct a pte
```c
static inline pte_t *kmap_get_fixmap_pte(unsigned long vaddr)
{
	pgd_t *pgd = pgd_offset_k(vaddr);
	p4d_t *p4d = p4d_offset(pgd, vaddr);
	pud_t *pud = pud_offset(p4d, vaddr);
	pmd_t *pmd = pmd_offset(pud, vaddr);
	return pte_offset_kernel(pmd, vaddr);
}
```

## kmap_init
kmap_init initializes the global variable kmap_pte and kmap_vstart. The kernel uses this variable to store the page table entry for the area later used to map pages from the highmem zone into kernel address space. 

```c
static void __init kmap_init(void)
{
	unsigned long kmap_vstart;

	/*
	 * Cache the first kmap pte:
	 */
	kmap_vstart = __fix_to_virt(FIX_KMAP_BEGIN);
	kmap_pte = kmap_get_fixmap_pte(kmap_vstart);
}
```
## kmap_pte
in x86. The first pte of fixmap area. It is initialized in kmap_init() as
```
kmap_pte = kmap_get_fixmap_pte(kmap_vstart);
```

## kmap_vstart
in x86. The address of the first fixmap area for highmem kernel mappings. It is initialized in kmap_init() as 
```
kmap_vstart = __fix_to_virt(FIX_KMAP_BEGIN);
```