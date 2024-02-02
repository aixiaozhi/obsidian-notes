### 步骤一 打开dump文件
在/eda/project/uvm_cases/coremark_o3_west下
打开 coremark.dump 搜索iterations
看到 addi s1 s1 1，这个是循环次数的跳变 s1寄存器
pc为9ff91c50
### 步骤二 看verdi找
在robQ.v文件里找到rob_retire1_ins_pc，rob_retire2_ins_pc，rob_retire3_ins_pc在这三个里面搜索对应的pc，然后找s1从7跳变到8，算cycle

### 步骤三 算跑分
4x10^5 为 2.5分
目前为334287 跑分为 2.99


