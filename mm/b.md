# b

## batch size
batch size is the unit of filling in per cpu page set for a certain zone, it is normally one fourth of the 1000th of total number of pages contained by the zone

## BIOS runtime services
n a PC, booting Linux begins in the BIOS at address 0xFFFF0. The first step of the BIOS is the power-on self test (POST). The job of the POST is to perform a check of the hardware. The
second step of the BIOS is local device enumeration and initialization. Given the different uses of BIOS functions, the BIOS is made up of two parts: the POST code and runtime services. After the POST is complete, it is flushed from memory, but the BIOS runtime services remain and are available to the target operating system.

To boot an operating system, the BIOS runtime searches for devices that are both active and bootable in the order of preference defined by the CMOS settings. A boot device can be a floppy disk, a CD-ROM, a partition on a hard disk, a device on the network, or a USB stick.

Commonly, Linux is booted from a hard disk, where the Master Boot Record(MBR) contains the primary boot loader. The MBR is a 512-byte sector, located in the first sector on the disk. After the MBR is loaded into the RAM, the BIOS yields control to it.

## boot_e820_entry
```c
/*
 * The E820 memory region entry of the boot protocol ABI:
 */
struct boot_e820_entry {
	__u64 addr;
	__u64 size;
	__u32 type;
} __attribute__((packed));
```
the type of e820 entry set up by boot memory allocator. It will be copied to e820_entry of e820_table.

## boot loader
the next step of booting after the BIOS runtime services. There are two stages of it, the first, primary boot loader is used for loading the second stage boot loader.

## boot loader(stage 1)
The primary boot loader that resides in the MBR is a 512-byte image containing both program code and a small partition table. The first 446 bytes are the primary boot loader, which contains both executable code and error message text. The next sixty-four bytes are the partition table, which contains a record for each of four partitions (sixteen bytes each). The MBR ends with two bytes that are defined as the magic number (0xAA55). The magic number serves as the validation check of the MBR.
```
To see the contents of your MBR, use this command:
dd if=/dev/hda of=mbr.bin bs=512 count=1
od -xa mbr.bin
The dd command, which needs to be run from root, reads the first 512 bytes from /dev/hda (the first Integrated Drive Electronics, or IDE drive) and writes them to the mbr.bin file.
The od command prints the binary file in hex and ASCII formats.
```
The job of the primary boot loader is to find and load the secondary boot loader. It does this by looking through the partition table for an active partition. When it finds an active partition, it scans the remaining partitions in the table to ensure that they are all inactive. When this is verified, the active partition's boot record is read from the device into RAM and executed.


## boot loader(stage 2)
The secondary, or second-stage, boot loader could be more aptly called the kernel loader. The task at this stage is to load the Linuxe kernel and optional initial RAM disk.

The first and second stage boot loaders combind are called Linuxe Loader(LILO) or GRand Unified Bootloader (GRUB) in the x86 PC environment. LILO has some disadvantages that were corrected in GRUB.

The great thing about GRUB is that it includes knowledge of Linux file systems. Instead of using raw sectors on the disk, as LILO does, GRUB can load a Linux kernel from an ext2 or ext3 file system. It does this by making the two-stage boot loader into a three-stage boot loader. Stage 1(MBR) boots a stage 1.5 boot loader that understands the particular file system containing the Linux kernel image. Examples include reiserfs_stage1_5(to load from a Reiser journaling file system) or e2fs_stage1_5(to load from an ext2 or ext3 file system). When the stage 1.5 boot loader is loaded and running, the stage 2 boot loader can be loaded.


## bootmem_data, bootmem_data_t
per node information used by the bootmem allocator
```c
/**
 * struct bootmem_data - per-node information used by the bootmem allocator
 * @node_min_pfn: the starting physical address of the node's memory
 * @node_low_pfn: the end physical address of the directly addressable memory
 * @node_bootmem_map: is a bitmap pointer - the bits represent all physical
 *		      memory pages (including holes) on the node.
 * @last_end_off: the offset within the page of the end of the last allocation;
 *                if 0, the page used is full
 * @hint_idx: the PFN of the page used with the last allocation;
 *            together with using this with the @last_end_offset field,
 *            a test can be made to see if allocations can be merged
 *            with the page used for the last allocation rather than
 *            using up a full new page.
 * @list: list entry in the linked list ordered by the memory addresses
 */
typedef struct bootmem_data {
	unsigned long node_min_pfn;
	unsigned long node_low_pfn;
	void *node_bootmem_map;
	unsigned long last_end_off;
	unsigned long hint_idx;
	struct list_head list;
} bootmem_data_t;
```

`node_min_pfn` is the start physical address of the node's memory, note that in early kernel, it used to be called node_boot_start.

`node_low_pfn` is the end physical address of the directly addressable memory, it is the end of ZONE_NORMAL, bootmem allocation happens only in ZONE_DMA and ZONE_NORMAL

`node_bootmem_map` is a pointer to virtual memory of the memory area in which the allocation bitmap is stored. On IA-32 systems, the memory area immediately following the kernel image is used for this purpose. The corresponding address is held in the _end variable, which is automatically patched into the kernel image during linking.

`last_end_off` the offset within the page of the end of the last allocation; if 0, the page used is full. It used to be call `last_pos` in older kernels.

