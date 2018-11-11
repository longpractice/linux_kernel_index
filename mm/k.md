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

`colour` specifies the maximum number of colors. The number of colour_offs must be smaller(not allowed to equal!) this value. Say we have colour == 5, then we could only have 0*colour_off to 4*colour_off colouring.

`colour_off` specifies the unit of colouring in bytes. Say, we have colour_off as 32 and colour is 4, we will have colour of 0, 32, 64, 96, 0, 32, 64, 96, 0, 32... on different slabs

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

## kmem_cache_create
```c
struct kmem_cache *
kmem_cache_create(const char *name, unsigned int size, unsigned int align,
		slab_flags_t flags, void (*ctor)(void *))
{
	return kmem_cache_create_usercopy(name, size, align, flags, 0, 0,
					  ctor);
}
```
routine used to create a new slab cache.

name is human-readable names appear in /proc/slabinfo. size is the size of managed objects in bytes. align is the alignment requirement of the allocation objects. ctor will be called when create one object. The task is delegated to kmem_cache_create_usercopy with 0 useroffset and 0 usersize.

## __kmem_cache_create
```c
int __kmem_cache_create(struct kmem_cache *cachep, slab_flags_t flags)
```

Defined in slab.c, core function to create a cache and set the `struct kmem_cache *cachep` for this cache(*cachep must be allocated before this call). It is used in create_cache and create_boot_cache. 

When calling this function, the cachep->object_size, cachep->size, cachep->align must be initialized. But some fields are not sanitized. For example cachep->size is not sanitised in that cachep->size does not need to consider alignment and in the kernel code simply set to the value of object_size before calling __kmem_cache_create(). It is not very clear in this part of code what are the requirements of cachep passed in here. We will see how cachp is further processed inside this __kmem_cache_create().

---

__kmem_cache_create(partial):
```c
int __kmem_cache_create(struct kmem_cache *cachep, slab_flags_t flags)
{

	size_t ralign = BYTES_PER_WORD;
	gfp_t gfp;
	int err;
	unsigned int size = cachep->size;
```
the alignment ralign is firstly inited to BYTES_PER_WORD, but is subject to larger values later according to flags and cachep->align. 

gfp is the buddy page allocation flag. It will be set to GFP_KERNEL or GFP_NOWAIT later. gfp will be used by setup_cpu_cache(cachep, gfp) later.

size is the initial value of cachep->size which is just equal to cachep->object_size without considering alignment.


---
__kmem_cache_create(continue):
```c
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
```
Firstly, we floor the size to BYTES_PER_WORD. If we have a REDZONE_ALIGN flag, which means that an additional memory area filled with a known byte pattern is placed at the start and end of each object, we make sure our size is subject also to alignment of REDZONE_ALIGN(__alignof__(unsigned long long)).  Then, ralign is floored to cachep->align. Note that cachep->align could have been set according to whether flags contain SLAB_HWCACHE_ALIGN and the user requirement, as seen from how the kmem_cache_create_usercopy calls create_cache():

part of kmem_cache_create_usercopy which presets alignment:
```c
	s = create_cache(cache_name, size,
			 calculate_alignment(flags, align, size),
			 flags, useroffset, usersize, ctor, NULL, NULL);
```

Then if we have our alignment requirement greater than the alignment of the red zoning, we need to disable the red zoning. It is because we do not want our added red zone at the start and at the end of the object disturb the object alignment.

Finally, the calculated ralign is saved back to cachep->align.


---

__kmem_cache_create(continue):
```c
	cachep->colour_off = cache_line_size();
	/* Offset must be a multiple of the alignment. */
	if (cachep->colour_off < cachep->align)
		cachep->colour_off = cachep->align;
	if (slab_is_available())
		gfp = GFP_KERNEL;
	else
		gfp = GFP_NOWAIT;

	kasan_cache_create(cachep, &size, &flags);

	size = ALIGN(size, cachep->align);
	/*
	 * We should restrict the number of objects in a slab to implement
	 * byte sized index. Refer comment on SLAB_OBJ_MIN_SIZE definition.
	 */
	if (FREELIST_BYTE_INDEX && size < SLAB_OBJ_MIN_SIZE)
		size = ALIGN(SLAB_OBJ_MIN_SIZE, cachep->align);
```

