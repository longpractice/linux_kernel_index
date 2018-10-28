# t

## _text
the start physical address of the text section that contains the compiled kernel code

## trim_bios_range()

```c
static void __init trim_bios_range(void)
{
	/*
	 * A special case is the first 4Kb of memory;
	 * This is a BIOS owned area, not kernel ram, but generally
	 * not listed as such in the E820 table.
	 *
	 * This typically reserves additional memory (64KiB by default)
	 * since some BIOSes are known to corrupt low memory.  See the
	 * Kconfig help text for X86_RESERVE_LOW.
	 */
	e820__range_update(0, PAGE_SIZE, E820_TYPE_RAM, E820_TYPE_RESERVED);

	/*
	 * special case: Some BIOSen report the PC BIOS
	 * area (640->1Mb) as ram even though it is not.
	 * take them out.
	 */
	e820__range_remove(BIOS_BEGIN, BIOS_END - BIOS_BEGIN, E820_TYPE_RAM, 1);

	e820__update_table(e820_table);
}
```

The routine makes sure that the first physical page is classified as E820_TYPE_RESERVED and take 640-1Mb range out of E820 table.

## trim_low_memory_range
reserve memory in memblock as configured by CONFIG_X86_RESERVE_LOW

## trim_platform_memory_ranges
reserve memory that causes trouble in certain devices. It calls here trim_snb_memory().

## trim_snb_memory
reserve memory due to problem in Sandy Bridge graphics(Intel HD Graphics). Some bad pages are to be avoided only when snb_gfx_workaround_needed returns true. snb_gfx_workaround only returns true only under some conditions when CONFIG_PCI is set. 
