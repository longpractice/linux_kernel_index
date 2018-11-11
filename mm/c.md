# C

## ____cache_alloc
allocate one object from a certain slab cache according to certain flags.
```c
static inline void *____cache_alloc(struct kmem_cache *cachep, gfp_t flags)
{
	void *objp;
	struct array_cache *ac;

	check_irq_off();

	ac = cpu_cache_get(cachep);
	if (likely(ac->avail)) {
		ac->touched = 1;
		objp = ac->entry[--ac->avail];

		STATS_INC_ALLOCHIT(cachep);
		goto out;
	}

	STATS_INC_ALLOCMISS(cachep);
	objp = cache_alloc_refill(cachep, flags);
	/*
	 * the 'ac' may be updated by cache_alloc_refill(),
	 * and kmemleak_erase() requires its correct value.
	 */
	ac = cpu_cache_get(cachep);

out:
	/*
	 * To avoid a false negative, if an object that is in one of the
	 * per-CPU caches is leaked, we need to make sure kmemleak doesn't
	 * treat the array pointers as a reference to the object.
	 */
	if (objp)
		kmemleak_erase(&ac->entry[ac->avail]);
	return objp;
}

```
The first choice is from CPU cache, which is likely to be hot. cpu_cache_get(cachep) get the array_cache element with some compile time checking that it is indeed a percpu var.
If we have ac->avail, object will be taken from the percpu cache. Otherwise, we will have to call the cache_alloc_refill to refill the percpu cache from the kmem_cache->node variables.

## cache_alloc_refill
used for __cache_alloc and defined in slab.c. It is used for slab allocator to refill the cachep->cpu_cache from the slabs in cachep->node.

---
cache_alloc_refill(partial):
```c
static void *cache_alloc_refill(struct kmem_cache *cachep, gfp_t flags)
{
	int batchcount;
	struct kmem_cache_node *n;
	struct array_cache *ac, *shared;
	int node;
	void *list = NULL;
	struct page *page;

	check_irq_off();
	node = numa_mem_id();

	ac = cpu_cache_get(cachep);
	batchcount = ac->batchcount;
	if (!ac->touched && batchcount > BATCHREFILL_LIMIT) {
		/*
		 * If there was little recent activity on this cache, then
		 * perform only a partial refill.  Otherwise we could generate
		 * refill bouncing.
		 */
		batchcount = BATCHREFILL_LIMIT;
	}
	n = get_node(cachep, node);

	BUG_ON(ac->avail > 0 || !n);

```
First we do some preparation. Remember that batchcount is the number of objects we refill once from node to the cachep->cpu_cache.

---
cache_alloc_refill(partial):
```c
	shared = READ_ONCE(n->shared);
	if (!n->free_objects && (!shared || !shared->avail))
		goto direct_grow;


```
When both of the following are true:
1. there is no more free_objects on current node
2. there is no shared array_cache or the shared array_cache has no more free objects(shared->avail == 0),
   
we have no more free objects in all of our slabs and we have to directly goto direct_grow label to grow the cache by adding slabs.


---
cache_alloc_refill(partial):
```c
	spin_lock(&n->list_lock);
	shared = READ_ONCE(n->shared);
	/* See if we can refill from the shared array */
	if (shared && transfer_objects(ac, shared, batchcount)) {
		shared->touched = 1;
		goto alloc_done;
	}
```
If we are not going to directly grow, we will firstly try to transfer from shared array_cache before probing the slabs in the nodes. It may only full-fill some of the batchcount. We need to further try out the nodes if our batchount is not yet zero.

---
cache_alloc_refill(partial):
```c

	while (batchcount > 0) {
		/* Get slab alloc is to come from. */
		page = get_first_slab(n, false);
		if (!page)
			goto must_grow;

		check_spinlock_acquired(cachep);

		batchcount = alloc_block(cachep, ac, page, batchcount);
		fixup_slab_list(cachep, n, page, &list);
	}
```
We try to fill the batchcount by finding the free page in partial filled slabs or free slabs(searched in get_first_slab). Here is a link discussing why the slab information is now in struct page: https://lwn.net/Articles/565097/. 

