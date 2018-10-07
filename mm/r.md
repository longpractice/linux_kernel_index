## 
reserve_bootmem
arch-irrelavant code to mark none-free pages in boot allocator, arch-specific code will call this code to mark according to its only need. No longer used by x86 though.