# x

## x86_init

```c
/*
 * The platform setup functions are preset with the default functions
 * for standard PC hardware.
 */
struct x86_init_ops x86_init __initdata
```
In x86_init.c.
This struct holds many sub structs holding function pointers. This oop feature groups init functions together and make it easily configurable.

I take some important parts useful for memory management.

```c
.resources = {
    .probe_roms = probe_roms,
    .reserve_resources = reserve_standard_io_resources,
    .memory_setup = e820__memory_setup_default,
}

.paging = {
    .pagetable_init		= native_pagetable_init,
},

```