```c
/*
 * Slab list should be fixed up by fixup_slab_list() for existing slab
 * or cache_grow_end() for new slab
 */
static __always_inline int alloc_block(struct kmem_cache *cachep,
		struct array_cache *ac, struct page *page, int batchcount)
{
	/*
	 * There must be at least one object available for
	 * allocation.
	 */
	BUG_ON(page->active >= cachep->num);

	while (page->active < cachep->num && batchcount--) {
		STATS_INC_ALLOCED(cachep); 
		STATS_INC_ACTIVE(cachep);
		STATS_SET_HIGH(cachep);

		ac->entry[ac->avail++] = slab_get_obj(cachep, page);
	}

	return batchcount;
}
```
Note that page->active shows the number of used objects in the page and the cachep->num shows the total number of objs in one slab. page->active < cachep->num therefore tells whether we still have free objects. The three calls of STATS_* functions does nothing when STATS is not defined. slab_get_obj gets the free object from the page->freelist(See slab_get_obj).

Our slab will no longer be free and it will become either partial or empty. fixup_slab_list will move the slab to a correct list.

```c

static inline void fixup_slab_list(struct kmem_cache *cachep,
				struct kmem_cache_node *n, struct page *page,
				void **list)
{
	/* move slabp to correct slabp list: */
	list_del(&page->lru);
	if (page->active == cachep->num) {
		list_add(&page->lru, &n->slabs_full);
		if (OBJFREELIST_SLAB(cachep)) {
			page->freelist = NULL;
		}
	} else
		list_add(&page->lru, &n->slabs_partial);
}
```

fixup_slab_list first remove the page(therefore also the slab it represents) from the its original list. It the slab is now full, it will be put in the node->slabs_full list. Otherwise, it will be put in the node->slabs_partial list. The cachep->flags is checked also to see whether we have objfreelist here which means that our freelist array will be on a free object on the slab. If we have objfreelist and we have no more free object, we set our page->freelist to NULL. This is used for marking this condition, objfreelist full slab, directly. When we put free obj on the slab, if we see that page->freelist is NULL, we will point it to the newly put free object which is the first free object(also will be the last to be used later).



---
cache_alloc_refill(partial):
```
must_grow:
	n->free_objects -= ac->avail;
alloc_done:
	spin_unlock(&n->list_lock);
	fixup_objfreelist_debug(cachep, &list);

direct_grow:
	if (unlikely(!ac->avail)) {
		/* Check if we can use obj in pfmemalloc slab */
		if (sk_memalloc_socks()) {
			void *obj = cache_alloc_pfmemalloc(cachep, n, flags);

			if (obj)
				return obj;
		}

		page = cache_grow_begin(cachep, gfp_exact_node(flags), node);

		/*
		 * cache_grow_begin() can reenable interrupts,
		 * then ac could change.
		 */
		ac = cpu_cache_get(cachep);
		if (!ac->avail && page)
			alloc_block(cachep, ac, page, batchcount);
		cache_grow_end(cachep, page);

		if (!ac->avail)
			return NULL;
	}
	ac->touched = 1;

	return ac->entry[--ac->avail];
}
```
Now, we go in must_grow label. Before calling cache_alloc_refill, we have ac->avail == 0. We have already moved ac_avail objects from the nodes, and hence need to subtract ac->avail from n->free_objects.

Then in direct_grow label, if we still have zero object in cpu_cache(ac->avail == 0), we have to fill our slabs by requesting more pages with our buddy page allocator. sk_memalloc_socks() will be false unless we have CONFIG_NET.




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

## cache_grow_begin
grow the number of slabs in a slab cache. It is defined in slab.c and used by cache_alloc_refill when we do not have enough free objects in the slabs.

cache_grow_begin(partial):
```c
/*
 * Grow (by 1) the number of slabs within a cache.  This is called by
 * kmem_cache_alloc() when there are no active objs left in a cache.
 */
static struct page *cache_grow_begin(struct kmem_cache *cachep,
				gfp_t flags, int nodeid)
{
	void *freelist;
	size_t offset;
	gfp_t local_flags;
	int page_node;
	struct kmem_cache_node *n;
	struct page *page;
```

First we prepare some variables.

---

cache_grow_begin(partial):

```c
	/*
	 * Be lazy and only check for valid flags here,  keeping it out of the
	 * critical path in kmem_cache_alloc().
	 */
	if (unlikely(flags & GFP_SLAB_BUG_MASK)) {
		gfp_t invalid_mask = flags & GFP_SLAB_BUG_MASK;
		flags &= ~GFP_SLAB_BUG_MASK;
		pr_warn("Unexpected gfp: %#x (%pGg). Fixing up to gfp: %#x (%pGg). Fix your code!\n",
				invalid_mask, &invalid_mask, flags, &flags);
		dump_stack();
	}
	WARN_ON_ONCE(cachep->ctor && (flags & __GFP_ZERO));
	local_flags = flags & (GFP_CONSTRAINT_MASK|GFP_RECLAIM_MASK);

	check_irq_off();
	if (gfpflags_allow_blocking(local_flags))
		local_irq_enable();
```