`hint_idx` the PFN of the page used with the last allocation; together with using this with the `last_end_offset` field, a test can be made to see if allocations can be merged with the page used for the last allocation rather than using up a full new page.

`list` system with discontinuous memory can require more than one bootmem allocator. This is typically the case on NUMA machines that register one bootmem allocator per node, but it would, for instance, also be possible to register one bootmem allocator for reach continuous memory region on systems where the physical address space is interspersed with holes. In addition to having the ability to store and boot a Linux image, these boot monitors perform some level of system test and hardware initialization. In an embedded target, these boot monitors commonly cover both the first and second stage boot loader.



## bootmem_node_data
a global array holding all the `bootmem_data_t` for all the nodes. For UMA, there is only one element in this array, and is assigned to contig_page_data.bdata.

## boot monitor
Normally on an embedded platform, a bootstrap environment is used when the system is powered on, or reset. Examples include U-Boot RedBoot, and MicroMonitor from Lucent. Embedded platforms are commonly shipped with a boot monitor. These programs reside in special region of flash memory on the target hardware and provide the means to download a Linux kernel image into flash memory and subsequently execute it. 

## build_all_zonelists
used in start_kernel(), called after setup_arch() or used in memory hot plug.

if the system state is not in SYSTEM_BOOTING(eg, memory hotplug), by calling __build_all_zonelist(), it sets up the fallback list of zone data structures. It determines the order that kernel allocates memory and frees memory. Generally speaking the highmem should be firstly allocated and then the normal memory and eventually the DMA part.

if the system state is in SYSTEM_BOOTING, it will first call build_all_zonelists_init(). build_all_zonelists_init then first calls __build_all_zonelists(NULL) and then it initializes the boot_pagesets for the per cpu pageset.

```c
/*
 * Initialize the boot_pagesets that are going to be used
 * for bootstrapping processors. The real pagesets for
 * each zone will be allocated later when the per cpu
 * allocator is available.
 *
 * boot_pagesets are used also for bootstrapping offline
 * cpus if the system is already booted because the pagesets
 * are needed to initialize allocators on a specific cpu too.
 * F.e. the percpu allocator needs the page allocator which
 * needs the percpu allocator in order to allocate its pagesets
 * (a chicken-egg dilemma).
 */
for_each_possible_cpu(cpu)
    setup_pageset(&per_cpu(boot_pageset, cpu), 0);
```

Note that later in start_kernel(), setup_per_cpu_pageset() will be called to do some final pageset initialization.

## build_all_zonelists_init(void)
called by build_all_zonelists() if the system state is in SYSTEM_BOOTING. 

```c
static noinline void __init
build_all_zonelists_init(void)
{
	int cpu;

	__build_all_zonelists(NULL);

	/*
	 * Initialize the boot_pagesets that are going to be used
	 * for bootstrapping processors. The real pagesets for
	 * each zone will be allocated later when the per cpu
	 * allocator is available.
	 *
	 * boot_pagesets are used also for bootstrapping offline
	 * cpus if the system is already booted because the pagesets
	 * are needed to initialize allocators on a specific cpu too.
	 * F.e. the percpu allocator needs the page allocator which
	 * needs the percpu allocator in order to allocate its pagesets
	 * (a chicken-egg dilemma).
	 */
	for_each_possible_cpu(cpu)
		setup_pageset(&per_cpu(boot_pageset, cpu), 0);

	mminit_verify_zonelist();
	cpuset_init_current_mems_allowed();
}

```

## __build_all_zonelists()
it sets up the node and zone data structures. It determines the order that kernel allocates memory and frees memory. Generally speaking the highmem should be firstly allocated and then the normal memory and eventually the DMA part.

```c
static void __build_all_zonelists(void *data)
{
	int nid;
	int __maybe_unused cpu;
	pg_data_t *self = data;
	static DEFINE_SPINLOCK(lock);

	spin_lock(&lock);
	/*
	 * This node is hotadded and no memory is yet present.   So just
	 * building zonelists is fine - no need to touch other nodes.
	 */
	if (self && !node_online(self->node_id)) 
	{
		build_zonelists(self);
	} 
	else 
	{

		for_each_online_node(nid) 
		{
			pg_data_t *pgdat = NODE_DATA(nid);
			build_zonelists(pgdat);
		}
	}

	spin_unlock(&lock);
}

```
when used under SYSTEM_BOOTING status, for UMA, data and self will be NULL, the code block
```c
	for_each_online_node(nid) 
	{
		pg_data_t *pgdat = NODE_DATA(nid);
		build_zonelists(pgdat);
	}
```
executes. NODE_DATA is just &page_contig_data. The core task therefore delegates to build_zonelist(nid).

## build_zonelists(pg_data_t *pgdat) 
In UMA situation, used inside __build_all_zonelists. build zonelists ordered by zone and nodes within zones. This results conserving DMA zones until all Normal memory is exhausted, but results in overflowing to remote node while memory may still exist in local DMA zone.

Say current node is ith node. It first add all the zones on the current node(ith), then add all the zones from i+1 to  MAX_NUMNODES and then adds the zones from 0 to i-1. So it is added in a round-robin manner.

## build_zonerefs_node(pg_data_t *pgdat, struct zoneref *zonerefs)
used inside build_zonelists(). Fill in zonerefs[] with all zones in fallback order in pgdat, only zones with managed_pages are added(using function managed_zone()). Returns the number of zones added.