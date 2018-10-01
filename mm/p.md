# P


## PAGE_ALIGN 
a standard macro that must be defined by each architecture(typically in page.h). It expects an address as parameter and "rounds" the address so that it is exactly at the start of the next page.

## PAGE_OFFSET

the virtual address at which the kernel portion starts, since the physical memory is mapped to the virtual memory space of the kernel. For a limited range, the physical memory plus PAGE_OFFSET gives you the virtual memory of the memory for the kernel

## _PAGE_ACCESSED
a flag in superfluous bit in PTE entry. It is set automatically by the CPU each time the page is accessed. The kernel regularly checks the field to establish how actively the page is used(in frequently used pages are good swapping candidates). The bit is set after either read or write access.

## _PAGE_BIT_NX
a flag in superfluous bit in PTE entry. Only IA-32 and AMD64 provide this information. It labels that the page is not executable.

## _PAGE_DIRTY
a flag in superfluous bit in PTE entry. It indicates whether the page is "dirty", that is, whether the page contents have been modified.

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

```
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
struct pglist_data is typedef-ed as pg_data_t

## __pgd
convert an unsigned long to pgd_t

## PHYSICAL_ALIGN
a kernel configuration to set the alignment requirement for loading the kernel onto

## PHYSICAL_START
it is a kernel configuration to set the start physical address to load the kernel onto. The default value is 0x1000000 which is 16 megabytes.

This field is ignored if config RELOCATABLE is enabled. 

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