The colour_off is set to the larger of cache_line_size() and cachep->align. For Intel cpu, that equals to l3 cache line size in bytes. The cache line(no matter l1, l2 or l3) size for Intel Core i7 cpu is 64 bytes(64 bytes of a memory block corresponding to an address span of 64, since we have byte-addressable memory). Therefore, cache_line_size() returns, for Intel Core i7 as an example, 64. 

Setting colour_off to the cache_line_size also shows that the colouring added to the beginning of the slab affects the set bits(a mem address is divided into , from higher significance bit to lower: tag bits, set bits and block offset bits) of the object address. Therefore, the objects on different slabs are more likely to go to different sets in the cache system, avoiding frequently flushing eaching other. 

If we somewhat have a preset alignment requirement greater than cache_line_size(), we are quite safe that our colouring will end up affecting only the set bits. 

For Core i7 L1, we have 32KB total L1 cache memory and each memory block is 64 bytes and we have therefore 32k/64 = 512 cache lines. L1 is 8 way associative and we therefore have 512/8=64 sets. That corresponds to address span of 64 * 64=4096 for the set bits which is normally much larger than the slab object alignment requirement of cachep->align. Say we have an address of 64 bits. In this example, for a cpu caching system, we have, from least significant to most significant bits, 8bits for block offset, 8 bits for set, and 48 bits for tags. Our coloring will mostly affect the set bits here.

kasan_cache_create only makes a difference when KernelAddressSANitizer CONFIG_KASA is set. We do not consider it here. The size is then floored again subject to KernelAddressSANitizer. We also make sure that size is no smaller than SLAB_OBJ_MIN_SIZE.

```c
/*
 * This restriction comes from byte sized index implementation.
 * Page size is normally 2^12 bytes and, in this case, if we want to use
 * byte sized index which can represent 2^8 entries, the size of the object
 * should be equal or greater to 2^12 / 2^8 = 2^4 = 16.
 * If minimum size of kmalloc is less than 16, we use it as minimum object
 * size and give up to use byte sized index.
 */
#define SLAB_OBJ_MIN_SIZE      (KMALLOC_MIN_SIZE < 16 ? \
                               (KMALLOC_MIN_SIZE) : 16)
```
---


__kmem_cache_create(continue):
```c

	if (set_objfreelist_slab_cache(cachep, size, flags)) {
		flags |= CFLGS_OBJFREELIST_SLAB;
		goto done;
	}

	if (set_off_slab_cache(cachep, size, flags)) {
		flags |= CFLGS_OFF_SLAB;
		goto done;
	}

	if (set_on_slab_cache(cachep, size, flags))
		goto done;

	return -E2BIG;
```

This part attemps to further initialize cachep. There are three choices of putting a freelist. set_objfreelist_slab_cache, set_off_slab_cache or set_on_slab_cache.

The first one is objfreelist, which puts the freelist on one of the free objects(must be the last one to be used for ____cache_alloc) in the slab. For this mode, it is even fine when we give out this last free object for ____cache_alloc, since it is no longer a free or partial slab(it is a full slab) and hence freelist is no longer needed. 
 

