### RV64FD 
总图
![[Pasted image 20231214144008.png]]
共有 10 条指令

| 指令  | 备注 | 说明|
| --- | --- | ----- |
| fmv.d.x | only RV64D | 寄存器X[rs1]的双精度浮点数复制到f[rd] |
| fmv.x.d| only RV64D| 寄存器X[rs1]的单精度浮点数复制到x[rd]|
| fcvt.d.l| only RV64D| 寄存器X[rs1]的64位二进制补码表示的整数转化为双精度浮点数，再写入f[rd]|
| fcvt.d.lu| only RV64D| 寄存器X[rs1]的64位无符号整数转化为双精度浮点数，再写入f[rd]|
| fcvt.s.l| only RV64F| 寄存器X[rs1]的64位二进制补码表示的整数转化为单精度浮点数，再写入f[rd]|
| fcvt.s.lu| only RV64F| 寄存器X[rs1]的64位无符号整数转化为单精度浮点数，再写入f[rd]|
| fcvt.l.d| only RV64D| 寄存器f[rs1]中的双精度浮点数转化为64位二进制补码表示的整数，写入x[rd]|
| fcvt.l.s| only RV64F| 寄存器f[rs1]中的单精度浮点数转化为64位二进制补码表示的整数，写入x[rd]|
| fcvt.lu.d| only RV64D| 寄存器f[rs1]中的双精度浮点数转化为64位无符号整数，写入x[rd]|
| fcvt.lu.s| only RV64F| 寄存器f[rs1]中的单精度浮点数转化为64位无符号整数，写入x[rd]|


### RV64C
总图
![[Pasted image 20231214151615.png]]
共涉及 11 条指令

| 指令  | 备注 | 说明|
| --- | --- | ----- |
| c.addiw | only RV64IC | 加立即数字，rd=x[0]非法，x[rd]=sext((x[rd]+sext(imm))[31:0]) |
| c.subw | only RV64IC | 减字，x[8+rd']=sext((x[8+rd']-x[8+rs2'])[31:0]) |
| c.flw | only RV32FC | 删除 |
| c.flwsp | only RV32FC | 删除 |
| c.fsw | only RV32FC | 删除 |
| c.fswsp | only RV32FC | 删除 |
| c.ld | only RV64IC | 双字加载，x[8+rd']=M[x[8+rs1']+uimm][63:0] |
| c.ldsp | only RV64IC | 栈指针相关双字加载，x[rd]=M[x[2]+uimm][63:0] |
| c.sd | only RV64IC | 双字存储，M[x[8+rs1']+uimm][63:0]=x[8+rs2'] |
| c.sdsp | only RV64IC | 栈指针相关双字存储，M[x[2]+uimm][63:0]=x[rs2] |
| c.jal | only RV32IC | 删除 |


