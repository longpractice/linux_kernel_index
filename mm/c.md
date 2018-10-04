# C

## contig_page_data

the variable representing the single global node for UMA systems. It is of type `struct pglist_data` which is the type of a node

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