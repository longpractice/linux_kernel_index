# C

## cache_estimate
routine used by calculate_slab_order(). Nothing to say since the comments are good enough. This function tells us that the all slabs are page-aligned.

```c
/*
 * Calculate the number of objects and left-over bytes for a given buffer size.
 */
static unsigned int cache_estimate(unsigned long gfporder, size_t buffer_size,
		slab_flags_t flags, size_t *left_over)
{
	unsigned int num;
	size_t slab_size = PAGE_SIZE << gfporder;

	/*
	 * The slab management structure can be either off the slab or
	 * on it. For the latter case, the memory allocated for a
	 * slab is used for:
	 *
	 * - @buffer_size bytes for each object
	 * - One freelist_idx_t for each object
	 *
	 * We don't need to consider alignment of freelist because
	 * freelist will be at the end of slab page. The objects will be
	 * at the correct alignment.
	 *
	 * If the slab management structure is off the slab, then the
	 * alignment will already be calculated into the size. Because
	 * the slabs are all pages aligned, the objects will be at the
	 * correct alignment when allocated.
	 */
	if (flags & (CFLGS_OBJFREELIST_SLAB | CFLGS_OFF_SLAB)) {
		num = slab_size / buffer_size;
		*left_over = slab_size % buffer_size;
	} else {
		num = slab_size / (buffer_size + sizeof(freelist_idx_t));
		*left_over = slab_size %
			(buffer_size + sizeof(freelist_idx_t));
	}

	return num;
}
```

## calculate_slab_order
```c
/**
 * calculate_slab_order - calculate size (page order) of slabs
 * @cachep: pointer to the cache that is being created
 * @size: size of objects to be created in this cache.
 * @flags: slab allocation flags
 *
 * Also calculates the number of objects per slab.
 *
 * This could be made much more intelligent.  For now, try to avoid using
 * high order pages for slabs.  When the gfp() functions are more friendly
 * towards high-order requests, this should be changed.
 */
static size_t calculate_slab_order(struct kmem_cache *cachep,
				size_t size, slab_flags_t flags)
{
	size_t left_over = 0;
	int gfporder;

	for (gfporder = 0; gfporder <= KMALLOC_MAX_ORDER; gfporder++) {
		unsigned int num;
		size_t remainder;

		num = cache_estimate(gfporder, size, flags, &remainder);
		if (!num)
			continue;

		/* Can't handle number of objects more than SLAB_OBJ_MAX_NUM */
		if (num > SLAB_OBJ_MAX_NUM)
			break;

		if (flags & CFLGS_OFF_SLAB) {
			struct kmem_cache *freelist_cache;
			size_t freelist_size;

			freelist_size = num * sizeof(freelist_idx_t);
			freelist_cache = kmalloc_slab(freelist_size, 0u);
			if (!freelist_cache)
				continue;

			/*
			 * Needed to avoid possible looping condition
			 * in cache_grow_begin()
			 */
			if (OFF_SLAB(freelist_cache))
				continue;

			/* check if off slab has enough benefit */
			if (freelist_cache->size > cachep->size / 2)
				continue;
		}

		/* Found something acceptable - save it away */
		cachep->num = num;
		cachep->gfporder = gfporder;
		left_over = remainder;

		/*
		 * A VFS-reclaimable slab tends to have most allocations
		 * as GFP_NOFS and we really don't want to have to be allocating
		 * higher-order pages when we are unable to shrink dcache.
		 */
		if (flags & SLAB_RECLAIM_ACCOUNT)
			break;

		/*
		 * Large number of objects is good, but very large slabs are
		 * currently bad for the gfp()s.
		 */
		if (gfporder >= slab_max_order)
			break;

		/*
		 * Acceptable internal fragmentation?
		 */
		if (left_over * 8 <= (PAGE_SIZE << gfporder))
			break;
	}
	return left_over;
}
```
Note that each slab consists of a certain order of pages. We iterate over oder of pages until we find a good fit. First, we get the remainer and number of objects using cache_estimate function which also takes the on/off-slab flags into consideration.

In case that we attemp off-slab, if (flags & CFLGS_OFF_SLAB), we attempt to find a struct kmem_cache object responsible for allocating the group of off-slab freelist_id_t for one slab. If it is not found, we cannot do off-slab on this gfporder and continue to the next gfporder(`if (!freelist_cache) continue;`).  If we again have OFF_SLAB(freelist_cache), we could possibly end up some strange recursive complicated condition which we would love to avoid. If we have freelist_cache->size > cachep->size / 2, which means that if we would have put freelist on-slab, we would not create much memory waste(we just need have one less object to accomodate the num*sizeof(freelist_idx_t) freelist indices memory). 

Note that we are just trying on off-slab. If this does not fit, when we get out of this function with cachep->num untouched(it was normally set to zero before hand). The caller of this function will again resolve to on-slab calculation.

If we found our left_over is smaller than 1/8 of the slab size, we believe that our fragmentation(memory waste) is small enough, we return with success.



## can_steal_fallback
```c
can_steal_fallback(unsigned int order, int start_mt)
```
when we are falling back to another migratetype during allocation, try to steal extra free pages from the same pageblocks to satisfy further allocations, instead of polluting multiple pageblocks.

If we are stealing a relatively large buddy order, it is likely that there will be more free pages in the pageblock, so try to steal them all. For reclaimable and unmovable allocations, we steal regardless of page size, as fragmentation caused by those allocations polluting movable pageblocks is worse than movable allocations stealing from unmovable and reclaimable pageblocks.

## contig_page_data


the variable representing the single global node for UMA systems. It is of type `struct pglist_data` which is the type of a node

## CONFIG_FORCE_MAX_ZONEORDER
a configuration parameter, when defined, overwrites the MAX_ORDER

## CONFIG_HAVE_MEMBLOCK_NODE_MAP

```c
 /*
  * With CONFIG_HAVE_MEMBLOCK_NODE_MAP set, an architecture may initialise its
  * zones, allocate the backing mem_map and account for memory holes in a more
  * architecture independent manner. This is a substitute for creating the
  * zone_sizes[] and zholes_size[] arrays and passing them to
  * free_area_init_node()
  */
  ```

## cond_schedule()
this is used for a process to trigger reschedule if the condition is safe for it. This gives a chance for other process to run. 

## create_boot_cache()
It is used during the init process of slab allocator. It creates a cache during boot when no slab services are available yet. Defined in slab_common.c.

```c
/* Create a cache during boot when no slab services are available yet */
void __init create_boot_cache(struct kmem_cache *s, const char *name,
		unsigned int size, slab_flags_t flags,
		unsigned int useroffset, unsigned int usersize)
{
	int err;

	s->name = name;
	s->size = s->object_size = size;
	s->align = calculate_alignment(flags, ARCH_KMALLOC_MINALIGN, size);
	s->useroffset = useroffset;
	s->usersize = usersize;

	slab_init_memcg_params(s);

	err = __kmem_cache_create(s, flags);

	if (err)
		panic("Creation of kmalloc slab %s size=%u failed. Reason %d\n",
					name, size, err);

	s->refcount = -1;	/* Exempt from merging for now */
}
```

it firstly sets the simple members of name, size, align, useroffset, usersize. slab_init_memcg_params() does nothing unless CONFIG_MEMCG_KMEM is defined. '

__kmem_cache_create(s, flags)