```c
static bool set_objfreelist_slab_cache(struct kmem_cache *cachep,
			size_t size, slab_flags_t flags)
{
	size_t left;

	cachep->num = 0;

	if (cachep->ctor || flags & SLAB_TYPESAFE_BY_RCU)
		return false;

	left = calculate_slab_order(cachep, size,
			flags | CFLGS_OBJFREELIST_SLAB);
	if (!cachep->num)
		return false;

	if (cachep->num * sizeof(freelist_idx_t) > cachep->object_size)
		return false;

	cachep->colour = left / cachep->colour_off;

	return true;
}

static bool set_on_slab_cache(struct kmem_cache *cachep,
			size_t size, slab_flags_t flags)
{
	size_t left;

	cachep->num = 0;

	left = calculate_slab_order(cachep, size, flags);
	if (!cachep->num)
		return false;

	cachep->colour = left / cachep->colour_off;

	return true;
}

```
The first one is objfreelist, which puts the freelist on one of the free objects(must be the last one to be used for ____cache_alloc) in the slab. For this mode, it is even fine when we give out this last free object for ____cache_alloc, since it is no longer a full slab or partial slab and freelist is no longer needed. This choice is the best since it takes no extra space for freelist. In this case, when we are calculating calculate_slab_order, we pass in the CFLGS_OBJFREELIST_SLAB flag. This tells calculate_slab_order that the memory space taken by on object is the aligned size instead of aligned size + sizeof(freelist_idx_t). The `cachep->num * sizeof(freelist_idx_t) > cachep->object_size` tell us that one object space will not be enough to accomodate a freelist which is of size `cachep->num * sizeof(freelist_idx_t)` .

The third choice of set_on_slab_cache means that the freelist is put at the end of slab. The strategy of telling whether this choice is good is similar to objfreelist, but when we calculate calculate_slab_order, each object will occupy mem space of aligned size + sizeof(freelist_idx_t) and we do not need to require that the freelist size smaller than one object size.


```c
static bool set_off_slab_cache(struct kmem_cache *cachep,
			size_t size, slab_flags_t flags)
{
	size_t left;

	cachep->num = 0;

	/*
	 * Always use on-slab management when SLAB_NOLEAKTRACE
	 * to avoid recursive calls into kmemleak.
	 */
	if (flags & SLAB_NOLEAKTRACE)
		return false;

	/*
	 * Size is large, assume best to place the slab management obj
	 * off-slab (should allow better packing of objs).
	 */
	left = calculate_slab_order(cachep, size, flags | CFLGS_OFF_SLAB);
	if (!cachep->num)
		return false;

	/*
	 * If the slab has been placed off-slab, and we have enough space then
	 * move it on-slab. This is at the expense of any extra colouring.
	 */
	if (left >= cachep->num * sizeof(freelist_idx_t))
		return false;

	cachep->colour = left / cachep->colour_off;

	return true;
}

```
Our second choice is off-slab freelist. Which requires that we allocate the freelist using another cache. set_off_slab_cache does not enforce off-slab, instead, it will return false if it sees that off-slab is not a good choice. 

Inside set_off_slab_cache, when calling calculate_slab_order, we have passed in the flag of CFLGS_OFF_SLAB. calculate_slab_order will take this into consideration(see `calculate_slab_order` for details).

Then, we strategically further verify if (left >= cachep->num * sizeof(freelist_idx_t)), under which case, we would better put it at the end of a slab since that left will be wasted otherwise.

In it turns out that off-slab is not a good choice, we finally have to put freelist on-slab at the end of slab using set_on_slab_cache. This could not fail unless our object size to too large to fit in the largest slab possible.

---
__kmem_cache_create: last part 
```c
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
		if (OFF_SLAB(cachep)) {
		cachep->freelist_cache =
			kmalloc_slab(cachep->freelist_size, 0u);
	}

	err = setup_cpu_cache(cachep, gfp);
	if (err) {
		__kmem_cache_release(cachep);
		return err;
	}

	return 0;
}
```
in "done" label, values are stored back in cachep withs some final clean up. We create the cachep->freelist_cache if the freelist ids are off slab. As discussed in `calculate_slab_order`, off-slab will not be selected when we do not have the cache ready to allocate cachep->freelist_size during bootstrapping. We are therefore free from allocating cache->freelist_size in __kmem_cache_create during slab initializing process of kmem_cache_init.

