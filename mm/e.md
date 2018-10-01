# e

## _edata
the end of data section of the kernel binary. The data section starts from _etext, the end of the text section. It is also the start of data section that is no longer used after kernel initialization.

## _end
the end of the kernel executable binary in physical memory. It is also the end of the data section that is not needed after kernel initialization.

## _etext
end of the text section of contains the kernel code. It is also the start of the data section of the kernel binary loaded in memory