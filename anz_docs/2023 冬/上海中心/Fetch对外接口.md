### fetch.v
| 信号名 | 备注(没有备注的一律为扩为64) |
| ---- | ---- |
| pipe_run_addr | 扩为80 |
| miack_pc等中断异常相关pc | 扩为64 |
| vextpt_br_pc |  |
| A0_pre_nobr_next_pc_e1 |  |
| A1_pre_nobr_next_pc_e1 |  |
| D0_pre_nobr_next_pc_e1 |  |
| DIV_pre_nobr_next_pc_e1 |  |
| F_pre_nobr_next_pc_e1 |  |
| pre_br_aim_pc_e1 |  |
| d_br_aim_pc |  |
| d_br_ins_pc |  |
| prgm_addr | 扩为80 |
| ins0_pre_br_pc |  |
| ins0_pc |  |
| ins1_pre_br_pc |  |
| ins1_pc |  |
|  |  |

除总fetch.v接口外，一律写的简单易懂一点
### fetch_pre.v
Fetch_pg主要用于将分支预测模块预测出来的pc以及中断对应csr改写的pc进行选择，决定下一条指令包应当取指的地址是什么，同时考虑到后续I buffer的容量，用于决定是否卡住取指请求

| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
| pg_next | 决定是否要继续预测 |  |
| fetch_pc_sel_temp | 如果没有中断和预测，则顺序取值+16 |  |
| exp_done | 来自decode，是否译码了一组128bit指令包 |  |

| 信号名 | 备注 |
| ---- | ---- |
| pipe_run_addr | 扩为80 |
| pre_br_pc | 扩为64*8 |
| pre_br_pc_sel |  |
| miack_pc等中断异常pc |  |
| pc_pre |  |
| progm_addr | 扩为80 |
| pre_br_pc_pg | 扩为64*8 |
| pc_pg |  |
|  |  |

### fetch_pre：分支预测模块整合
**分支预测部分大部分只写了功能理解**

| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
| d_br_ins | 实际是不是分支 |  |
| pre_br_ins_e1 | 当时预测是不是分支 |  |
| d_br_en | 实际是否跳转 |  |
| pre_br_en_e1 | 当时预测是否跳转 |  |
|  |  |  |
pre_br_ins预测是不是分支，pre_br_en预测跳不跳转，

### btb_count:分支预测器
| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
| pc_pre | 当前pc |  |
| pg_next | 现在是不是需要分支预测 |  |
| pc_mem_tmp | btb输入 |  |
| br_ins_pc_clip | 后端输入的pc |  |
| pre_br_ins_tmp | 定值 |  |

![[Pasted image 20240104164937.png]]
相当于输入pc立马有预测跳转，一拍输出是否跳转
用0到11位，最低一位没有用，所以实际是1到11位，2048个饱和计数器

只更改了输入 20和25行

### btb_table：BTB存放的位置
| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
| d_br_ins | 是不是有实际的分支 |  |
| db | 跳转到的目的地址 |  |
| web | 写使能 |  |
| addrb | 后端跳转的起始地址 |  |
| addra | 此时的pc，读地址 |  |
| cea | 读使能 | 一直为1，不明白 |
|  |  |  |

SRAM里面存放的data是aim_pc就是跳转的地址，addra是8位的，用跳转前pc的4到11位来寻址，写使能是看有没有分支，所以SRAM一个元素大小32位，共有2的8次方个这么多元素
cpl是关于特权架构的，不用关心
bypass是如果此时pc和后端传过来的pc一样那就bypass
改成64位后BTB没有扩容，aliasing只会更严重

修改了 19 21 22 25-33 51 52 60-67

### br_ins_buf：记录是否是call return 记录是不是分支
| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
| br_mem_en | 写使能 | 如果两条后端的指令只要有call or return or分支，就拉高 |
| pre_br_ins | 判断是不是分支 |  |

要判断8条指令是否是call or return or 分支 一共有2048个entry
写这个buffer需要的是后端的信息，decode完两条指令里是否包含call和return，读是前端的信息
发生重名的话就不好判断，因为只记录了是不是分支，并没有做预测，就像分支预测，只记录一个比特一样
tmp是一输入就输出，没有tmp的是打一拍输出
在这里面，只要曾经是分支的地址打进来就会判断他是分支，没有其他的更新逻辑，会有aliasing的问题，即pre_br_ins为1不代表他一定是分支

### fetch_ras：就是RAS
| 信号名 | 作用 | 备注 |
| ---- | ---- | ---- |
|  |  |  |

深度为16，分预测的ptr和实际的ptr,实际的ptr由decode出来的push和pop掌握，预测的由预测的掌握
push只由decode出来的掌握，pop分pre和实际的，所以说，如果call的话地址由BTB预测，只有RET才由RAS预测
当预测错误的ptr和实际的对不上的时候，预测的回归实际的

### fetch_ir
| 信号名 | 作用 |
| ---- | ---- |
| pre_br_pc_pg | 改为64*8 |
| pc_pg |  |
| fp0_pre_br_pc | 改为64*8 |
| fp0_pc |  |
| fp1_pre_br_pc | 改为64*8 |
| fp1_pc |  |
|  |  |
fetch ir只是相当于一个3bit的FIFO，FIFO的更新逻辑没变，只是内部对应的fetch_set各个信号的位置变了，如下
![[Pasted image 20240112152727.png]]

### fetch_ins
| 信号名 | 作用 |
| ---- | ---- |
| fp0_pre_br_pc | 改为64*8 |
| fp0_pc |  |
| fp1_pre_br_pc | 改为64*8 |
| fp1_pc |  |
| ins0_pre_br_pc |  |
| ins0_pc |  |
| ins1_pre_be_pc |  |
| ins1_pc |  |
|  |  |
fetch_ins做指令切分，因为位宽改变而导致部分信号位置改变
![[Pasted image 20240112153051.png]]
