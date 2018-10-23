# C

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