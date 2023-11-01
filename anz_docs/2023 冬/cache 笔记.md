## tips
1. cache包含tag，index，BlockOffset，是由index来寻找放在set的哪个地方的，存储一个数据可以放在多个cache line里，可以放在几个里就是有几个set
2. cache 分bank操作的原因是cache需要多端口读写，但是SRAM 多端口功耗延迟太大，所以变成分bank操作了，比如分为奇偶两个bank。
![[Pasted image 20231101172408.png|1200]]
