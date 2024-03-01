```
|-- designs
|   |-- sky130hd    
|   |   |-- ibex   
|   |   `-- pdk   
|   `-- src #RTL文件目录
|       `-- ibex
|-- result #flow结果存放目录
|-- Dummy
`-- scripts #flow脚本存放目录
    |-- extractRC
    |-- fml
    |-- ir_v
    |-- lib2db
    |-- pr
    |-- pv
    |-- sta
    `-- syn
```

## running step

	- 可以根据需要设置make run_all
	- make chip_done/v2lvs后的脚本待更新（make run_all外的脚本暂未用到）

### make syn
	（完成RTL2Netlist，并插入扫描链。检查setup时序结果）

	- 读入timing db
	- 读入RTL
	- 读入SDC
	- 设置Path Group
	- 设置dont_use cell
	- 进行综合
	- 插入scanchain
	- 进行增量综合
	- 输出网表和报告

### make fml_syn
	（综合之后的网表通过Formal验证）

	- 读入timing db
	- 读入svf：记录了综合过程对网表的改变
	- 读入reference design：读入综合前的网表
	- 读入implement design：读入综合后的网表
	- 将scan cell的使能端无效
	- match：进行match并打印match相关报告
	- verify：进行verify并打印verify相关报告
	- 输出形式化验证相关报告


### make data_init
	（完成进行PnR前的数据导入）

	- 指定design的PG(power&groud) net名
	- 指定综合之后的网表路径
	- 指定top design名
	- 指定lef文件
	- 指定mmmc文件
	- 初始化design
	- 检查设计导入


### make floorplan
	（完成芯片面积的设定与io的摆放）

	- 定义block的die area和core area
	- 摆放block port：读入io.file
	- 设置dont use cell
	- 保存当前设计


### make power_plan
	（完成对设计的电源网络规划,并完成connectivity检查）

	- 电源地逻辑连接
	- 创建P/G Stripe
	- 连接stdcell Follow pin
	- 检查电源地连接

### make placement
	（完成stadcell的placement,Congestion情况合理,并输出了powerplan.enc）

	- 读入scan.def：在综合步骤输出的
	- 设置path group
	- 设置Place Mode：设置布局引擎的模式
	- 设置Global routing Mode
	- 进行Place
	- 输出Place后的Congestion和Timing 报告

### make cts
	（完成时钟树的综合）

	- 设置ccopt时用到的stdcell：pdk config.mk中配置
	- 将step 1相关的stdcell dont use 状态设置为false
	- 创建CTS的routing non-default rule
	- 建立时钟树：在pdk config.mk中设置NanoRoute可以使用的metal范围
	- 进行ccopt
	- 进行post cts后的优化

### make routing
	（完成block的routing，时序满足要求，完成电源地的检查和drc的检查）

	- 设置NanoRouteMode
	- 进行Routing
	- 输出Routing后的时序报告
	- 进行Routing后的优化
	- 检查电源地连接
	- 检查drc违例

### make routing_opt


### make chip_done
	（输出设计的.def,.v,.gdsii文件）

	- 处理routing后的网表，删除空的Module与悬空的net
	- 输出routing后的def
	- 输出routing后的网表
	- 输出lvs所用标记PG net text的脚本和static ir所用的P/G location脚本
	- 输出lvs 和 static ir所用hcell list
	- 输出GDS：pdk config.mk中指定使用的gds_map File

### make v2lvs

	- 将innovus生成的netlist转为spi用于LVS
	- (打开.spi文本文件) 生成spi中的4个cornercell删掉

### 其他make指令
	- （此次后端暂未用到，有些tcl也未用到）

- make drc

		 (利用calibre对输出的版图进行drc)

		- merge stdcell和top design的gds
		- 执行drc验证
		- 输出result db
		- 读入result并查看违例点

-  make lvs

		(利用calibre对输出的版图进行lvs)

		- 将输出的gds打上P/G net的text
		- 将routing后的网表转化成.spi网表
		- 将gds转成.spi网表
		- 进行Lvs验证

- make extract_rc

		（利用StarRC抽取routing之后的网表spef）

		- 指定routing之后的def文件
		- 指定lef文件
		- 指定mapfile文件
		- 指定nxtgrd文件
		- 指定抽取spef时相关变量

- make run_opt

		(利用prime time对routing之后网表进行时序检查)
	
		- 设置opt工具的相关变量
		- 读入timing lib
		- 读入routing之后的网表
		- 读入SDC
		- 读入spef
		- 输出反标报告
		- 设置path group
		- 输出Timing报告

- make static_ir

		(利用Voltus完成对design的静态功耗与静态ir drop的分析)

		- 读入routing之后的保存的.enc.dat
		- 生成PGV
		- 读入spef
		- 设置Static power analysis mode
		- 进行Static power analysis
		- 设置Static Rail analysis mode
		- 设置rail analysis mode
		- 读入电源地location文件
		- 读入static power analysis输出的.ptiavg文件
		- 进行static rail分析


## Makefile

```tcl
#Design config
DESIGN_CONFIG ?= ./designs/sky130hd/ibex/config.mk
#Process config
PDK_CONFIG ?= ./designs/sky130hd/pdk/config.mk
include $(DESIGN_CONFIG)
include $(PDK_CONFIG)

#Dir variable 
export FLOW_HOME ?= .
export DESIGN_HOME  ?= $(FLOW_HOME)/designs
export SCRIPTS_DIR   ?= $(FLOW_HOME)/scripts
export RESULT_DIR ?= $(FLOW_HOME)/result
export DUMMY_DIR ?= $(FLOW_HOME)/Dummy

#Flow optin
syn: 
	cd $(RESULT_DIR); mkdir -p syn/data; \
		mkdir -p syn/log; \
		mkdir -p syn/report; \
		mkdir -p syn/work; \
		mkdir -p scanchain/data; \
		mkdir -p scanchain/report; \
		mkdir -p scanchain/log
	dc_shell -f -64 $(SCRIPTS_DIR)/syn/dc_main.tcl | tee  $(RESULT_DIR)/syn/log/syn.log

fml_syn:
	cd $(RESULT_DIR); mkdir -p fml_syn/data; \
		mkdir -p fml_syn/log; \
		mkdir -p fml_syn/report; \
		mkdir -p fml_syn/work
	fm_shell -64 -f $(SCRIPTS_DIR)/fml/fml_main.tcl \
		-work_path $(RESULT_DIR)/fml_syn/work \
		| tee $(RESULT_DIR)/fml_syn/report/fml.log

data_init:
	cd $(RESULT_DIR); mkdir -p pr/data; \
		mkdir -p pr/log; \
		mkdir -p pr/report;
	innovus -files $(SCRIPTS_DIR)/pr/init.tcl \
		-log "$(RESULT_DIR)/pr/log/init" -overwrite

floorplan:
	innovus -files $(SCRIPTS_DIR)/pr/floor_plan.tcl \
		-log "$(RESULT_DIR)/pr/log/floorplan" -overwrite

power_plan:
	innovus -files $(SCRIPTS_DIR)/pr/power_plan.tcl \
		-log "$(RESULT_DIR)/pr/log/power_plan" -overwrite

placement:
	innovus -files $(SCRIPTS_DIR)/pr/placement.tcl \
		-log "$(RESULT_DIR)/pr/log/placement" -overwrite
cts:
	innovus -files $(SCRIPTS_DIR)/pr/cts.tcl \
		-log "$(RESULT_DIR)/pr/log/cts" -overwrite

post_cts_opt:
	innovus -files $(SCRIPTS_DIR)/pr/post_cts_opt.tcl \
		-log "$(RESULT_DIR)/pr/log/post_cts_opt" -overwrite

routing:
	innovus -files $(SCRIPTS_DIR)/pr/routing.tcl \
		-log "$(RESULT_DIR)/pr/log/routing" -overwrite

routing_opt:
	innovus -files $(SCRIPTS_DIR)/pr/routing_opt.tcl \
		-log "$(RESULT_DIR)/pr/log/routing_opt" -overwrite

chip_done:
	innovus -files $(SCRIPTS_DIR)/pr/chip_done.tcl \
		-log "$(RESULT_DIR)/pr/log/chip_done" -overwrite

extract_rc:
	cd $(RESULT_DIR); mkdir -p extractRC/data; \
		mkdir -p extractRC/log; \
		mkdir -p extractRC/report
	StarXtract $(SCRIPTS_DIR)/extractRC/extractRC.cmd

drc:
	cd $(RESULT_DIR); mkdir -p pv/drc/data; \
	mkdir -p pv/drc/log; \
	mkdir -p pv/drc/report
	calibredrv $(SCRIPTS_DIR)/pv/gds_merge.tcl | tee $(RESULT_DIR)/pv/drc/log/merge.log
	calibre -hier -drc $(SCRIPTS_DIR)/pv/drc_rule | tee $(RESULT_DIR)/pv/drc/log/drc.log

lvs:	
	cd $(RESULT_DIR); mkdir -p pv/lvs/data; \
		mkdir -p pv/lvs/log; \
		mkdir -p pv/lvs/report
	calibredrv -64 $(SCRIPTS_DIR)/pv/create_text_ibex.tcl
	bash $(SCRIPTS_DIR)/pv/trans_spi.sh
	calibre -lvs \
		-hier \
		-hcell $(SCRIPTS_DIR)/pv/hcell_list $(SCRIPTS_DIR)/pv/lvs_rule \
		| tee $(RESULT_DIR)/pv/lvs/log/lvs.log
		