We then check the flags.

---

cache_grow_begin(partial):

```c
	/*
	 * Get mem for the objs.  Attempt to allocate a physical page from
	 * 'nodeid'.
	 */
	page = kmem_getpages(cachep, local_flags, nodeid);
	if (!page)
		goto failed;
```

We then try to get more pages from the buddy page allocator. The important funciton here is

```c
/*
 * Interface to system's page allocator. No need to hold the
 * kmem_cache_node ->list_lock.
 *
 * If we requested dmaable memory, we will get it. Even if we
 * did not request dmaable memory, we might get it, but that
 * would be relatively rare and ignorable.
 */
static struct page *kmem_getpages(struct kmem_cache *cachep, gfp_t flags,
								int nodeid)
{
	struct page *page;
	int nr_pages;

	flags |= cachep->allocflags;

	page = __alloc_pages_node(nodeid, flags, cachep->gfporder);
	if (!page) {
		slab_out_of_memory(cachep, flags, nodeid);
		return NULL;
	}

	if (memcg_charge_slab(page, flags, cachep->gfporder, cachep)) {
		__free_pages(page, cachep->gfporder);
		return NULL;
	}

	nr_pages = (1 << cachep->gfporder);
	if (cachep->flags & SLAB_RECLAIM_ACCOUNT)
		mod_lruvec_page_state(page, NR_SLAB_RECLAIMABLE, nr_pages);
	else
		mod_lruvec_page_state(page, NR_SLAB_UNRECLAIMABLE, nr_pages);

	__SetPageSlab(page);
	/* Record if ALLOC_NO_WATERMARKS was set when allocating the slab */
	if (sk_memalloc_socks() && page_is_pfmemalloc(page))
		SetPageSlabPfmemalloc(page);

	return page;
}

```
__alloc_pages_node attemps to ask for order of cachep->gfporder pages from the buddy allocator. It eventually delegate the task to __alloc_pages_nodemask which is the "heart" of the zoned buddy allocator. If we could not get any pages from buddy allocator, we let slab_out_of_memory to do some dbg print(if not debug, do nothing) and return NULL. Then memcg_charge_slab will do nothing unless we have CONFIG_MEMCG_KMEM. Then mod_lruvec_page_state will modify the some counter status stored in pglist_data->vm_state and the vm_node_state.  



---


cache_grow_begin(partial):
```c

	page_node = page_to_nid(page);
	n = get_node(cachep, page_node);

	/* Get colour for the slab, and cal the next value. */
	n->colour_next++;
	if (n->colour_next >= cachep->colour)
		n->colour_next = 0;

	offset = n->colour_next;
	if (offset >= cachep->colour)
		offset = 0;

	offset *= cachep->colour_off;
```
Here we set some colouring. It is done this way since we want to make sure we could have concurrency in accessing n->colour_next without using locks(that is way before the commit of 03d1d43a1262b347a9aa814980438fff8eb32edc). offset = n->colour_next may not be the one ceiled in `if (n->colour_next >= cachep->colour) n->colour_next = 0;` since other thread may in the meantime update n->colour_next. But the check of offset >= cachep->colour deserves a patch since we should allow the boundary value of cachep->colour to be used.



```
	/* Get slab management. */
	freelist = alloc_slabmgmt(cachep, page, offset,
			local_flags & ~GFP_CONSTRAINT_MASK, page_node);
	if (OFF_SLAB(cachep) && !freelist)
		goto opps1;

	slab_map_pages(cachep, page, freelist);

	kasan_poison_slab(page);
	cache_init_objs(cachep, page);

	if (gfpflags_allow_blocking(local_flags))
		local_irq_disable();

	return page;

opps1:
	kmem_freepages(cachep, page);
failed:
	if (gfpflags_allow_blocking(local_flags))
		local_irq_disable();
	return NULL;
}

```
offset is the coloring space, calculated as n->colour_off * (++n->colour_next). n->colour_next will be set to zero if it turns to be greater or equal to the cachep->colour which is calculated according to the space left from all the objects in slab.

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

