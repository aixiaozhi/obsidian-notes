指令包大小：128bit 最多8指令，最少4指令
## fetch_pre
### btb_count
| 信号名  | 作用 | 备注 |
| --- | --- | ----- |
| pc_pre | 当前pc |  |
| pg_next| 现在是不是需要分支预测| |
| pc_mem_tmp | btb输入 |  |
| br_ins_pc_clip | 后端输入的pc |  |
| pre_br_ins_tmp | 定值 |  |

相当于输入pc立马有预测跳转，一拍输出是否跳转