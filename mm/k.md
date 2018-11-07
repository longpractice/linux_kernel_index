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

## kmem_cache
struct kmem_cache donates one slab cache. This structure contains all crucial metadata of the cache. It includes a list of slab descriptors, each hosting a page or a group frames. Pages under slabs contain objects or memory blocks, which are the allocation unit of the cache. 

Each cache is responsible for just one object type, instances of struct unix_sock, for example, or general buffers. The number of slambs in each caches vaires according to the number of pages used, the object size, and the number of objects managed. All caches in the system are also kept in a doubly linked list. This give the kernel the opportunity to traverse all caches one after the other; this is necessary, for example, when schrinking cache memory because of an impending memory shortage.

Besides management data(such as the numebr of used and free objects or flags), the cache structure includes two elements of special significance:
* A pointer to an array in which the last freed objects can be kept for each specific CPU
* Three list head per memory node under which slabs can be listd. The first list contains full slabs, the second partially freed slabs and the thrid free slabs.
  
The cache structure points to an array that contains as many entries as there are CPUs in the system. Each element is a pointer to a further structure known as an array cache, which contains the management data for each particular system CPU. The memory area immediately following the management data contains an arrya with pointers to as-yet-unused objects in the slabs.

The per-CPU pointers are important to best exploit the CPU caches. The LIFO principle is applied when objects are allocated and returned. The kernel assums that an object just returned is still in the cache and allocates it again as quickly as possible(in response to the next request). Only when the per-CPU caches are empty are free objects from the slabs used to refill them.

This resulds in a three-level hierarchy for object allocation whitin which both the allocation cost and the negative impact of the operation on caches and TLBs rise from level to level:
1. Per-CPU objects in the CPU cache.
2. Unused objects from an existing slab
3. Unused objects from a new slab just reserved using the buddy system
   
```c
struct kmem_cache {
	struct array_cache __percpu *cpu_cache;

/* 1) Cache tunables. Protected by slab_mutex */
	unsigned int batchcount;
	unsigned int limit;
	unsigned int shared;

	unsigned int size;
	struct reciprocal_value reciprocal_buffer_size;
/* 2) touched by every alloc & free from the backend */

	slab_flags_t flags;		/* constant flags */
	unsigned int num;		/* # of objs per slab */

/* 3) cache_grow/shrink */
	/* order of pgs per slab (2^n) */
	unsigned int gfporder;

	/* force GFP flags, e.g. GFP_DMA */
	gfp_t allocflags;

	size_t colour;			/* cache colouring range */
	unsigned int colour_off;	/* colour offset */
	struct kmem_cache *freelist_cache;
	unsigned int freelist_size;

	/* constructor func */
	void (*ctor)(void *obj);

/* 4) cache creation/removal */
	const char *name;
	struct list_head list;
	int refcount;
	int object_size;
	int align;
/* 5) statistics */
//omitted CONFIG_DEBUG_SLAB specific fields.

	unsigned int useroffset;	/* Usercopy region offset */
	unsigned int usersize;		/* Usercopy region size */

	struct kmem_cache_node *node[MAX_NUMNODES];
};
```
`cpu_cache` is a pointer to an array with an entry for each CPU in the system. 

`batchcount` specifies the number of objects to be taken from the slabs of a cache and added to the per-CPU list if it is empty. It also indicates the number of objects to be allocated when a cache is grown.

`limit` specifies the maximum number of objects that may be held in a per-CPU list. If this value is exceeded, the kernel returns batchcount number of objects to the slabs(if the kernel then shrinks the caches, memory is returned from the slabs to the buddy system). 

`size` specifies the number of objects managed in the cache.

`reciprocal_buffer_size`
suppose the kernel ahs a pointer to an element in a slab and wants to determine the corresponding object index. The easiest way to do this is to devide the offset of the pointer compared to the start of the slab area by the object size. Buffer_size here indicates the size occupied by one object and reciprocal_buffer_size is a structure to facilitate fast calculation of a certain offset dividing by the buffer_size.

`flags` is a flag register to define the global properties of the cache. 




`num` donates the number of objects per slab

