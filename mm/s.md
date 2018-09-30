# S

## set_pgd
Must be implemented by all architectures.
Set the value of a page global directory

## set_pud
Must be implemented by all architectures.
set the value of a page upper directory

## set_pmd
Must be implemented by all architectures.
set the value of a page middle directory

## set_pte
Must be implemented by all architectures.
set ent value of a page table entry

## start_kernel
global starte routine that is executed after kernel loading to start various subsystems

## setup_arch
routine used in start_kernel(). Contains architecture specific code that is responsible for boot-time initializations.
Related to memory management is that it initialize the boot allocator.

## setup_per_cpu_areas
routine used in start_kernel(). Contains architecture specific code. For SMP systems, it initialize per-CPU variables defined statically in the source code and of which there is a seperate copy for each CPU in the system. Variables of this kind are stored in a seperate section of the kernel binaries. The purpose of setup_per_cpu_areas is to create a copy of these data for each system CPU. For non-SMP systems, it does nothing.

## setup_per_cpu_pageset
routine used in start_kernel(). Set up the pageset arrays from each struct zone.