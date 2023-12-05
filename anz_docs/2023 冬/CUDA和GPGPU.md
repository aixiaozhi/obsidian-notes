## 2023/11/21
CUDA编程可以直接指定线程块的数量和块的大小 
SIMD线程代指一个warp，硬件创建、管理、调度、执行的机器对象是一个warp，是一个只包含SIMD指令的传统线程，一个warp一个pc，运行在多线程SIMD处理器上

idea:频繁被替换的dcache或者icache，直接不通过l1cache，直接bypass?