`gforder` specifies the slab size of a binray logarithm of the number of pages, or, expressed differently, the slab comprises 2^(gforder) pages.

`colour` specifies the maximum number of colors.

`freelist_cache` 
for each object, the management data(object descriptor) can be positioned either on the slab itself or in an external memory area allocated using kmalloc. Which alternative the kernel selects depends on the size of the slab and of the objects used. When stored outside, external object descriptors are stored in the general cache pointed to by the freelist_cache.

`freelist_size` indicates the size of `freelist_cache`

`ctor` constructor to construct the object in the slab cache

`name` name of the slab cache, normally donate the object type. It is used, for example, to list the available caches in /proc/slabinfo

`list` is used for putting current cache on a global cache list.

`refcount` ref count for current cache

`object_size` the size of the object

`align` the alignment of the object

`useroffset` and `usersize`: TODO


`node` is an array that contains an entry for each possible node in the system. Each entry hold an instance of kmem_cache_node that groups three slab list(full free, partially free) together in a sperate structure. The element must be placed at the end of the structure. While it formally always has MAX_NUMNODES entries, it is possible that fewer nodes are usable on NUMA machines. The array thus requires fewer entries, and the array thus requires fewer entires, and the kernel can achieve this at run time by simply allocating less memory than the array formally requires. This would not be possible if node were placed in the middle of the structure. This arrangement enables quick object allocations, since allocator routines can look up to the partial slab for a free object, and possibly move on to an empty slab if required. It also helps easier expansion of the cache with new page frames to accommodate more objects(when required), and facilitates safe and quick reclaim(slabs in empty state can be reclaimed).

## __kmem_cache_create
__kmem_cache_create - Create a cache. See the added comments under Docu, firstly, the size, alignment and the flags will be adjusted. 

In slab.c:

it firstly set the field of align and colour_off. 

It then calls the set_objfreelist_slab_cache() to set the num, gfporder, and colour field of cachep. If it turns out to be not a good fit, it will run set_off_slab_cache(). Otherwise, it goto done.

set_off_slab_cache does similar things as set_objfreelist_slab_cache() but only for off_slab_cache. 

For "done" used for goto, the interesting part is that if it is off_slab, we need to init the cachep->freelist_cache which is used for object descriptors that are off slab. Note that when __kmeme_cache_create is used in the sequence of kmem_cache_init() -> create_boot_cache() -> __kmem_cache_create(), slab_state is still down. The kmem_cache_init() -> create_boot_cache() -> __kmem_cache_create()->setup_cpu_cache() will point global variable kmem_cache(for allocating struct kmem_cache class) kmem_cache->node to &init_kmem_cache_node array members(0th, 2nd, ... the others 1st, 3rd, ... are used for the struct cachep->node and cachep is for allocating type struct kmem_cache_node).



