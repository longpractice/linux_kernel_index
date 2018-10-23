# g

## get_page_from_freelist

```c
static struct page *
get_page_from_freelist(gfp_t gfp_mask, unsigned int order, int alloc_flags, const struct alloc_context *ac)
```

function used by the buddy system. It refers to the flags set and the allocation order to decide whether allocation can be made; if so, it initiates actual page allocation.

## get_pageblock_bitmap
in page_alloc.c.
```c
/* Return a pointer to the bitmap storing bits affecting a block of pages */
static inline unsigned long *get_pageblock_bitmap(struct page *page,
							unsigned long pfn)
{
#ifdef CONFIG_SPARSEMEM
	return __pfn_to_section(pfn)->pageblock_flags;
#else
	return page_zone(page)->pageblock_flags;
#endif /* CONFIG_SPARSEMEM */
}
```

Ignoring sparsemem, we are basically returning the pageblock_flags of zone of containing a certain page.

## __get_pfnblock_flags_mask
```c
/**
 * get_pfnblock_flags_mask - Return the requested group of flags for the pageblock_nr_pages block of pages
 * @page: The page within the block of interest
 * @pfn: The target page frame number
 * @end_bitidx: The last bit of interest to retrieve
 * @mask: mask of bits that the caller is interested in
 *
 * Return: pageblock_bits flags
 */
static __always_inline unsigned long __get_pfnblock_flags_mask(struct page *page,
					unsigned long pfn,
					unsigned long end_bitidx,
					unsigned long mask)
{
	unsigned long *bitmap;
	unsigned long bitidx, word_bitidx;
	unsigned long word;

	bitmap = get_pageblock_bitmap(page, pfn);
	bitidx = pfn_to_bitidx(page, pfn);
	word_bitidx = bitidx / BITS_PER_LONG;
	bitidx &= (BITS_PER_LONG-1);

	word = bitmap[word_bitidx];
	bitidx += end_bitidx;
	return (word >> (BITS_PER_LONG - bitidx - 1)) & mask;
}

```
The bits of one page-block is stored from the most significant bit of the unsigned long. 
 
## __GFP_COLD
buddy allocation mask bit. Specifies that allocation of a "cold" page that is not resident in the CPU cache is required.

## __GFP_COMP 
buddy allocation mask bit. If __GFP_COMP is set and more than one page has been requested, the kernel must group the pages into compand pages. The first page is called the head page, while all other pages are called tail pages. 

All pages are identified as compound pages by the PG_compound bit. The private elements of the page instance of all pages - even the head page itself - point to the head page. Besides, the kernel needs to store information on how many pages compose the compound page. The LRU list element of the first tail page is abused for this purpose: A pointer to a destructor function is thus kept in lru.next, while the allocation order is stored in lru.prev. Notice that the lru element cannot be used for this purpose because it is required if the compound page is to be kept on a kernel list. 



Why is this information required? The kernel can combine multiple adjacent physical pages to a so-called huge-TLB page. When a userland application works with large chunks of data, many processors allow using huge-TLB pages to keep the data in memory. Since the page size of a huge-TLB page is larger than the regular page size, this reduces the amount of information that must be stored in the translation lookaside buffer (TLB), that, in turn, reduces the probability of a TLB cache miss -- and thus speeds things up. However, huge-TLB pages need to be freed differently than compound pages composed of multiple regular pages, so an explicit destructor is required. free_compound_pages is used for this purpose. The function essentially determines the page order stored in lru.prev and frees the pages one after another when the compound page is freed.


## __GFP_DMA
buddy allocation mask bit, meaning to get free pages from DMA.

## __GFP_HIGHMEME
buddy allocation mask bit, meaning to get free pages from highmem

## __GFP_DMA32
buddy allocation mask bit, meaning to get free pages from DMA

## __GFP_HARDWALL
is meaningful on NUMA systems only. It limits memory allocation to the nodes associated with the CPUs assigned to a process. The flag is meaningless if a process is allowed run on all CPUs(this is the default). It only has an explicit effect if the CPUs on which a process may run is limited.

## __GFP_HIGH
buddy allocation mask bit. It is set if the request is very important, that is, when the kernel urgently needs memory. This flag is always used when failure to allocate memory would have massive consequences for the kernel resulting in a threat to system stability or even a system crash.

## __GFP_IO
buddy allocation mak bit. It specifies that the kernel can perform I/O operation during an attempt to find fresh memory. In rael terms, this means that if the kernel begins to swap out pages during memory allocation, the selected pages may be swapped out.

## __GFP_MOVABLE
buddy allocation mask bit, meaning to get free pages from virtual MOVABLE zone.

## __GFP_NOWARN
suppresses a kernel failure warning if allocation fails. There are very few occations when this flag is useful.

## __GFP_NOFAIL
retries the a failed allocation until it succeeds

## __GFP_RECLAIMABLE
mark that the allocated memory will be movable. So the memory will be taken from the reclaimable sublist of the freelists.

## __GFP_REPEAT
automatically retries a failed allocation but stops after a few attempts.

## __GFP_THISNODE
only makes sense on NUMA systems. It the bit is set, then fallback to other nodes is not permitted, and the memory is guaranteed to be allocated on either the current node or on an explicitly specified node.

## __GFP_WAIT
buddy allocation mask bit. Indicates that the memory request may be interrupted; that is the scheduler is free to select another process during the request, or the request can be interrupted by a more important event. The allocator is also permitted to wait for an event on a queue (and to put the process to sleep) before memory is returned.

## __GFP_ZERO
returns a page filled with zeros if allocation succeeds