We also call setup_cpu_cache to set up the field of cachep->cpu_cache(the task is delegated to per-cpu allocator, another allocating system). 

setup_cpu_cache also setup the cachep->node array, according to the slab_state(see setup_cpu_cache for details).






## kmem_cache_create_usercopy
Defined in slab_common.c. 
```c
/*
 * kmem_cache_create_usercopy - Create a cache.
 * @name: A string which is used in /proc/slabinfo to identify this cache.
 * @size: The size of objects to be created in this cache.
 * @align: The required alignment for the objects.
 * @flags: SLAB flags
 * @useroffset: Usercopy region offset
 * @usersize: Usercopy region size
 * @ctor: A constructor for the objects.
 *
 * Returns a ptr to the cache on success, NULL on failure.
 * Cannot be called within a interrupt, but can be interrupted.
 * The @ctor is run when new pages are allocated by the cache.
 *
 * The flags are
 *
 * %SLAB_POISON - Poison the slab with a known test pattern (a5a5a5a5)
 * to catch references to uninitialised memory.
 *
 * %SLAB_RED_ZONE - Insert `Red' zones around the allocated memory to check
 * for buffer overruns.
 *
 * %SLAB_HWCACHE_ALIGN - Align the objects in this cache to a hardware
 * cacheline.  This can be beneficial if you're counting cycles as closely
 * as davem.
 */
```
the calling sequence is 

kmem_cache_create_usercopy() -> 
	__kmem_cache_alias() and 
	if not that is not possible create_cache()

__kmem_cache_alias() -> find_mergeable()

create_cache() -> kmem_cache_zalloc() and __kmem_cache_create()



## kmem_cache_init
Routine used for initializing slab allocator and is called in mm_init() in start_kernel() in main.c. Called after the buddy page allocator has been initialized and the bootmem are returned to the buddy page allocator. It does most of the initialization task and some small amount task is left over to kmem_cache_init_late() called later in start_kernel() directly.

kmem_cache_init is complicated since when called from mm_init(), the slab allocator system is not yet ready. Small data structures used here cannot be directly allocated using slab system. Several pointer/array fields in the struct kmeme_cache are particularly "annoying": `struct array_cache __percpu *cpu_cache`,  `struct kmem_cache *freelist_cache` and `struct kmem_cache_node *node[MAX_NUMNODES]`. 

`struct array_cache __percpu *cpu_cache` field will be handled by percpu allocator which we do not discuss here. `struct kmem_cache *freelist_cache` is needed for off-slab, and when we are not ready allocate freelist_cache array, we will fall back to on-slab(see calculate_slab_order and __kmem_cache_create for details). Therefore, the only annoying part during bootstrapping is `struct kmem_cache_node *node[MAX_NUMNODES]`.

The solution is firstly statically allocated variables. We need `static struct kmem_cache kmem_cache_boot`(note that `struct kmem_cache *kmem_cache` points to it) and the array `struct kmem_cache_node __initdata init_kmem_cache_node[NUM_INIT_LISTS=2 * MAX_NUMNODES]` with elements enough for nodes of two objects of struct kmem_cache. Using `struct kmem_cache *kmem_cache` and half of the elements in `init_kmem_cache_node` we could allocate another object of type `struct kmem_cache`. This new object and another half of elements in `init_kmem_cache_node` are used for allocating more objects of type `struct kmem_cache_node`. After this we are able to slab-allocate other types of objects with different sizes. This is not easy to understand here but you could try to read this again after you reach the end of talking about kmem_cache_init to get a better grasp.

We use global var `enum slab_state slab_state` to indicate whether the slab_state is DOWN, PARTIAL, PARTIAL_NODE, UP or FULL. It is used to branch some operations during this slab initialization process to circumvent some chick-&-egg problems, especially in set_up_node() function, as shown in detail below. 

---
**Step 1: point `struct kmem_cache *kmem_cache` to `static struct kmem_cache kmem_cache_boot`**

slab_state==DOWN at beginning

The var `struct kmem_cache *kmem_cache` is used for allocating variables of type struct kmem_cache variable. It is set to address of the statically allocated kmem_cache_boot.
Note that kmem_cache_boot is not of attribute __init_data and therefore is still valid after booting.

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

Note that although the kmem_cache_boot is statically allocated, its member array of `struct kmem_cache_node *node[MAX_NUMNODES]` is still not valid(the pointer elements are not properly set). They cannot be dynamically allocated by slab-allocator yet. We will see later that we temporarily set the pointers to point to the elements of `static struct kmem_cache_node __initdata init_kmem_cache_node[NUM_INIT_LISTS]`(details below).


---
**Step 2: initializes var `static struct kmem_cache_node __initdata init_kmem_cache_node[NUM_INIT_LISTS]`**

slab_state==DOWN at beginning

kmem_cache_init then initializes the init_kmem_cache_node static array using kmem_cache_node_init(which simply inits the fields of init_kmem_cache_node element):

kmem_cache_init(void): _step_2_
```c
	for (i = 0; i < NUM_INIT_LISTS; i++)
		kmem_cache_node_init(&init_kmem_cache_node[i]);