```c
int __kmem_cache_create(struct kmem_cache *cachep, slab_flags_t flags)
{

	size_t ralign = BYTES_PER_WORD;
	gfp_t gfp;
	int err;
/* Docu: ************* Sanitize the size, flags and the alignment ******************************/
	unsigned int size = cachep->size;
	/*
	 * Check that size is in terms of words.  This is needed to avoid
	 * unaligned accesses for some archs when redzoning is used, and makes
	 * sure any on-slab bufctl's are also correctly aligned.
	 */
	size = ALIGN(size, BYTES_PER_WORD);

	if (flags & SLAB_RED_ZONE) {
		ralign = REDZONE_ALIGN;
		/* If redzoning, ensure that the second redzone is suitably
		 * aligned, by adjusting the object size accordingly. */
		size = ALIGN(size, REDZONE_ALIGN);
	}

	/* 3) caller mandated alignment */
	if (ralign < cachep->align) {
		ralign = cachep->align;
	}
	/* disable debug if necessary */
	if (ralign > __alignof__(unsigned long long))
		flags &= ~(SLAB_RED_ZONE | SLAB_STORE_USER);
	/*
	 * 4) Store it.
	 */
	cachep->align = ralign;
	cachep->colour_off = cache_line_size();
	/* Offset must be a multiple of the alignment. */
	if (cachep->colour_off < cachep->align)
		cachep->colour_off = cachep->align;

	if (slab_is_available())
		gfp = GFP_KERNEL;
	else
		gfp = GFP_NOWAIT;

	kasan_cache_create(cachep, &size, &flags); //Docu: not considered if CONFIG_KASAN is not configured 

	size = ALIGN(size, cachep->align);
	/*
	 * We should restrict the number of objects in a slab to implement
	 * byte sized index. Refer comment on SLAB_OBJ_MIN_SIZE definition.
	 */
	if (FREELIST_BYTE_INDEX && size < SLAB_OBJ_MIN_SIZE)
		size = ALIGN(SLAB_OBJ_MIN_SIZE, cachep->align);

/* Docu: ************* set obj freelist ******************************/
	if (set_objfreelist_slab_cache(cachep, size, flags)) {
		flags |= CFLGS_OBJFREELIST_SLAB;
		goto done;
	}

/* Docu: ************* set_off_slab_cache when obj descripter is off cache ******************************/
	if (set_off_slab_cache(cachep, size, flags)) {
		flags |= CFLGS_OFF_SLAB;
		goto done;
	}
/* Docu: ************* set_off_slab_cache when obj descripter is on cache ******************************/
	if (set_on_slab_cache(cachep, size, flags))
		goto done;

	return -E2BIG;

done:
	cachep->freelist_size = cachep->num * sizeof(freelist_idx_t);
	cachep->flags = flags;
	cachep->allocflags = __GFP_COMP;
	if (flags & SLAB_CACHE_DMA)
		cachep->allocflags |= GFP_DMA;
	if (flags & SLAB_RECLAIM_ACCOUNT)
		cachep->allocflags |= __GFP_RECLAIMABLE;
	cachep->size = size;
	cachep->reciprocal_buffer_size = reciprocal_value(size);
/* Docu: set feelist_cache if offslab, OFF_SLAB(cachep) will be false in kmem_cache_init->create_boot_cache->__kmem_cache_create call sequence */
	if (OFF_SLAB(cachep)) {
		cachep->freelist_cache =
			kmalloc_slab(cachep->freelist_size, 0u);
	}
/* Docu: setup_cpu_cache will do different things according to slab_state, therefore, doing right things during initialization */
	err = setup_cpu_cache(cachep, gfp);
	if (err) {
		__kmem_cache_release(cachep);
		return err;
	}

	return 0;
}

```


## kmem_cache_init
Initialisation of slab allocator called in mm_init() in main.c. Called after the buddy page allocator has been initialized and the bootmem pages are returned to the buddy page allocator.

It is complicated since when calling kmem_cache_init(), the slab system is not ready yet, therefore we use slab_state to branch some operations, as shown in detail below, to make initialization possible.

For kmem_cache_init, first of all(now the global variable of slab_state is enum slab_state::down), the kmem_cache global variable is used for allocating the struct kmem_cache variable itself. It is statically allocated as kmem_cache_boot so that it does not require any dynamic allocation for now.

kmem_cache_init(void): _step_1_
```c
void __init kmem_cache_init(void)
{
	int i;

	kmem_cache = &kmem_cache_boot;
```


```c
/* internal cache of cache description objs */
static struct kmem_cache kmem_cache_boot = {
	.batchcount = 1,
	.limit = BOOT_CPUCACHE_ENTRIES,
	.shared = 1,
	.size = sizeof(struct kmem_cache),
	.name = "kmem_cache",
};
```

Note that although the kmem_cache_boot is statically allocated, its field of `struct kmem_cache_node *node[MAX_NUMNODES]` is still not valid. The allocation cannot be taken from there. We need to use init_kmem_cache_node which is statically allocated.  Note that each kmem_cache_init needs MAX_NUMNODES of kmem_cache_node. init_kmem_cache_node[2 * MAX_NUMNODES] are therefore used for both the kmem_cache for allocating struct kmem_cache and the kmem_cache for allocating struct kmem_cache_node. It will be seen later how this is utilized in set_up_node().

```c
#define NUM_INIT_LISTS (2 * MAX_NUMNODES)
static struct kmem_cache_node __initdata init_kmem_cache_node[NUM_INIT_LISTS];
```

