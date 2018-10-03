# e

## e820_table

```
/*
 * The whole array of E820 entries:
 */
struct e820_table {
	__u32 nr_entries;
	struct e820_entry entries[E820_MAX_ENTRIES];
};
```

## e820__memory_setup(void)

in x86 architecture, routine invoked inside setup_arch() which is inside start_kernel(). It creates a list with the memory regions occupied by the system and the free memory regions. It will call the e820__memory_setup_default() by default. It will also print out the memory information.

## e820__memory_setup_default(void)

int x86 architecture, the default function called by e820__memory_setup(). Pass the firmware (bootloader) E820 map to the kernel and process it. It pops the variable of e820_table.

## _edata
the end of data section of the kernel binary. The data section starts from _etext, the end of the text section. It is also the start of data section that is no longer used after kernel initialization.

## _end
the end of the kernel executable binary in physical memory. It is also the end of the data section that is not needed after kernel initialization.

## _etext
end of the text section of contains the kernel code. It is also the start of the data section of the kernel binary loaded in memory