```

```c
#define NUM_INIT_LISTS (2 * MAX_NUMNODES)
static struct kmem_cache_node __initdata init_kmem_cache_node[NUM_INIT_LISTS];
```

```c
static void kmem_cache_node_init(struct kmem_cache_node *parent)
{
	INIT_LIST_HEAD(&parent->slabs_full);
	INIT_LIST_HEAD(&parent->slabs_partial);
	INIT_LIST_HEAD(&parent->slabs_free);
	parent->total_slabs = 0;
	parent->free_slabs = 0;
	parent->shared = NULL;
	parent->alien = NULL;
	parent->colour_next = 0;
	spin_lock_init(&parent->list_lock);
	parent->free_objects = 0;
	parent->free_touched = 0;
}
```
Note that each kmem_cache needs MAX_NUMNODES of kmem_cache_node.  init_kmem_cache_node[2 * MAX_NUMNODES] are therefore used for two slab caches, the `struct kmem_cache *kmem_cache` for allocating objs of `struct kmem_cache` and the another object of type `struct kmem_cache` for allocating objs of `struct kmem_cache_node`. We will see it later how this is utilized in set_up_node().




---
**Step 3: initialize var `struct kmem_cache *kmem_cache`, put it in slab_caches**

slab_state==DOWN at beginning

kmem_cache_init then further inits `struct kmem_cache *kmem_cache` variable. This cache is used for allocating vars of type struct kmem_cache. 

`offsetof(struct kmem_cache, node) +
nr_node_ids * sizeof(struct kmem_cache_node *)` is used for calculating the size of struct kmem_cache. Note that the struct kmem_cache's last field(`struct kmem_cache_node *node[MAX_NUMNODES]`) does not need to be fully of size MAX_NUMNODES and can be shrunk according to the real number of nodes(nr_node_ids). The alignment requirements of allocating struct kmem_cache is required to be hardware cache alignment(SLAB_HWCACHE_ALIGN). kmem_cache is then put on the list of slab_caches. 



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

Here we give some more details about the above part. The calling routine sequence going deeper is kmem_cache_init() -> create_boot_cache() -> __kmem_cache_create(). 

part of __kmem_cache_create() before exiting(in its "done:" part) is
```c
	if (OFF_SLAB(cachep)) {
		cachep->freelist_cache =
			kmalloc_slab(cachep->freelist_size, 0u);
	}

	err = setup_cpu_cache(cachep, gfp);
	if (err) {
		__kmem_cache_release(cachep);
		return err;
	}