For kmem_cache_init, then it will initialize the init_kmem_cache_node static variable using kmem_cache_node_init(which is simple to init the fields of init_kmem_cache_node[i]):


kmem_cache_init(void): _step_2_
```c
	for (i = 0; i < NUM_INIT_LISTS; i++)
		kmem_cache_node_init(&init_kmem_cache_node[i]);
```


For kmem_cache_init, it then try to init the kmem_cache variable which is &kmem_cache_boot discussed before. This cache is used for allocating type struct kmem_cache. Note that the kmem_cache's last field does not need to be fully MAX_NUMNODES and be allocated according to the real number of nodes(nr_node_ids). The alignment requirements of allocating struct kmem_cache is required to be hardware cache alignment(SLAB_HWCACHE_ALIGN). kmem_cache is then put on the list of kmem_cache. From now on, the global variable of slab_state is set to PARTIAL. 

kmem_cache_init(void): _step_3_
```c
	/*
	 * struct kmem_cache size depends on nr_node_ids & nr_cpu_ids
	 */
	create_boot_cache(kmem_cache, "kmem_cache",
		offsetof(struct kmem_cache, node) +
				  nr_node_ids * sizeof(struct kmem_cache_node *),
				  SLAB_HWCACHE_ALIGN, 0, 0);
	list_add(&kmem_cache->list, &slab_caches);
	memcg_link_cache(kmem_cache); //doing nothing if CONFIG_MEMCG_KMEM is not defined
	slab_state = PARTIAL;
```

the important calling routine sequence for the above part is kmem_cache_init() -> create_boot_cache() -> __kmem_cache_create(); Note that in the __kmem_cache_create(), in this perticular case, we will not have an off-slab cache here. We will have a on-slab cache since we return true from set_objfreelist_slab_cache() in __kmem_cache_create(), therefore for this cache, we do not need and do not set the kmem_cache->freelist_size in __kmem_cache_create()(in the following function from __kmeme_cache_create, kmaloc_slad give the slot in global kmalloc_caches array which is yet not initialized, we therefore overcomes the chicken-and-egg problem here):



part of __kmem_cache_create() that needs to use the array of kmalloc_caches when off_slab(but that is not the case here): 
```c
	if (OFF_SLAB(cachep)) {
		cachep->freelist_cache =
			kmalloc_slab(cachep->freelist_size, 0u);
	}.
```

Now that we have the variable kmem_cache global on-slab cache properly initialized and the slab_state is in partial. We therefore at least could create objects of type struct kmem_cache using our global variable of kmem_cache(the name of global object happens to equal to the name of the type without struct keyword, kind of misleading in documentations. When we explicitly say struct kmem_cache, we mean the type).

Keep going on kmem_cache_init, it then initializes the kmalloc_caches array element that is used for allocating struct kmem_cache_node using create_kmalloc_cache.

kmem_cache_init(void): _step_4_
```c
	/*
	 * Initialize the caches that provide memory for the  kmem_cache_node
	 * structures first.  Without this, further allocations will bug.
	 */
	kmalloc_caches[INDEX_NODE] = create_kmalloc_cache(
				kmalloc_info[INDEX_NODE].name,
				kmalloc_size(INDEX_NODE), ARCH_KMALLOC_FLAGS,
				0, kmalloc_size(INDEX_NODE));
	slab_state = PARTIAL_NODE;
```

create_kmalloc_cache is defined as:
```c
struct kmem_cache *__init create_kmalloc_cache(const char *name,
		unsigned int size, slab_flags_t flags,
		unsigned int useroffset, unsigned int usersize)
{
	struct kmem_cache *s = kmem_cache_zalloc(kmem_cache, GFP_NOWAIT);

	if (!s)
		panic("Out of memory when creating slab %s\n", name);

	create_boot_cache(s, name, size, flags, useroffset, usersize);
	list_add(&s->list, &slab_caches);
	memcg_link_cache(s);
	s->refcount = 1;
	return s;
}
```

