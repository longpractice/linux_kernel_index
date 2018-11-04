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

`
