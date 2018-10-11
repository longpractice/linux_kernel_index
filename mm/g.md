# g

## get_page_from_freelist

```c
static struct page *
get_page_from_freelist(gfp_t gfp_mask, unsigned int order, int alloc_flags, const struct alloc_context *ac)
```

function used by the buddy system. It refers to the flags set and the allocation order to decide whether allocation can be made; if so, it initiates actual page allocation.

## __GFP_COLD
buddy allocation mask bit. Specifies that allocation of a "cold" page that is not resident in the CPU cache is required.

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

