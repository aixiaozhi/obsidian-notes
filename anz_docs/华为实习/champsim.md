DDR频率是3200Mhz，就是每秒3200M次，DDR是上升沿下降沿都传输，所以是6400MTPS
![[Pasted image 20240717185816.png]]
champsim只有在load or 下层prefetch的时候激活预取

![[Pasted image 20240717185926.png|400]]
改成no_instr，要不会报错