```

Note that in the __kmem_cache_create(see details in entry for `__kmem_cache_create`), in this particular moment, we will not have an off-slab cache. That is because that even if we fall into the case in the call sequence of _kmem_cache_create->set_off_slab_cache->calculate_slab_order->kmalloc_slab, our entry in the kmalloc_caches[KMALLOC_NORMAL][] will probably be null since they are not set. Therefore, for this case, we do NOT need to and do NOT set the kmem_cache->freelist_size using kmalloc_slab from __kmem_cache_create(). 

In setup_cpu_cache, we will call set_up_node(kmem_cache, CACHE_CACHE) to point our `struct kmem_cache *kmem_cache`->node to the init_kmem_cache_node elements with indices of 0, 2, 4, .... This is temporary solution and later on in step 6, we will replace these static node to our dynamically allocated nodes. 

Remember this point here, since later on in step 4, we will see that the init_kmem_cache_node elements with indices of 1, 3, 5, 7... is used for the node field for kmem_caches[INDEX_NODE], which is used for allocating objs of type `struct kmem_cache_node`;

part of setup_cpu_cache():
```c
	if (slab_state == DOWN) {
		/* Creation of first cache (kmem_cache). */
		set_up_node(kmem_cache, CACHE_CACHE);
	} else if (slab_state == PARTIAL) {
		/* For kmem_cache_node */
		set_up_node(cachep, SIZE_NODE);
	} else {
		int node;

		for_each_online_node(node) {
			cachep->node[node] = kmalloc_node(
				sizeof(struct kmem_cache_node), gfp, node);
			BUG_ON(!cachep->node[node]);
			kmem_cache_node_init(cachep->node[node]);
		}
	}
```

Now that we have the variable kmem_cache global on-slab cache properly initialized and the slab_state is in partial. We therefore can now create objects of type `struct kmem_cache` using our global `struct kmem_cache *kmem_cache`.

From now on, the global variable of slab_state is set to PARTIAL. 

---
**Step 4:  setup kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE](which is used for allocating `struct kmem_cache_node`)**

slab_state==PARTIAL at beginning

kmem_cache_init then initializes kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE]. This element is responsible of allocating objs of type `struct kmem_cache_node`. It calls create_kmalloc_cache() for this initialization.

kmem_cache_init(void): _step_4_
```c
	/*
	 * Initialize the caches that provide memory for the  kmem_cache_node
	 * structures first.  Without this, further allocations will bug.
	 */
	kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE] = create_kmalloc_cache(
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

Inside create_kmalloc_cache, we first allocate a zeroed variable with name "s" of the type of struct kmem_cache. This is legit, since we have already initialized the global `struct kmem_cache *kmem_cache` in step 3(remember its node field was also temporarily set to the some elements in init_kmem_cache_node). It then calls create_boot_cache which we are familiar with(used before in kmem_cache_init to in _step_3_). Remind that the calling chain is create_boot_cache() -> __kmem_cache_create() -> setup_cpu_cache() -> setup_node().  The params passed to setup_node depends on the slab_state. When the slab_state==PARIAL, it will call set_up_node(cachep, SIZE_NODE). set_up_node(cachep, SIZE_NODE) sets the struct kmem_cache::node array of cachep(cachep here is the `s` inside create_kmalloc_cache as shown in above code snapet). 

Since the SIZE_NODE==1, the s->node array will be set to the init_kmem_cache_node elements with indices of 1, 3, 5, 7... Remember in step 3, init_kmem_cache_node elements with indices 0, 2, 4, 6 ... was used for the node field of `struct kmem_cache *kmem_cache`. This is also a temporary solution and in step 6, this temporary static nodes will be replaced with properly dynamically allocated ones.

Keep going with kmem_cache_init. Let's stage what we have here. We are now at a slab_state of PARTIAL_NODE. We have two objects of type struct kmem_cache, one is `kmem_cache *kmem_cache` (== &kmem_cache_boot which is static) to allocate objects of `struct kmem_cache` and the other one is kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE] used for allocate struct kmem_cache_node). These two objects have their node array fields point to the static init_kmem_cache_node which is only a temporary solution here which will be remedied in step 6.

---
**Step 5: patch the array of size_index**

slab_state = PARTIAL_NODE at beginning.

We call setup_kmalloc_cache_index_table() to patch the array of size_index. 


kmem_cache_init(void): _step_5_
```c
	setup_kmalloc_cache_index_table();
	slab_early_init = 0;
```

---
**Step 6: replace nodes of `struct kmem_cache* kmem_cache` and kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE] with dynamically allocated ones.**

