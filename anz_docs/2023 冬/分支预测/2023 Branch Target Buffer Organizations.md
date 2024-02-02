### Background
**direct cost**：跳转地址直接来自于指令，但是如果没有BTB，需要取值，译码，执行之后才会知道地址。
**opportunity cost**：只有在superscalar处理器中会出现，因为一次取N个指令，所以一个指令包里的跳转指令如果处于指令包中间的位置，那么之后的指令取出来其实都是没用的
**未执行的分支不用在BTB里面存储**
在<u>Glenn Reinman, Brad Calder, and Todd Austin. 1999. Fetch Directed Instruction
Prefetching. In Proceedings of the International Symposium on Microarchitecture.
16–27.</u>
这篇文章中提出了FTQ，所以icache的停止不会造成整个fetch的停止，BPU可以继续运行

### 对比R-BTB B-BTB