v2lvs:
	cd $(RESULT_DIR)/pr/data; v2lvs -v ibex_lvs.vg \
	-a \<\> \
	-s /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/cdl/sage-x_tsmc_cl018g_rvt_PG.cdl \
	-s /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Back_End/spice/tpd018nv_270a/tpd018nv_1_2.spi \
	-s /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/FILLCAP/lvs_netlist/tsmc18_decap.cdl \
	-o ibex_lvs.spi

run_pt: 
	cd $(RESULT_DIR); mkdir -p sta/data; \
		mkdir -p sta/log; \
		mkdir -p sta/report;
		mkdir -p sta/work
	pt_shell -f $(SCRIPTS_DIR)/sta/run_timing.tcl | tee $(RESULT_DIR)/sta/log/run_pt.log

static_ir:
	cd $(RESULT_DIR); rm -rf static_ir; \
		mkdir -p static_ir/data; \
		mkdir -p static_ir/log; \
		mkdir -p static_ir/report
	voltus -files $(SCRIPTS_DIR)/ir_v/run_static.tcl -log "$(RESULT_DIR)/ir_v/log/voltus"
	
add_dummy_metal:
	cd $(DUMMY_DIR)/dum_metal; calibre -drc -hier -64 -turbo Dummy_Metal_Calibre_0.18um.215a

add_dummy_via:
	cd $(DUMMY_DIR)/dum_via; calibre -drc -hier -64 -turbo C018_RedundantViaInsertion_6M.V2.6a.encrypt

add_dummy_OD:
	cd $(DUMMY_DIR)/dum_OD; calibre -drc -hier -64 -turbo Dummy_OD_PO_Calibre_0.18um.215_2a
	
run_all:
	${MAKE} syn
	${MAKE} data_init
	${MAKE} floorplan
	${MAKE} power_plan
	${MAKE} placement
	${MAKE} cts
	${MAKE} post_cts_opt
	${MAKE}	routing
	${MAKE} routing_opt
	${MAKE} chip_done
	${MAKE} v2lvs
	
```

## configure

### config.mk (ibex)

```tcl
export DESIGN_NICKNAME = ibex
export DESIGN_NAME = CPU
export PLATFORM    = sky130hd

export VERILOG_FILES = ./designs/src/$(DESIGN_NICKNAME)/defines.v \
            ......
	          ./designs/src/$(DESIGN_NICKNAME)/CPU.v \
	          
            //./designs/src/$(DESIGN_NICKNAME)/CPU_wrapper_IO.v     
            //不需要加IO进行dc综合
            
export PR_SDC_FILE   = ./designs/$(PLATFORM)/$(DESIGN_NICKNAME)/constraint_for_pr.sdc 
export SDC_FILE      = ./designs/$(PLATFORM)/$(DESIGN_NICKNAME)/constraint.sdc
export IO_FILE       = ./designs/$(PLATFORM)/$(DESIGN_NICKNAME)/io.file

```

### constraint.sdc

```tcl
current_design $::env(DESIGN_NAME)

set clk_name  core_clock
set clk_port_name clk
set clk_period 20 
set clk_io_pct 0.2
set clk_uncertainty_pct 0.35

set clk_port [get_ports $clk_port_name]

create_clock -name $clk_name -period $clk_period $clk_port

set non_clock_inputs [lsearch -inline -all -not -exact [all_inputs] $clk_port]

set_input_delay  [expr $clk_period * $clk_io_pct] -clock $clk_name $non_clock_inputs 
set_output_delay [expr $clk_period * $clk_io_pct] -clock $clk_name [all_outputs]
set_clock_uncertainty [expr $clk_period * $clk_uncertainty_pct] [all_clocks]

set_max_area 0

```

### constraint_for_pr.sdc

```tcl
current_design CPU_wrapper_IO

set clk_name  core_clk
set clk_port_name clk
set clk_period 30
set clk_io_pct 0.2
set clk_uncertainty_pct_setup 0.1
set clk_uncertainty_pct_hold 0.01

set clk_port [get_pins CPU_u/$clk_port_name]

create_clock -name $clk_name -period $clk_period $clk_port

set non_clock_inputs [all_inputs -no_clock]

set_input_delay  [expr $clk_period * $clk_io_pct] -clock $clk_name $non_clock_inputs 
set_output_delay [expr $clk_period * $clk_io_pct] -clock $clk_name [all_outputs]
set_clock_uncertainty -setup [expr $clk_period * $clk_uncertainty_pct_setup] [all_clocks]
set_clock_uncertainty -hold [expr $clk_period * $clk_uncertainty_pct_hold] [all_clocks]

#set_max_area 0
```

### io.file

```verilog
(globals
    version = 3
    io_order = default
    space = 24
)
(iopad
    (topleft
    (inst name="CornerCellW" )
    )
    (left
    (inst name="POWER_VSSIOW0" )
    (inst name="POWER_VDDIOW0" )
    (inst name="mInA00" )
    (inst name="mInA01" )
    (inst name="mInA02" )
    (inst name="mInA03" )
    (inst name="POWER_VDDW" )
    (inst name="POWER_VSSW" )
    (inst name="mInA04" )
    (inst name="mInA05" )
    (inst name="mInA06" )
    (inst name="mInA07" )
    (inst name="POWER_VDDIOW1" )
    (inst name="POWER_VSSIOW1" )
    )
    (topright
    (inst name="CornerCellN" )
    )
    (top
    (inst name="POWER_VSSION0" )
    (inst name="POWER_VDDION0")
    (inst name="mInA08" )
    (inst name="mInA09" )
    (inst name="mInA10" )
    (inst name="mInA11" )
    (inst name="POWER_VDDN" )
    (inst name="POWER_VSSN" )
    (inst name="mInA12" )
    (inst name="mOutB16" )
    (inst name="mOutB17" )
    (inst name="mOutB18" )
    (inst name="POWER_VDDION1")
    (inst name="POWER_VSSION1" )
    )
    (bottomright
    (inst name="CornerCellE" )
    )
    (right
    (inst name="POWER_VSSIOE0" )
    (inst name="POWER_VDDIOE0" )
    (inst name="mOutB00" )
    (inst name="mOutB01" )
    (inst name="mOutB02" )
    (inst name="mOutB03" )
    (inst name="POWER_VDDE" )
    (inst name="POWER_VSSE" )
    (inst name="mOutB04" )
    (inst name="mOutB05" )
    (inst name="mOutB06" )
    (inst name="mOutB07" )
    (inst name="POWER_VDDIOE1" )
    (inst name="POWER_VSSIOE1" )
    )
    (bottomleft
    (inst name="CornerCellS" )
    )
    (bottom
    (inst name="POWER_VSSIOS0" )
    (inst name="POWER_VDDIOS0" )
    (inst name="PowerOnControl" )
    (inst name="mOutB08" )
    (inst name="mOutB09" )
    (inst name="mOutB10" )
    (inst name="mOutB11" )
    (inst name="POWER_VDDS" )
    (inst name="POWER_VSSS" )
    (inst name="mOutB12" )
    (inst name="mOutB13" )
    (inst name="mOutB14" )
    (inst name="mOutB15" )
    (inst name="POWER_VDDIOS1" )
    (inst name="POWER_VSSIOS1" )
    )
)

```

### CPU_wrapper_IO.v

```verilog
module CPU_wrapper_IO(
   
   input wire [12:0] InA,
   output wire [18:0] OutB
);
   
   wire [12:0] InA_core;
   wire [18:0] OutB_core;
   