we first allocate a variable s of the type of struct kmem_cache. This is fine, since we have already initialized the kmem_cache global variable which serves this purpose in _step_3_. We then need to call create_boot_cache which we are familiar with(used before in kmem_cache_init to in _step_3_). Remind that the calling chain is create_boot_cache() -> __kmem_cache_create() -> setup_cpu_cache() -> setup_node().  The params passed to setup_node depends on state.When the slab_state==PARIAL, will call set_up_node(cachep, SIZE_NODE). set_up_node(cachep, SIZE_NODE) will set the struct kmem_cache::node array of cachep(cachep here is the `s` inside create_kmalloc_cache as shown above code snapet). 
Since the SIZE_NODE is one, the s->node array will be set to the odd-indexed(remember that even indexed elements are give to the struct kmem_cache::node for the static variable of kmem_cache used for allocating objects of type struct kmem_cache) elements of static array of init_kmem_cache_node.

Keep going with kmem_cache_init. Let's stage what we have here. We are now at a slab_state of PARTIAL_NODE. We have two objects of type struct kmem_cache (one is static var with name of kmem cache, to allocate struct kmem_cache and the other one is kmalloc_caches[INDEX_NODE] used for allocate struct kmem_cache_node). These two objects have their node array fields point to the static init_kmem_cache_node which is only a temporary solution here which will be remedied soon.

Keep going with kmem_cache_init for step_5, we call setup_kmalloc_cache_index_table() to patch the array of size_index. It does not affect our logic flow.


kmem_cache_init(void): _step_5_
```c
	setup_kmalloc_cache_index_table();
	slab_early_init = 0;
```

Then, as noted before, the node elements for two struct kmem_cache objects(one for alloc struct kmem_cache and one for alloc struc kmem_cache_node) are inited to init_kmem_cache_node which is only one temporary solution. The next step  will remedy this:

kmem_cache_init(void): _step_6_
```c
	{
		int nid;

		for_each_online_node(nid) {
			init_list(kmem_cache, &init_kmem_cache_node[CACHE_CACHE + nid], nid);

			init_list(kmalloc_caches[INDEX_NODE],
					  &init_kmem_cache_node[SIZE_NODE + nid], nid);
		}
	}
```
init_list function will allocate new kmem_cache_node objects(we are ready to do that already) and copy the corresponding init_kmem_cache_node element in.


Finally, we call create_kmalloc_caches will fill the yet-un-inited kmem_ca
kmem_cache_init(void): _step_7_
```c
		create_kmalloc_caches(ARCH_KMALLOC_FLAGS);
		//docu: close the end of above func, the state of slab will be set to up
```

create_kmalloc_caches populates kmalloc_caches static array with new_kmalloc_cache function. The basic calling sequence is create_malloc_caches() -> (for each kmalloc_caches element) new_kmalloc_cache() -> create_kmalloc_cache() -> create_boot_cache() -> _kmem_cache_create() -> setup_cpu_cache(). In setup_cpu_cache, since the slab_state is now PARTIAL_NODE, we first allocate the cachep->node array and then set them using kmem_cache_node_init function.

create_kmalloc_caches will also set the slab_state to up! We are almost done now.


The analysis of kmem_cache_init is done. Note that this function is called inside the start_kernel->mm_init->kmem_cache_init. 

Inside start_kernel, later, we will call kmem_cache_init_late() which enables the cpu cache and makes slab_state to FULL.














## kmem_cache_node
struct kmem_cache_node holds all the slabs for a certain node. It is contained by kmem_cache.
```c
/*
 * The slab lists for all objects.
 */
struct kmem_cache_node {
	spinlock_t list_lock;
	struct list_head slabs_partial;	/* partial list first, better asm code */
	struct list_head slabs_full;
	struct list_head slabs_free;
	unsigned long total_slabs;	/* length of all slab lists */
	unsigned long free_slabs;	/* length of free slab list only */
	unsigned long free_objects;
	unsigned int free_limit;
	unsigned int colour_next;	/* Per-node cache coloring */
	struct array_cache *shared;	/* shared per node */
	struct alien_cache **alien;	/* on other nodes */
	unsigned long next_reap;	/* updated without locking */
	int free_touched;		/* updated without locking */
};
```
`slabs_partial`, `slab_full` and `slabs_free` holds the list of slabs of three kinds.

