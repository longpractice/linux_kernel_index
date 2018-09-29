# P

## PAGE_ALIGN 
a standard macro that must be defined by each architecture(typically in page.h). It expects an address as parameter and "rounds" the address so that it is exactly at the start of the next page.

## PAGE_OFFSET

the virtual address at which the kernel portion starts, since the physical memory is mapped to the virtual memory space of the kernel. For a limited range, the physical memory plus PAGE_OFFSET gives you the virtual memory of the memory for the kernel

## _PAGE_PRESENT
a flag in superfluous bit in PTE entry that specifies whether the virtual page is present in RAM memory. This need not necessary be the case because pages may be swapped out into a swap area.

## pgd_bad
check whether entries of the global page directory is valid. The semantics is fuzzy across different architectures. They are used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pgd_clear
delete the passed page table entry. This is usually done by setting it to zero

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

## __pgd
convert an unsigned long to pgd_t

## pmd_bad
check whether the page middle directory entry is valid. The semantics is somewhat fuzzy across different architecture. Used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pmd_clear
delete the passed page table entry. This is usually done by setting it to zero

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

## pte_clear
delete the passed page table entry. This is usually done by setting it to zero

## pte_page
return the address of the page holding the page entry

## pte_present
check whether the PAGE_VALID bit of the entry is set

## pte_t
?? direct entries of page tables

## pte_val
convert a variable of pte_t to unsigned long

## __pte
convert an unsigned long to pte_t

## pud_bad
checks whether the page upper directory is valid. The semantics is fuzzy across different archs. They are used for safety purposes in functions that receive input parameters from the outside where it cannot be assumed that the parameters are valid.

## pud_clear
delete the passed page table entry. This is usually done by setting it to zero

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






