## r

## reserve_bootmem
arch-irrelavant code to mark none-free pages in boot allocator, arch-specific code will call this code to mark according to its only need. No longer used by x86 though.

## required_kernelcore
the administrator could specify this value for the non-movable memory. The non-movable memory is spread evenly across the different nodes. 

## required_movablecore
the administrator could specify the value for the movable memory. Note that the non-movable memory is spread evenly across the different nodes and the movable memory is only from highest zone(in 32bit systems, ZONE_HIGHMEM, for 64bit systems, ZONE_NORMAL or ZONE_DMA32).