slab_state = PARTIAL_NODE at beginning.

Remember that in step 4, we are ready to allocate objects of type `struct kmem_cache_node`. We therefore are going to replace the node field for `struct kmem_cache* kmem_cache` and kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE], which pointed to static init_kmem_cache_node array, with dynamically allocated objects of type kmem_cache_node. 

kmem_cache_init(void): _step_6_
```c
	{
		int nid;

		for_each_online_node(nid) {
			init_list(kmem_cache, &init_kmem_cache_node[CACHE_CACHE + nid], nid);

			init_list(kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE],
					  &init_kmem_cache_node[SIZE_NODE + nid], nid);
		}
	}
```
init_list function will allocate new kmem_cache_node objects and memcpy the corresponding init_kmem_cache_node element in. Since the init_kmem_cache_node is with attribute of __initdata, we will eventually reclaim the memory space statically allocated for init_kmem_cache_node.


---
**Step 7: create caches of different types**

slab_state = PARTIAL_NODE at beginning;

slab_state = up at end.

Finally, create_kmalloc_caches will initialize the yet uninitialized elements of the global array kmem_caches(remind that one of the elements, kmalloc_caches[KMALLOC_NORMAL][INDEX_NODE], has been initialized before).

kmem_cache_init(void): _step_7_
```c
		create_kmalloc_caches(ARCH_KMALLOC_FLAGS);
		//docu: close the end of above func, the state of slab will be set to up
```

The basic calling sequence is create_malloc_caches() -> (for each kmalloc_caches element) new_kmalloc_cache() -> create_kmalloc_cache() -> create_boot_cache() -> _kmem_cache_create() -> setup_cpu_cache(). In setup_cpu_cache, since the slab_state is now PARTIAL_NODE, we first allocate the cachep->node array and then set them using kmem_cache_node_init function.

a revisit of part of setup_cpu_cache():
```c
	if (slab_state == DOWN) {
		/* Creation of first cache (kmem_cache). */
		set_up_node(kmem_cache, CACHE_CACHE);
	} else if (slab_state == PARTIAL) {
		/* For kmem_cache_node */
		set_up_node(cachep, SIZE_NODE);
	} else {
		int node;

		for_each_online_node(node) {
			cachep->node[node] = kmalloc_node(
				sizeof(struct kmem_cache_node), gfp, node);
			BUG_ON(!cachep->node[node]);
			kmem_cache_node_init(cachep->node[node]);
		}
	}
```

create_kmalloc_caches also sets the slab_state to UP. We are almost done now.

---
The analysis of kmem_cache_init is done. Note that this function is called inside the start_kernel->mm_init(). 

Inside start_kernel(), later, we will call kmem_cache_init_late() which enables the cpu cache and sets slab_state to FULL. After that step, the slab allocator is fully set up.














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
`slabs_partial`, `slab_full` and `slabs_free` each holds a list of pages. Each page donates one slab (this page is the start page of the slab) that contains a certain order of pages(order is saved in struct kmem_cache->gfporder). Slab information is saved in unions in struct page simply to avoid to waste memory. Since kmem_cache_node is the owner of the struct page when they are slabs_partial, slabs_full and slab_free, the field of `struct list_head lru` is used for list_head in this list(struct page->lru can be used as a generic list by the page owner).