// Input Ports
PDDW0204CDG mInA00(.OEN(1'b1),.I(1'b0),.PAD(InA[00]),.C(InA_core[00]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA01(.OEN(1'b1),.I(1'b0),.PAD(InA[01]),.C(InA_core[01]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA02(.OEN(1'b1),.I(1'b0),.PAD(InA[02]),.C(InA_core[02]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA03(.OEN(1'b1),.I(1'b0),.PAD(InA[03]),.C(InA_core[03]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA04(.OEN(1'b1),.I(1'b0),.PAD(InA[04]),.C(InA_core[04]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA05(.OEN(1'b1),.I(1'b0),.PAD(InA[05]),.C(InA_core[05]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA06(.OEN(1'b1),.I(1'b0),.PAD(InA[06]),.C(InA_core[06]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA07(.OEN(1'b1),.I(1'b0),.PAD(InA[07]),.C(InA_core[07]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA08(.OEN(1'b1),.I(1'b0),.PAD(InA[08]),.C(InA_core[08]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA09(.OEN(1'b1),.I(1'b0),.PAD(InA[09]),.C(InA_core[09]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA10(.OEN(1'b1),.I(1'b0),.PAD(InA[10]),.C(InA_core[10]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA11(.OEN(1'b1),.I(1'b0),.PAD(InA[11]),.C(InA_core[11]),.DS(1'b0),.PE(1'b0),.IE(1'b1));
PDDW0204CDG mInA12(.OEN(1'b1),.I(1'b0),.PAD(InA[12]),.C(InA_core[12]),.DS(1'b0),.PE(1'b0),.IE(1'b1));

// Output Ports
PDDW0204CDG mOutB00(.OEN(1'b0),.I(OutB_core[00]),.PAD(OutB[00]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB01(.OEN(1'b0),.I(OutB_core[01]),.PAD(OutB[01]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB02(.OEN(1'b0),.I(OutB_core[02]),.PAD(OutB[02]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB03(.OEN(1'b0),.I(OutB_core[03]),.PAD(OutB[03]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB04(.OEN(1'b0),.I(OutB_core[04]),.PAD(OutB[04]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB05(.OEN(1'b0),.I(OutB_core[05]),.PAD(OutB[05]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB06(.OEN(1'b0),.I(OutB_core[06]),.PAD(OutB[06]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB07(.OEN(1'b0),.I(OutB_core[07]),.PAD(OutB[07]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB08(.OEN(1'b0),.I(OutB_core[08]),.PAD(OutB[08]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB09(.OEN(1'b0),.I(OutB_core[09]),.PAD(OutB[09]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB10(.OEN(1'b0),.I(OutB_core[10]),.PAD(OutB[10]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB11(.OEN(1'b0),.I(OutB_core[11]),.PAD(OutB[11]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB12(.OEN(1'b0),.I(OutB_core[12]),.PAD(OutB[12]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB13(.OEN(1'b0),.I(OutB_core[13]),.PAD(OutB[13]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB14(.OEN(1'b0),.I(OutB_core[14]),.PAD(OutB[14]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB15(.OEN(1'b0),.I(OutB_core[15]),.PAD(OutB[15]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));

PDDW0204CDG mOutB16(.OEN(1'b0),.I(OutB_core[16]),.PAD(OutB[16]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB17(.OEN(1'b0),.I(OutB_core[17]),.PAD(OutB[17]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));
PDDW0204CDG mOutB18(.OEN(1'b0),.I(OutB_core[18]),.PAD(OutB[18]),.C(),.DS(1'b1),.PE(1'b0),.IE(1'b0));


// Corner Cell
PCORNER CornerCellN();
PCORNER CornerCellS();
PCORNER CornerCellE();
PCORNER CornerCellW();

// Power-on Control
PVDD2POC PowerOnControl();

// Core Power and Ground
PVDD1CDG POWER_VDDN();
PVDD1CDG POWER_VDDS();
PVDD1CDG POWER_VDDE();
PVDD1CDG POWER_VDDW();
PVSS1CDG POWER_VSSN();
PVSS1CDG POWER_VSSS();
PVSS1CDG POWER_VSSE();
PVSS1CDG POWER_VSSW();

// IO Power and Ground
PVDD2CDG POWER_VDDION0();
PVDD2CDG POWER_VDDION1();
PVDD2CDG POWER_VDDIOS0();
PVDD2CDG POWER_VDDIOS1();
PVDD2CDG POWER_VDDIOE0();
PVDD2CDG POWER_VDDIOE1();
PVDD2CDG POWER_VDDIOW0();
PVDD2CDG POWER_VDDIOW1();
PVSS2CDG POWER_VSSION0();
PVSS2CDG POWER_VSSION1();
PVSS2CDG POWER_VSSIOS0();
PVSS2CDG POWER_VSSIOS1();
PVSS2CDG POWER_VSSIOE0();
PVSS2CDG POWER_VSSIOE1();
PVSS2CDG POWER_VSSIOW0();
PVSS2CDG POWER_VSSIOW1();


CPU CPU_u(
   .clk(InA_core[0]),
   .rst_n(InA_core[1]),
    
   .chip_sel(InA_core[2]),
   .ready(InA_core[3]), 
   .r_valid(InA_core[4]),
   .r_data(InA_core[12:5]),  
   
   .mode(OutB_core[0]),
   .w_valid(OutB_core[1]),
   .w_data(OutB_core[9:2])
);


assign OutB_core[18:10] = 9'b0;

endmodule

```

### config.mk (pdk)

```tcl
# Process node
export PROCESS = 180

#-----------------------------------------------------
# Tech/Libs
# ----------------------------------------------------

export LEF_FILES = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/lef/tech_sage-x_tsmc_cl018g_6lm.lef \
                    /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/lef/sage-x_tsmc_cl018g.lef \
                    /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Back_End/lef/tpd018nv_280a/mt_2/6lm/lef/tpd018nv_6lm.lef \
                    /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/Bonding_PAD/Back_End/lef/tpb018v_190a/wb/6lm/lef/tpb018v_6lm.lef \
                    /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/FILLCAP/lef/tsmc18_decap_6lm.lef \
                    /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/FILLCAP/lef/tsmc18_decap_6lm_antenna.lef

export DB_FILES = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/db/sage-x_tsmc_cl018g_rvt_tt_1p8v_25c.db \
				          /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Front_End/timing_power_noise/NLDM/tpd018nv_280a/tpd018nvtc.db \
				          $(ADDITIONAL_DBS)

export LIB_FILES = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/lib/sage-x_tsmc_cl018g_rvt_tt_1p8v_25c.lib \
					        /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Front_End/timing_power_noise/NLDM/tpd018nv_280a/tpd018nvtc.lib \
					        $(ADDITIONAL_LIBS)

# Dont use cells list
export DONT_USE_CELLS = \
    RF1R1WX2 \
    RF2R1WX2

# Define fill cells
export FILL_CELLS = FILL1 FILL16 FILL2 FILL32 FILL4 FILL64 FILL8 FILLCAP4 FILLCAP8 FILLCAP16
FILLCAP32 FILLCAP64
export Filler_GDS = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/FILLCAP/gds2/tsmc18_decap.gds2
#Define tie cells
export TIEHI_CELL_AND_PORT = TIEHI HI
export TIELO_CELL_AND_PORT = TIELO LO
#Define Cap table
export CAP_TABLE = /data2/class/chenh/chenh63/TSMC18_Lib_new/t018s6ml.capTabl

#Define the scan enable port
#export SCAN_ENABLE_PORT = SCE

#--------------------------------------------------------
# Floorplan
# -------------------------------------------------------
export PLACE_DENSITY = 0.8
export PLACE_SITE = tsmc3site
export TAP_CELL_NAME = FILL2

#--------------------------------------------------------
# IO placement
# -------------------------------------------------------
export IO_LAYER = 4
export IO_FILLER_CELLS = PFILLER0005 PFILLER05 PFILLER1 PFILLER10 PFILLER20 PFILLER5
export IO_GDS = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Back_End/gds/tpd018nv_280a/mt_2/6lm/tpd018nv.gds

#--------------------------------------------------------
# Power plan
# -------------------------------------------------------
export PWR_PORT = VDD
export GND_PORT = VSS
export IO_PWR_PORT = VDDPST
export IO_GND_PORT = VSSPST
export V_STRIPE_METAL = M6
export H_STRIPE_METAL = M5
export STRIPE_WIDTH = 6
export STRIPE_SPACING = 2
export STRIPE_DISTANCE = 30
export SROUTE_MIN_LAYER = M1
export SROUTE_MAX_LAYER = M6

#---------------------------------------------------------
# Place
# --------------------------------------------------------
export MIN_GLOBAL_ROUTE_LAYER = 1
export MAX_GLOBAL_ROUTE_LAYER = 4

# --------------------------------------------------------
#  CTS
#  -------------------------------------------------------
export CTS_BUF_CELL = CLKBUFX4
export CTS_INV_CELL = CLKINVX1 CLKINVX12 CLKINVX16 CLKINVX2 CLKINVX4 CLKINVX8

#Set the routing non-default rule which are used for clk tree routing 
export CTS_ROUTING_MUL = 2
export NDR_CTS_MIN_LAYER = 2
export NDR_CTS_MAX_LAYER = 4
#Set the routing metal which are used for clk tree routing 
export CTS_ROUTING_LAYER_RANGE = 6 2

# ---------------------------------------------------------
#  Route
# ---------------------------------------------------------
export MIN_ROUTING_LAYER = 2
export MAX_ROUTING_LAYER = 6

# ---------------------------------------------------------
#  Chip Finish
# ---------------------------------------------------------
#Set the layer text num for adding PG net text when running lvs 
export LAYER_TEXT_NUM = 45
export GDS_MAP_FILE = /data2/class/chenh/chenh63/TSMC18_Lib_new/streamOut.map

# ---------------------------------------------------------
#  ExtraceRC
# ---------------------------------------------------------
#export RCX_RULES = $(DESIGN_HOME)/nangate45/pdk/rcx_patterns.rules

# ---------------------------------------------------------
#  Drc
# ---------------------------------------------------------
export STDCELL_GDS =  /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/gds2/sage-x_tsmc_cl018g_rvt.gds2

# ---------------------------------------------------------
#  Lvs
# ---------------------------------------------------------
export STDCELL_SPICE = /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/cdl/sage-x_tsmc_cl018g_rvt.cdl
export STDCELL_NETLIST =

# ---------------------------------------------------------
#  IR Drop
# ---------------------------------------------------------
export QRC_FILE = 
export PWR_NETS_VOLTAGES = 0.9
export PWR_THRESHOLD = 0.85
export GND_NETS_VOLTAGES = 0.0
export GND_THRESHOLD = 0.05
export RAIL_ANALYSIS_TEMPERATURE = 85

```

## scripts

### syn

#### dc_main.tcl

```tcl
# -------------------------------------------------------------
# Set env variable
# -------------------------------------------------------------
source $::env(SCRIPTS_DIR)/syn/dc_setup.tcl

# -------------------------------------------------------------
# Lib set up
# -------------------------------------------------------------
set target_library $::env(DB_FILES)
set link_library "* $target_library"
lappend link_library {dw_foundation.sldb}
puts $::env(VERILOG_FILES)

# -------------------------------------------------------------
# Design in
# -------------------------------------------------------------
#foreach file [ split $::env(VERILOG_FILES) {' '} ] {
#    read_file -format sverilog $file
#}
analyze -format sverilog $::env(VERILOG_FILES)
elaborate $::env(DESIGN_NAME)
current_design $::env(DESIGN_NAME)
link
check_design

# -------------------------------------------------------------
# Read sdc
# -------------------------------------------------------------
source $::env(SDC_FILE)

# -------------------------------------------------------------
# Group setting
# -------------------------------------------------------------

#remove_path_group -all
set reg [filter_collection [all_registers] "is_clock_gate != true"]
set input [all_inputs]
set output [all_outputs]
group_path -name reg2reg -weight 50 -critical_range 6 -from $reg -to $reg
group_path -name in2reg -weight 10 -critical_range 0.5 -from $input -to $reg
group_path -name reg2out -weight 10 -critical_range 0.5 -from $reg -to $output

# -------------------------------------------------------------
# Setting dontuse cell 
# -------------------------------------------------------------
#set dont use cell
puts "DONTUSE"
foreach cell [split $::env(DONT_USE_CELLS)] {
    get_lib_cell  */$cell
    set_dont_use [get_lib_cell */$cell]
}
redirect $::env(RESULT_DIR)/syn/report/check_timing.rpt {check_timing}

# -------------------------------------------------------------
# Compile design
# -------------------------------------------------------------
set_fix_multiple_port_nets -all -buffer_constants [get_designs *]
set verilog_no_tri true
compile_ultra -timing_high_effort_script
change_name -rule verilog -hier
#compile_ultra -timing_high_effort_script -scan

# -------------------------------------------------------------
# Dft configuration
# -------------------------------------------------------------
#set_dft_insertion_configuration -preserve_design_name true
#set_dft_signal -view existing_dft -type ScanClock -timing {45 55} -port {clk}
#set_dft_signal -view existing_dft -port rst_n -type Reset -active_state 0
#set_dft_signal -view existing_dft -port test_en_i -type ScanEnable -active_state 1
#set_dft_insertion_configuration -synthesis none -preserve_design_name true
#set_autofix_configuration -type bidirectional -method input
#create_test_protocol -infer_async -infer_clock

# -------------------------------------------------------------
# Scanchain insert
# -------------------------------------------------------------
#preview_dft -show all -verbose -test_points all -test_wrappers all > $::env(RESULT_DIR)/scanchain/report/preview_dft.rpt
#insert_dft
#report_dft_signal > $::env(RESULT_DIR)/scanchain/report/dft_signal.rpt
#write_test_protocol -output $::env(RESULT_DIR)/scanchain/data/test_protocol.stil
#dft_drc  -verbose > $::env(RESULT_DIR)/scanchain/report/pre_dft_drc.rpt

# -------------------------------------------------------------
# Compile design incremental
# -------------------------------------------------------------
#compile_ultra -scan -incremental

# -------------------------------------------------------------
# post-syn report&data out
# -------------------------------------------------------------

source $::env(SCRIPTS_DIR)/syn/dc_report.tcl
exit

```

#### dc_report.tcl

```tcl

#-----------------------------------------------------------------------
# output synthesis report
#-----------------------------------------------------------------------
redirect $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_check_design_after_syn.rpt {check_design}
report_timing -tran -net -input -max_paths 10000 -nworst 5 -nosplit -slack_lesser_than 0.0
redirect $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_qor.rpt {report_qor -nosplit}
redirect $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_violation.rpt {report_constraint -all_violators}

report_area -hierarchy > $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_report_area.log
report_power -hier > $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_report_power.log
report_timing -nworst 50 > $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_report_timing.log
report_timing -max 50 > $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_report_timing_max.log
report_timing -nets -max 50 > $::env(RESULT_DIR)/syn/report/$::env(DESIGN_NAME)_report_timing_net_max.log

#-----------------------------------------------------------------------
# output netlist for P&R
#-----------------------------------------------------------------------
write_sdf $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).sdf
write -format verilog -h -output $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).syn.v
write -format ddc -h -output $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).rpt.ddc
write_scan_def -output $::env(RESULT_DIR)/scanchain/data/$::env(DESIGN_NAME).scan.def

#-----------------------------------------------------------------------
# dont touch cell list output
#-----------------------------------------------------------------------
foreach cell [get_object_name [get_cells -h -f "is_mapped == true && is_hierarchical == false && dont_touch == true"]] {
    redirect -append -file $::env(RESULT_DIR)/syn/report/dont_touch.syn.tcl {puts "set_dont_touch \[get_cells $cell\]"}
}

#-----------------------------------------------------------------------
# dont use cell list output
#-----------------------------------------------------------------------
foreach cell [get_object_name [get_lib_cells -f "dont_use == true" */*]] {
    redirect -append -file $::env(RESULT_DIR)/syn/report/dont_use.syn.tcl {puts "set_dont_use \[get_lib_cells */$cell\]"}
}

```

#### dc_setup.tcl

```tcl
define_design_lib work -path $::env(RESULT_DIR)/syn/work
set sh_command_log_file $::env(RESULT_DIR)/syn/work/command.log
set_app_var alib_library_analysis_path $::env(RESULT_DIR)/syn/work
set_svf $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).svf

```


### fml

#### fml_main.tcl

```tcl
#-----------------------------------------------------------------------
# fml variable setting
#-----------------------------------------------------------------------
source $::env(SCRIPTS_DIR)/fml/fml_setup.tcl

#-----------------------------------------------------------------------
# Read db
#-----------------------------------------------------------------------
read_db -technology_library $::env(DB_FILES)

#-----------------------------------------------------------------------
# svf In
#-----------------------------------------------------------------------
set_svf $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).svf

#-----------------------------------------------------------------------
# Reference design
#-----------------------------------------------------------------------
foreach file $::env(VERILOG_FILES) {
  read_verilog -r $file
}
set_top r:WORK/$::env(DESIGN_NAME)

#-----------------------------------------------------------------------
# Implement design
#-----------------------------------------------------------------------
read_verilog -i $::env(RESULT_DIR)/syn/data/$::env(DESIGN_NAME).syn.v
set_top i:WORK/$::env(DESIGN_NAME)

#-----------------------------------------------------------------------
# Disable the scan port
#-----------------------------------------------------------------------
#set enble $::env(SCAN_ENABLE_PORT)
#setup
#current_design CPU

#-----------------------------------------------------------------------
# Run match
#-----------------------------------------------------------------------
match
report_matched_points > $::env(RESULT_DIR)/fml_syn/report/matched_points.rpt
report_unmatched_points > $::env(RESULT_DIR)/fml_syn/report/unmatched_points.rpt
report_unmatched_points -status unread > $::env(RESULT_DIR)/fml_syn/report/unread.rpt

#-----------------------------------------------------------------------
# Run verify
#-----------------------------------------------------------------------
verify
report_failing_points > $::env(RESULT_DIR)/fml_syn/report/fail_points.rpt
report_aborted_points > $::env(RESULT_DIR)/fml_syn/report/aborted.rpt
save_session -replace $::env(RESULT_DIR)/fml_syn/data/rtl2syn.fss

exit
```

#### fml_setup.tcl

```tcl
set sh_command_log_file "$::env(RESULT_DIR)/fml_syn/work/fm_shell_command.log"
set formality_log_name "$::env(RESULT_DIR)/fml_syn/work/formality.log"
define_design_lib -r -path "$::env(RESULT_DIR)/fml_syn/work" work
set verification_set_undriven_signals 0
set hdlin_vhdl_auto_file_order false

```

### pr

#### init.tcl

```tcl
set defHierChar {/}
set init_gnd_net {VSS}
set init_pwr_net {VDD}
set init_verilog {./result/syn/data/CPU.syn.v ./designs/src/ibex/CPU_wrapper_IO.v}
set init_top_cell $::env(DESIGN_NAME)_wrapper_IO
set init_lef_file [split $::env(LEF_FILES)]
puts $::env(SCRIPTS_DIR)/pr/mmmc.view
set init_mmmc_file $::env(SCRIPTS_DIR)/pr/mmmc.view
init_design

# -------------------------------------------------------------
# Check design after init
# -------------------------------------------------------------
checkDesign -netList -noHtml -outfile $::env(RESULT_DIR)/pr/report/check_data_init.report
timeDesign -prePlace \
    -pathReports \
    -drvReports \
    -slackReports \
    -numPaths 50 \
    -prefix prePlace \
    -outDir $::env(RESULT_DIR)/pr/report/init_data_timing

# -------------------------------------------------------------
# Save design 
# -------------------------------------------------------------
saveDesign $::env(RESULT_DIR)/pr/data/init_design.enc
exit

```

#### mmmc.view

```tcl
# -------------------------------------------------------------
# Create RC corner：
# -------------------------------------------------------------
if [info exists ::env(CAP_TABLE)] {create_rc_corner -name rc_max -preRoute_res 1.05 \
    -cap_table $::env(CAP_TABLE) \
    -preRoute_cap 1.05 \
    -preRoute_clkres 1.05 \
    -postRoute_res 1.05 \
    -postRoute_cap 1.05 \
    -postRoute_xcap 1.05 \
    -postRoute_clkres 1.05 \
    -postRoute_clkcap 1.05 
create_rc_corner -name rc_min -preRoute_res 1 \
    -cap_table $::env(CAP_TABLE) \
    -preRoute_cap 1 \
    -preRoute_clkres 1 \
    -postRoute_res 1 \
    -postRoute_cap 1 \
    -postRoute_xcap 1 \
    -postRoute_clkres 1 \
    -postRoute_clkcap 1 
} else {create_rc_corner -name rc_max -preRoute_res 1.05 \
    -preRoute_cap 1.05 \
    -preRoute_clkres 1.05 \
    -postRoute_res 1.05 \
    -postRoute_cap 1.05 \
    -postRoute_xcap 1.05 \
    -postRoute_clkres 1.05 \
    -postRoute_clkcap 1.05 
create_rc_corner -name rc_min -preRoute_res 1 \
    -preRoute_cap 1 \
    -preRoute_clkres 1 \
    -postRoute_res 1 \
    -postRoute_cap 1 \
    -postRoute_xcap 1 \
    -postRoute_clkres 1 \
    -postRoute_clkcap 1 
}


# -------------------------------------------------------------
# Set the lib：
# -------------------------------------------------------------
create_library_set -name lib_set_max -timing $::env(LIB_FILES)
create_library_set -name lib_set_min -timing $::env(LIB_FILES)
#create_library_set -name lib_set_max -timing $::env(MAX_LIB_FILES)
#create_library_set -name lib_set_min -timing $::env(MIN_LIB_FILES)

# -------------------------------------------------------------
# Set the SDC FILE ：
# -------------------------------------------------------------
create_constraint_mode -name common -sdc_files $::env(PR_SDC_FILE)

# -------------------------------------------------------------
# Create the delay corner：
# -------------------------------------------------------------
create_delay_corner -name delay_max -library_set {lib_set_max} -rc_corner {rc_max}
create_delay_corner -name delay_min -library_set {lib_set_min} -rc_corner {rc_min}

# -------------------------------------------------------------
# Create the analysis view：
# -------------------------------------------------------------
create_analysis_view -name max_view -constraint_mode {common} -delay_corner {delay_max}
create_analysis_view -name min_view -constraint_mode {common} -delay_corner {delay_min}

# -------------------------------------------------------------
# Set the analysis view for setup&hold：
# -------------------------------------------------------------
set_analysis_view -setup {max_view} -hold {min_view}
```

#### floor_plan.tcl

```tcl
set begin [clock format [clock seconds] -format %Y%m%d_%I:%M_%p]
puts "The FloorPlan Start: $begin"
source $::env(RESULT_DIR)/pr/data/init_design.enc

# -------------------------------------------------------------
# Define the block die area
# -------------------------------------------------------------
floorPlan -site $::env(PLACE_SITE) -su 1 $::env(PLACE_DENSITY) 20 20 20 20
#floorPlan -site $::env(PLACE_SITE) -d {1550 1550 50 50 50 50}

# -------------------------------------------------------------
# Place the block port
# -------------------------------------------------------------
if {[info exists ::env(IO_FILE)]} {
    if {$::env(IO_FILE)!=""} {
        loadIoFile $::env(IO_FILE)
    } else {
        source $::env(SCRIPTS_DIR)/pr/place_io.tcl
    }
} else {
    source $::env(SCRIPTS_DIR)/pr/place_io.tcl
}

#addIoInstance -cell PVDD1CDG -repeat 2 -inst POWER_VDD   -side W -skip 3 -spread
#addIoInstance -cell PVSS1CDG -repeat 2 -inst POWER_VSS   -side E -skip 3 -spread
#addIoInstance -cell PVDD2CDG -repeat 2 -inst POWER_VDDIO -side N -skip 3 -spread
#addIoInstance -cell PVSS2CDG -repeat 2 -inst POWER_VSSIO -side S -skip 3 -spread
#addIoInstance -cell PCORNER  -repeat 2 -inst CornerCellL -side W -skip 3 -spread
#addIoInstance -cell PCORNER  -repeat 2 -inst CornerCellH -side E -skip 3 -spread
#loadIoFile $::env(IO_FILE)

# -------------------------------------------------------------
# Add IO Filler
# -------------------------------------------------------------
addIoFiller -cell $::env(IO_FILLER_CELLS) -prefix FILLER -side n
addIoFiller -cell $::env(IO_FILLER_CELLS) -prefix FILLER -side e
addIoFiller -cell $::env(IO_FILLER_CELLS) -prefix FILLER -side w
addIoFiller -cell $::env(IO_FILLER_CELLS) -prefix FILLER -side s

setAttribute -net clk -skip_routing true
setAttribute -net rst_n -skip_routing true
setAttribute -net mode -skip_routing true
setAttribute -net ready -skip_routing true
setAttribute -net chip_sel -skip_routing true
setAttribute -net w_valid -skip_routing true
setAttribute -net r_valid -skip_routing true
setAttribute -net w_data[0] -skip_routing true
setAttribute -net w_data[1] -skip_routing true
setAttribute -net w_data[2] -skip_routing true
setAttribute -net w_data[3] -skip_routing true
setAttribute -net w_data[4] -skip_routing true
setAttribute -net w_data[5] -skip_routing true
setAttribute -net w_data[6] -skip_routing true
setAttribute -net w_data[7] -skip_routing true
setAttribute -net r_data[0] -skip_routing true
setAttribute -net r_data[1] -skip_routing true
setAttribute -net r_data[2] -skip_routing true
setAttribute -net r_data[3] -skip_routing true
setAttribute -net r_data[4] -skip_routing true
setAttribute -net r_data[5] -skip_routing true
setAttribute -net r_data[6] -skip_routing true
setAttribute -net r_data[7] -skip_routing true

#placeBondPad -ioInstName mclk_IO 		-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mrst_n_IO 		-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mready_IO 		-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_valid_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data0_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data1_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data2_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data3_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data4_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data5_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data6_IO 	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mr_data7_IO 	-pad PAD50LA -pinName PAD -position I
#
#placeBondPad -ioInstName mmode_IO 		-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_valid_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data0_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data1_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data2_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data3_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data4_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data5_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data6_IO	-pad PAD50LA -pinName PAD -position I
#placeBondPad -ioInstName mw_data7_IO	-pad PAD50LA -pinName PAD -position I


# -------------------------------------------------------------
# Set the dont use cell
# -------------------------------------------------------------
foreach cell [split $::env(DONT_USE_CELLS)] {
    get_lib_cell  */$cell
    set_dont_use [get_lib_cells */$cell] true
}

# -------------------------------------------------------------
# Add endcap cell ToDo
# -------------------------------------------------------------
set endcap_right $::env(TAP_CELL_NAME)
set endcap_left $::env(TAP_CELL_NAME)
set endcap_top $::env(TAP_CELL_NAME)
set endcap_bottom $::env(TAP_CELL_NAME)
setEndCapMode -reset
setEndCapMode -leftEdge $endcap_left -rightEdge $endcap_right -topEdge $endcap_top -bottomEdge $endcap_bottom -prefix ENDCAP

# -------------------------------------------------------------
# Add WellTap cell
# -------------------------------------------------------------
#addWellTap -cell $::env(TAP_CELL_NAME) -cellInterval 60 -prefix TAP

# -------------------------------------------------------------
# save Design
# -------------------------------------------------------------
saveDesign $::env(RESULT_DIR)/pr/data/floor_plan.enc
defOut -floorplan -noStdCells $::env(RESULT_DIR)/pr/data/ibex.floorplan.def

set end [clock format [clock seconds] -format %Y%m%d_%I:%M_%p]
puts "The FloorPlan End: $end"
exit

```

####  place_io.tcl

```tcl
set pin [dbget top.hInst.hInstTerms.defname]
set x 30
set y 300
foreach pin_line $pin {
    editPin -pin $pin_line \
        -layer $::env(IO_LAYER) \
        -assign ($x $y) \
        -pinWidth 0.10 \
        -pinDepth 0.5 \
        -global_location \
        -fixOverlap 1 \
        -fixedPin 1 \
        -edge 1 \
        -snap TRACK
    set x [expr $x+0.15]
}
saveDesign $::env(RESULT_DIR)/pr/data/io_placement.enc

```

#### power_plan.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/floor_plan.enc

# -------------------------------------------------------------
# Add Power Ring
# -------------------------------------------------------------
addRing -skip_via_on_wire_shape Noshape -exclude_selected 1 -skip_via_on_pin Standardcell \
	-center 1 -stacked_via_top_layer M6 -type core_rings -jog_distance 0.56 -threshold 0.56 \
	-nets {VDD VSS} -follow core -stacked_via_bottom_layer M1 -layer {bottom M5 top M5 right M6 left M6} \
	-width 18 -spacing 2 -offset 2
# -------------------------------------------------------------
# Global PG net connect
# -------------------------------------------------------------
globalNetConnect VDD -type pgpin -pin "$::env(PWR_PORT)" -inst *
globalNetConnect VDD -type tiehi -pin "$::env(PWR_PORT)" -inst *
globalNetConnect VDD -type net -net VDD
globalNetConnect VSS -type pgpin -pin "$::env(GND_PORT)" -inst *
globalNetConnect VSS -type tielo -pin "$::env(GND_PORT)" -inst *
globalNetConnect VSS -type net -net VSS

# globalNetConnect VDDPST -type pgpin -pin $::env(IO_PWR_PORT) -inst *
# globalNetConnect VSSPST -type pgpin -pin $::env(IO_GND_PORT) -inst *

# -------------------------------------------------------------
# Add the power stripe
# -------------------------------------------------------------
addStripe -nets {VSS VDD} \
    -layer $::env(V_STRIPE_METAL) \
    -direction vertical \
    -width $::env(STRIPE_WIDTH) \
    -spacing $::env(STRIPE_SPACING) \
    -set_to_set_distance $::env(STRIPE_DISTANCE) \
    -start_from left \
    -start_offset 1 \
    -uda power_stripe_v

addStripe -nets {VSS VDD} \
    -layer $::env(H_STRIPE_METAL) \
    -direction horizontal \
    -width $::env(STRIPE_WIDTH) \
    -spacing $::env(STRIPE_SPACING) \
    -set_to_set_distance $::env(STRIPE_DISTANCE) \
    -start_from bottom \
    -start_offset 1 \
    -uda power_stripe_h

addStripe -nets {VSS VDD} \
	-layer M4 \
	-direction vertical \
	-width $::env(STRIPE_WIDTH) \
	-spacing $::env(STRIPE_SPACING) \
	-set_to_set_distance $::env(STRIPE_DISTANCE) \
	-start_from left \
	-start_offset 1 \
	-uda power_stripe_v

# -------------------------------------------------------------
# Add the power rail
# -------------------------------------------------------------
set sroute_min_layer $::env(SROUTE_MIN_LAYER)
set sroute_max_layer $::env(SROUTE_MAX_LAYER)
sroute -connect { padPin padRing corePin floatingStripe } \
    -layerChangeRange " $sroute_min_layer $sroute_max_layer " \
    -blockPinTarget { nearestTarget } \
    -padPinPortConnect { allPort oneGeom } \
    -padPinTarget { nearestTarget } \
    #-corePinTarget { firstAfterRowEnd } \
    -corePinTarget { stripe } \
    -floatingStripeTarget { blockring padring ring stripe ringpin blockpin followpin } \
    -allowJogging 1 \
    -crossoverViaLayerRange " $sroute_min_layer $sroute_max_layer " \
    -nets { VDD VSS } \
    -allowLayerChange 1 \
    -targetViaLayerRange " $sroute_min_layer $sroute_max_layer " \
    -uda power_rail

# -------------------------------------------------------------
# Verify connect violation
# -------------------------------------------------------------
verifyConnectivity -type special \
    -noAntenna \
    -noWeakConnect \
    -noUnroutedNet \
    -error 1000 \
    -warning 50
verify_PG_short  -no_routing_blkg

# -------------------------------------------------------------
# Save design
# -------------------------------------------------------------
saveDesign $::env(RESULT_DIR)/pr/data/powerplan.enc


exit

```

#### placement.tcl

```tcl

source $::env(RESULT_DIR)/pr/data/powerplan.enc
#defIn $::env(RESULT_DIR)/scanchain/data/CPU.scan.def
# -------------------------------------------------------------
# Timing Derate setting
# -------------------------------------------------------------
set_timing_derate -delay_corner {delay_max} -early 0.97 -late 1.03 -clock
set_timing_derate -delay_corner {delay_max} -late 1.05 -data
setAnalysisMode -cppr both

# -------------------------------------------------------------
# Path group setting
# -------------------------------------------------------------
reset_path_group -all
set reg [filter_collection [all_registers] "is_integrated_clock_gating_cell != true"]
set input [all_inputs]
set output [all_outputs]
#set mem [get_cells -q -hier -filter "@is_hierarchical == false && @is_macro_cell == true"]
set ckgating [filter_collection [all_registers] "is_integrated_clock_gating_cell == true"]
set ignore_path_groups [list inp2reg reg2out reg2out feedthr]

#Group Path setting
group_path -name reg2reg -from $reg -to $reg
group_path -name reg2cg -from $reg -to $ckgating
group_path -name in2reg -from $input
group_path -name reg2out -to $output
group_path -name feedthr -from $input -to $output

setPathGroupOptions reg2reg -effortLevel high
setPathGroupOptions reg2cg -effortLevel high
setPathGroupOptions in2reg -effortLevel high
setPathGroupOptions reg2out -effortLevel low
setPathGroupOptions feedthr -effortLevel low
setOptMode -ignorePathGroupsForHold $ignore_path_groups

# -------------------------------------------------------------
# PlaceMode setting
# -------------------------------------------------------------
setPlaceMode -reset
setPlaceMode -place_global_place_io_pins false
setPlaceMode -place_detail_legalization_inst_gap 2

# -------------------------------------------------------------
# global route layer setting
# -------------------------------------------------------------
setDesignMode -process $::env(PROCESS)

# -------------------------------------------------------------
# place the design & report congestion
# -------------------------------------------------------------
place_opt_design
reportCongestion -overflow

# -------------------------------------------------------------
# Add Tie cells TODO
# -------------------------------------------------------------
setTieHiLoMode -prefix Tie -maxFanout 8 -cell {TIEHI TIELO}
setTieHiLoMode -maxDistance 30
addTieHiLo

# -------------------------------------------------------------
# Output the time report
# -------------------------------------------------------------
set report_timing_format [list timing_point arc net cell fanout load slew incr_delay delay arrival total_derate ]
setOptMode -timeDesignNumPaths 100
set_table_style -no_frame_fix_width -nosplit
timeDesign -preCTS \
    -idealClock \
    -pathReports \
    -drvReports \
    -slackReports \
    -numPaths 50 \
    -prefix preCTS \
    -outDir $::env(RESULT_DIR)/pr/report/placement_timing

# -------------------------------------------------------------
# Save design
# -------------------------------------------------------------
saveDesign $::env(RESULT_DIR)/pr/data/placement.enc
exit

```

#### cts.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/placement.enc
# -------------------------------------------------------------
# ccopt property setting
# -------------------------------------------------------------
set_ccopt_property use_inverters false

# -------------------------------------------------------------
# set cts opt use cell
# -------------------------------------------------------------

set cts_inv_cells [split $::env(CTS_INV_CELL)]
foreach libraryname $cts_inv_cells {
  puts $libraryname
  echo "setDontUse $libraryname false"
  setDontUse $libraryname false
}
set_ccopt_property inverter_cells [get_lib_cells "$::env(CTS_INV_CELL)"]

# -------------------------------------------------------------
# clk net routing ndr setting
# -------------------------------------------------------------

set mul $::env(CTS_ROUTING_MUL)
set ndr_min_layer $::env(NDR_CTS_MIN_LAYER)
set ndr_max_layer $::env(NDR_CTS_MAX_LAYER)
set routing_top_layer [lindex [split $::env(CTS_ROUTING_LAYER_RANGE)] 0]
set routing_bottom_layer [lindex [split $::env(CTS_ROUTING_LAYER_RANGE)] 1] 
add_ndr -name cts_1 \
  -width_multiplier "$ndr_min_layer:$ndr_max_layer $mul" \
  -spacing_multiplier "$ndr_min_layer:$ndr_max_layer $mul"
create_route_type -name clk_net_rule \
  -non_default_rule cts_1 \
  -top_preferred_layer $ndr_min_layer \
  -bottom_preferred_layer $ndr_max_layer
set_ccopt_property  route_type clk_net_rule -net_type trunk
setNanoRouteMode -quiet \
  -routeTopRoutingLayer $routing_top_layer \
  -routeBottomRouting $routing_bottom_layer


# -------------------------------------------------------------
# create the cts 
# -------------------------------------------------------------
create_ccopt_clock_tree_spec -file $::env(RESULT_DIR)/pr/data/clk.spec
source $::env(RESULT_DIR)/pr/data/clk.spec


# -------------------------------------------------------------
# run ccopt
# -------------------------------------------------------------
ccopt_design -cts
report_ccopt_skew_groups
timeDesign -postCTS \
  -pathReports \
  -drvReports \
  -slackReports \
  -numPaths 50 \
  -prefix postCTS \
  -outDir $::env(RESULT_DIR)/pr/report/cts_timing

saveDesign $::env(RESULT_DIR)/pr/data/cts.enc
exit

```

#### post_cts_opt.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/cts.enc

set_interactive_constraint_modes [all_constraint_modes -active]
set_propagated_clock [all_clocks]
setOptMode -fixDrc true -fixFanoutLoad true

# -------------------------------------------------------------
# opt design after CTS
# -------------------------------------------------------------
optDesign -postCTS
optDesign -postCTS -hold

saveDesign $::env(RESULT_DIR)/pr/data/post_cts_opt.enc
timeDesign -postCTS -pathReports -drvReports -slackReports -numPaths 50 -prefix postCTS -outDir $::env(RESULT_DIR)/pr/report/cts_opt_timing

exit
```

#### routing.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/post_cts_opt.enc


# -------------------------------------------------------------
# NanoRoute Mode setting
# -------------------------------------------------------------
setNanoRouteMode -quiet -routeWithTimingDriven true
setAnalysisMode -analysisType onChipVariation
setNanoRouteMode -quiet -drouteEndIteration 70
setNanoRouteMode -quiet -drouteFixAntenna true
setNanoRouteMode -quiet -drouteUseMultiCutViaEffort medium
setNanoRouteMode -quiet -drouteMinSlackForWireOptimization 0.1
setNanoRouteMode -quiet -routeTopRoutingLayer $::env(MAX_ROUTING_LAYER)
setNanoRouteMode -quiet -routeBottomRoutingLayer $::env(MIN_ROUTING_LAYER)
setDelayCalMode -engine default -siAware true

setNanoRouteMode -drouteFixAntenna true
setNanoRouteMode -quiet -routeAntennaCellName "ANTENNA"
setNanoRouteMode -quiet -routeInsertAntennaDiode true

# -------------------------------------------------------------
# Route Design
# -------------------------------------------------------------
routeDesign -globalDetail
saveDesign $::env(RESULT_DIR)/pr/data/routing.enc

# -------------------------------------------------------------
# save Design
# -------------------------------------------------------------
timeDesign -postRoute -prefix postRoute -outDir $::env(RESULT_DIR)/pr/report/routing_timing

exit
```

#### routing_opt.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/routing.enc

# -------------------------------------------------------------
# OptDesign after routing
# -------------------------------------------------------------
optDesign -postRoute -setup

# -------------------------------------------------------------
# Add filler cell
# -------------------------------------------------------------
set filler_cell [split $::env(FILL_CELLS)]
setFillerMode -doDRC true -corePrefix Filler -core $filler_cell
addFiller

# -------------------------------------------------------------
# Save Design
# -------------------------------------------------------------
timeDesign -postRoute -prefix postRouteOpt -outDir $::env(RESULT_DIR)/pr/report/routing_opt_timing
saveDesign $::env(RESULT_DIR)/pr/data/routing_opt.enc
```

#### chip_done.tcl

```tcl
source $::env(RESULT_DIR)/pr/data/routing_opt.enc
# -------------------------------------------------------------
# Remove the unused net&module
# -------------------------------------------------------------
remove_assigns -buffering
deleteDanglingNet
deleteEmptyModule

# -------------------------------------------------------------
# Re-connect the PG net after add physical cell
# -------------------------------------------------------------
globalNetConnect VDD -type net -net VDD
globalNetConnect VSS -type net -net VSS
verifyConnectivity -type all -error 1000 -warning 50

# -----------------------------------------------------------------
# output the hcell list for lvs & ir-drop analysis
# -----------------------------------------------------------------
set hcell_file [open $::env(SCRIPTS_DIR)/pv/hcell_list w]
set hcell_list_ir [open $::env(SCRIPTS_DIR)/ir_v/cell_list w]
set cells [dbGet [dbGet top.insts.isPhysOnly 0 -p].cell.name]
set cells [lsort -u $cells]
foreach cell $cells {
    puts $hcell_file "$cell $cell"
    puts $hcell_list_ir "$cell"
}
close $hcell_file
close $hcell_list_ir

# -----------------------------------------------------------------
# Write out the netlist
# ibex_routing.vg does not contain the PG port&net
# ibex_lvs.vg contain the PG port&net for run v2lvs 
# -----------------------------------------------------------------
#saveNetlist $::env(RESULT_DIR)/pr/data/ibex_routing.vg
#saveNetlist -excludeLeafCell -includePowerGround -flattenBus $::env(RESULT_DIR)/pr/data/ibex_lvs.vg
saveNetlist -excludeLeafCell -includePowerGround -includePhysicalCell {FILLCAP4 FILLCAP8 FILLCAP16 FILLCAP32 FILLCAP64} -flattenBus $::env(RESULT_DIR)/pr/data/ibex_lvs.vg

# -----------------------------------------------------------------------
#  output netlist&sdf for post_pr simulation
# -----------------------------------------------------------------------
saveNetlist $::env(RESULT_DIR)/sim/post_pr/$::env(DESIGN_NAME).pr.v
write_sdf	$::env(RESULT_DIR)/sim/post_pr/$::env(DESIGN_NAME).syn.sdf

# ----------------------------------------------------------------
# Write out the routing def
# ----------------------------------------------------------------
set lefDefOutVersion 5.8
defOut -floorplan -netlist -routing $::env(RESULT_DIR)/pr/data/ibex_routing.def


# -----------------------------------------------------------------
# Gds streamOut
# -----------------------------------------------------------------
set IO_GDS $::env(IO_GDS)
set STDCELL_GDS $::env(STDCELL_GDS)
set Filler_GDS $::env(Filler_GDS)

setStreamOutMode -textSize 5
setStreamOutMode -virtualConnection true
setStreamOutMode -uniquifyCellNamesPrefix true
setStreamOutMode -check_map_file true
streamOut \
	$::env(RESULT_DIR)/pr/data/CPU.gds \
	-mapFile $::env(GDS_MAP_FILE) \
	-merge  {$IO_GDS $STDCELL_GDS $Filler_GDS} \
	-libName DesignLib  \
	-units 1000 \
	-mode ALL

saveDesign $::env(RESULT_DIR)/pr/data/chip_done.enc
exit

```


### pv

#### drc_rule

```tcl
// ENVIROMENT SETTING

PRECISION 1000
RESOLUTION 5

LAYOUT SYSTEM GDSII
LAYOUT PATH "./result/pv/drc/data/CPU.mergeSTD.gds"
LAYOUT PRIMARY "CPU_wrapper_IO"

DRC RESULTS DATABASE "DRC_RES.db"


//Layer Define 
LAYER diff 2
LAYER tap 3
LAYER nwell 4
LAYER dwell 5
LAYER pwbm 6
LAYER pwde 7
LAYER hvtr 8
LAYER hvtp 9
LAYER ldntm 10
LAYER hvi 11
LAYER tunm 12
LAYER lvtn 13
LAYER poly 14
LAYER hvntm 15
LAYER nsdm 16 
LAYER psdm 17
LAYER rpm 18
LAYER urpm 19
LAYER npc 20
LAYER licon1 21
LAYER li1 22
LAYER mcon 23
LAYER met1 24
LAYER via 25
LAYER met2 26
LAYER via2 27
LAYER met3 28
LAYER via3 29
LAYER met4 30
LAYER via4 31
LAYER met5 32

//Rule define
rule_1 {@met1 minimum width is 100
		INTERNAL met1<100
}

```

#### gds_merge.tcl

```tcl
layout filemerge -in ../PDK/pdk/CPU_pad_sealring_dum.gds \
-in $::env(STDCELL_GDS) \
-topcell CPU_wrapper_IO \
-mode overwrite \
-out $::env(RESULT_DIR)/pv/drc/data/$::env(DESIGN_NAME).mergeSTD.gds

layout filemerge -in ../PDK/pdk/CPU_pad_sealring_dum.gds \
-in $::env(IO_GDS) \
-topcell CPU_wrapper_IO \
-mode overwrite \
-out $::env(RESULT_DIR)/pv/drc/data/$::env(DESIGN_NAME).mergeIO.gds

```

#### lvs rule

```tcl
LAYOUT SYSTEM GDSII
LAYOUT PATH "./result/pv/lvs/data/CPU.text.gds"
LAYOUT PRIMARY CPU
SOURCE SYSTEM SPICE
SOURCE PATH "./result/pv/lvs/data/CPU.spi.gz"
SOURCE PRIMARY CPU
MASK RESULTS DATABASE ibex_lvs.db
LVS REPORT ibex_lvs.rpt

//Layer Define 
LAYER diff 2
LAYER tap 3
LAYER nwell 4
LAYER dwell 5
LAYER pwbm 6
LAYER pwde 7
LAYER hvtr 8
LAYER hvtp 9
LAYER ldntm 10
LAYER hvi 11
LAYER tunm 12
LAYER lvtn 13
LAYER poly 14
LAYER hvntm 15
LAYER nsdm 16 
LAYER psdm 17
LAYER rpm 18
LAYER urpm 19
LAYER npc 20
LAYER licon1 21
LAYER li1 22
LAYER mcon 23
LAYER met1 24
LAYER via 25
LAYER met2 26
LAYER via2 27
LAYER met3 28
LAYER via3 29
LAYER met4 30
LAYER via4 31
LAYER met5 32

```

#### trans_spi.sh

```tcl
cells_path=$DESIGN_HOME/sky130hd/pdk/
v2lvs -sl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__einvn_0.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__einvn_8.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__einvn_1.cdl \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__einvn_4.v.copy \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__einvn_2.cdl \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__einvn_0.v.copy \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__einvn_2.v.copy \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__einvn_4.cdl \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__einvn_8.v.copy \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__einvn_1.v.copy \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__dlymetal6s6s_1.v.copy \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__dlymetal6s6s_1.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__or3b_1.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__or3b_2.cdl \

......

-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__clkbuf_8.v.copy \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__clkbuf_1.v.copy \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__clkbuf_16.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__clkbuf_4.cdl \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__clkbuf_8.cdl \
-l $cells_path/all_stdcell_netlist/sky130_fd_sc_hd__clkbuf_4.v.copy \
-s $cells_path/all_stdcell_spice/sky130_fd_sc_hd__clkbuf_1.cdl \
-v $RESULT_DIR/pr/data/ibex_lvs.vg \
-o $RESULT_DIR/pv/lvs/data/CPU.spi.gz \
-s0 VSS -s1 VDD \
-log vslvs.log

```

###  extractRC

#### extractRC.cmd

```tcl
STAR_DIRECTORY: ./result/extractRC/work
SUMMARY_FILE: ./result/extractRC/report/summary.log
TOP_DEF_FILE: ./result/pr/data/ibex_routing.def
LEF_FILE: /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/lef/tech_sage-x_tsmc_cl018g_6lm.lef
LEF_FILE: /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/SC_ARM/lef/sage-x_tsmc_cl018g.lef
LEF_FILE: /data2/class/chenh/chenh63/TSMC18_Lib_new/lib/IO/Back_End/lef/tpd018nv_280a/mt_2/6lm/lef/tpd018nv_6lm.lef
	
MAPPING_FILE:	/data2/class/chenh/chenh65/HUAWEI/ibex_work_upload/streamOut.map
TCAD_GRD_FILE: /data2/class/chenh/chenh63/TSMC18_Lib_new/star_rc/t018s6ml.nxtgrd
NETLIST_FILE: ./result/extractRC/data/ibex.spef.gz
NETLIST_FORMAT: spef
NETLIST_COMPRESS_COMMAND: gzip -q -f
OPERATING_TEMPERATURE: 125
NUM_CORES: 4

CASE_SENSITIVE:YES
NETLIST_UNSCALED_COORDINATES:YES
NETLIST_UNSCALED_RES_PROP:YES
BUS_BIT: []
HIERARCHICAL_SEPARATOR: /
EXTRACTION: RC
COUPLING_REL_THRESHOLD: 0.03
COUPLING_ABS_THRESHOLD: 1e-15
REDUCTION: YES
NETS: *
REMOVE_FLOATING_NETS: NO
REMOVE_DANGLING_NETS: NO

```


### sta

#### lib_setup.tcl

```tcl
set target_library $::env(DB_FILES)
set link_library "* $target_library"
```

#### run_timing.tcl

```tcl
# -------------------------------------------------------------
# Set pt variable
# -------------------------------------------------------------
set read_parasitics_load_locations true
set sh_source_uses_search_path true
set sh_command_log_file $::env(RESULT_DIR)/sta/log/pt_shell_command.log
set parasitics_log_file $::env(RESULT_DIR)/sta/log/parasitics_command.log
printvar sh_eco_enabled
set EnablePTSI true
if {$EnablePTSI == "true"} {
    set si_enable_analysis true
    set si_xtalk_double_switching_mode clock_network
}

# -------------------------------------------------------------
# Read the timing lib
# -------------------------------------------------------------
source $::env(SCRIPTS_DIR)/sta/lib_setup.tcl

# -------------------------------------------------------------
# Read the post-routing netlist
# -------------------------------------------------------------
read_verilog $::env(RESULT_DIR)/pr/data/ibex_routing.vg
set DESIGN_NAME CPU
current_design $DESIGN_NAME
link

# -------------------------------------------------------------F
# Read the SDC
# -------------------------------------------------------------
source -e -v $::env(SDC_FILE)

# -------------------------------------------------------------
# Read the spef
# -------------------------------------------------------------
read_parasitics -keep_capacitive_coupling -format spef $::env(RESULT_DIR)/extractRC/data/ibex.spef.gz

# -------------------------------------------------------------
# Output the annotated report
# -------------------------------------------------------------
redirect $::env(RESULT_DIR)/sta/report/CPU_annotated_parasitics.report {report_annotated_parasitics}

# -------------------------------------------------------------
# Setting the group path
# -------------------------------------------------------------
set reg [filter_collection [all_registers] "is_integrated_clock_gating_cell != true"]
set input [all_inputs]
set output [all_outputs]
set ckgating [filter_collection [all_registers] "is_integrated_clock_gating_cell == true"]
set ignore_path_groups [list inp2reg reg2out reg2out feedthr]
#set mem [get_cells -q -hier -filter "@is_hierarchical == false && @is_macro_cell == true"]

group_path -name reg2reg -from $reg -to $reg
group_path -name reg2cg -from $reg -to $ckgating
group_path -name in2reg -from $input
group_path -name reg2out -to $output
group_path -name feedthr -from $input -to $output
#group_path -name mem2reg -from $mem -to $reg
#group_path -name mem2cg -from $mem -to $ckgating
#group_path -name reg2men -from $reg -to $men
#group_path -name mem2men -from $mem -to $men

# -------------------------------------------------------------
# Enable propagated clock 
# -------------------------------------------------------------
set_noise_parameters -enable_propagation
set_propagated_clock [all_clocks]
update_timing -full

# -------------------------------------------------------------
# output timing report
# -------------------------------------------------------------
check_timing -v > $::env(RESULT_DIR)/sta/report/CPU_check_timing.report
redirect $::env(RESULT_DIR)/sta/report/CPU_report_timing.report {report_timing \
    -crosstalk_delta \
    -delay max \
    -nosplit \
    -input \
    -net \
    -sign 4}
redirect $::env(RESULT_DIR)/sta/report/CPU_report_constraints.report {report_constraint \
    -significant_digit 4 \
    -all_violators \
    -nosplit}
report_design > $::env(RESULT_DIR)/sta/report/CPU_report_design.rpt
report_net > $::env(RESULT_DIR)/sta/report/CPU_report_net.rpt
report_clock -skew -attribute > $::env(RESULT_DIR)/sta/report/CPU_report_clock.rpt
report_analysis_coverage -status_details { untested violated } \
    -nosplit \
    -sort_by slack \
    -check_type { setup \
        hold \
        recovery \
        removal \
        nochange \
        min_period \
        min_pulse_width \
        clock_separation \
        max_skew \
        clock_gating_setup \
        clock_gating_hold \
        out_set out_hold} \
    -exclude_untested {constant_disabled \
        mode_disabled \
        user_disabled \
        no_paths \
        false_paths \
        no_endpoint_clock no_clock} \
    > $::env(RESULT_DIR)/sta/report/CPU_analysis_coverage.rpt

# -------------------------------------------------------------
#save pt session
# -------------------------------------------------------------
save_session $::env(RESULT_DIR)/sta/data/CPU_session

```


### ir_v

#### run_static.tcl

```tcl
#------------------------------------------------------
# load design
#------------------------------------------------------
source $::env(SCRIPTS_DIR)/ir_v/load_design.tcl

#------------------------------------------------------
# gen technology file
#------------------------------------------------------
source $::env(SCRIPTS_DIR)/ir_v/gen_techfile.tcl

#------------------------------------------------------
# set multi CPU
# ------------------------------------------------------
setMultiCpuUsage -localCpu  8

#-----------------------------------------------------------------------
# Read spef file
#-----------------------------------------------------------------------
spefIn -rc_corner rc_max $::env(RESULT_DIR)/extractRC/data/ibex.spef.gz

#-----------------------------------------------------------------------
# Static Power Analysis
#-----------------------------------------------------------------------
set_power_analysis_mode \
    -reset
set_power_analysis_mode \
    -leakage_power_view 	max_view \
    -dynamic_power_view        	max_view \
    -write_static_currents      true \
    -binary_db_name             staticPower.db \
    -create_binary_db           true \
    -method                     static

#-----------------------------------------------------------------------
# set toggle rate on the rst pin and propagate activities:
#-----------------------------------------------------------------------
set_switching_activity \
    -reset

set_switching_activity \
    -input_port                 rst \
    -activity                   0.5 \
    -duty                       0.5

propagate_activity

#-----------------------------------------------------------------------
# set default activities for all nets/pins/etc for unclocked nets
#-----------------------------------------------------------------------
set_default_switching_activity \
    -input_activity             0.5 \
    -period                     4.0 \
    -clock_gates_output_ratio   0.5

#-----------------------------------------------------------------------
# define output directory
#-----------------------------------------------------------------------
set_power_output_dir            $::env(RESULT_DIR)/ir_v/data

#-----------------------------------------------------------------------
# run power analysis
#-----------------------------------------------------------------------
report_power \
    -outfile                    static.rpt

#-----------------------------------------------------------------------
# Static Rail Analysis
#-----------------------------------------------------------------------
set_rail_analysis_mode \
    -method                     static \
    -accuracy                   xd \
    -analysis_view              max_view \
    -power_grid_library         { \
        ./result/ir_v/data/tech_pgv/techonly.cl \
    } \
    -enable_rlrp_analysis       true \
    -verbosity true \
    -temperature "$::env(RAIL_ANALYSIS_TEMPERATURE)"

#-----------------------------------------------------------------------
# Define voltages and thresholds that are used by the rail analysis engine.
#-----------------------------------------------------------------------
set_pg_nets -net VDD -voltage $::env(PWR_NETS_VOLTAGES) -threshold $::env(PWR_THRESHOLD) -force
set_pg_nets -net VSS -voltage $::env(GND_NETS_VOLTAGES) -threshold $::env(GND_THRESHOLD) -force

#-----------------------------------------------------------------------
# define voltage source location
#-----------------------------------------------------------------------
set_power_pads \
    -reset

set_power_pads \
    -net                        VDD \
    -format                     xy \
    -file                       ./scripts/ir_v/VDD.pp

set_power_pads \
    -net                        VSS \
    -format                     xy \
    -file                       ./scripts/ir_v/VSS.pp

#-----------------------------------------------------------------------
#define power consumption
#-----------------------------------------------------------------------
set_power_data -reset
set_power_data \
    -format                     current \
    { \
        ./result/ir_v/data/static_VDD.ptiavg \
        ./result/ir_v/data/static_VSS.ptiavg \
    }

analyze_rail \
    -output           ./result/ir_v/report \
    -type                       net \
                                VDD

```

#### load_design.tcl

```tcl
#------------------------------------------------------
# load design
#------------------------------------------------------
read_design -physical_data $::env(RESULT_DIR)/pr/data/chip_done.enc.dat $::env(DESIGN_NAME)

```

#### gen_techfile.tcl

```tcl
set_pg_library_mode \
    -celltype                   techonly \
    -ground_pins                VSS \
    -power_pins                 "VDD $::env(PWR_NETS_VOLTAGES)" \
    -default_area_cap           0.01 \
    -extraction_tech_file       "$::env(QRC_FILE)" 
    #-decap_cells                "$decap_cell_name" \
    #-filler_cells               "$filler_name" \
    #-cell_decap_file            ../data/voltus/decap.cmd 

generate_pg_library \
    -output                     ./result/ir_v/data/tech_pgv

#-----------------------------------------------------------------------
# Standard cells pgv generation ToDo
#-----------------------------------------------------------------------
#set_pg_library_mode \
#    -ground_pins                VSS \
#    -power_pins                 {VDD 0.9} \
#    -decap_cells                {DECAP8 DECAP64 DECAP4 DECAP32 DECAP2 DECAP16 DECAP1} \
#    -filler_cells               { FILL8  FILL64  FILL4  FILL32  FILL2  FILL16  FILL1} \
#    -celltype                   stdcells \
#    -cell_decap_file            ../data/voltus/decap.cmd \
#    -cell_list_file 		../data/voltus/cell.list \
#    -spice_subckts { \
#        ../data/netlists/gsclib090.sp \
#        ../data/netlists/pso_header.spi \
#       ../data/netlists/pso_ring.spi \
#    } \
#    -current_distribution       propagation \
#    -spice_models               ../data/netlists/spectre_load.sp \
#    -extraction_tech_file       ../data/qrc/gpdk090_9l.tch 

#generate_pg_library \
#    -output                      ./result/ir_v/data/stdcell_pgv

```

### lib2db

#### lib2db.tcl

```tcl
set lib_path "/home/ic/my_work/designs/nangate45/pdk/lib"
set lib_file "[glob -directory $lib_path *.lib]"
foreach curlib $lib_file {
	read_lib $curlib
	set libname "[get_object_name [get_libs]]"
	write_lib -format db $libname -output "[format %s.%s [file rootname $curlib] db]"
	remove_design -all
}

```