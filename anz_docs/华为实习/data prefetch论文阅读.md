## 感兴趣程度低：
ISCA 2024 无
ISCA 2023 无
ISCA 2022 一篇
Register file prefetching
机构：INTEL
具体领域：从L1 预取到 寄存器堆中

ISCA 2021 
A Cost-Effective Entangling Prefetcher for Instructions
机构：University of Murcia
<u>具体领域：ICACHE的预取</u> **感兴趣程度：低**

## 一般


Prefetched Address Translation
加速TLB的

MICRO 2020
RnR: A Software-Assisted Record-and-Replay Hardware Prefetcher
RnR（Record-and-Replay）的软件辅助硬件预取器。这种预取器专注于处理具有重复不规则内存访问模式的数据结构（时间局部性，指针套指针）。提供一个编程接口，在内存访问模式第一次出现时记录缓存缺失序列，并通过在接下来的重复中重播该模式来预取。提出的RnR预取器提供了一个轻量级的软件接口，以便程序员可以在应用程序代码中指定:1)哪些数据结构具有不规则的内存访问，2)何时开始记录，以及3)何时开始重播(预取)。




问题模型的转化：基于strided和best offset这种预取器，很好预测，本来需要的存储空间就较小，在带宽很大的情况下，本来的收益就是利用带宽“猛取”，很难说放在把预测表放在片外有什么收益。真正有收益的应该是基于时间局部性的预取，指针套指针



# 高

**到底为啥prefetch不能跨页**
虚拟地址跨页仍然连续的，因为程序可见的是虚拟地址，物理地址跨页可能没了，因为连续的虚拟地址跨页可能被分到很远的物理地址
## 滤波
ISCA 2019
Perceptron-Based Prefetch Filtering
机构：Texas A&M University INTEL
<u>具体领域：用神经网络滤波预取请求 </u> **感兴趣程度：高**
提及sota的预取器有：Best Offset Prefetcher (BOP), DRAM Aware -Access Map Pattern Matching (DA-AMPM) [32] and Signature Path Prefetcher (SPP)
BOP是第二届锦标赛的冠军 SPP表现出比BOP更优的性能

## Offset
HPCA'16
Best Offset Prefetch

ISCA-DPC3'19
Multi-Lookahead Offset Prefetching

MICRO'22
Berti: an Accurate Local-Delta Data Prefetcher
带宽受限情况下的预取，高精度的预取



## 时间
**用了哪些信息（历史 and 环境信息），增加面积有没有益**
**（时机：每次都要访问肯定不现实，增加了带来的好处和挑战，比如过预测 ）**
**（隐性的反馈机制，空间复杂度，，可分割，能不能提前拿出来meta的信息，trick，如果可以trigger是什么，）**

HPCA'09
Practical Off-chip Meta-data for Temporal Memory Streaming（STMS）

MICRO'13
Linearizing Irregular Memory Accesses for Improved Correlated Prefetching（ISB)

HPCA'18
Domino Temporal Data Prefetcher(Donmio)

ISCA'19
Efficient metadata management for irregular data prefetching (MISB)
<u>具体领域：时间局部性的预取器，高效布局在片内 </u> **感兴趣程度：高**
提及sota的irregular 的预取器：STMS和ISB

MICRO'19
Temporal Prefetching Without the Off-Chip Metadata(Triage)
机构：ARM
<u>具体领域：时间局部性的预取器，旨在用资源换带宽，牺牲部分预取，仍然有效，换取多核系统的带宽 </u> **感兴趣程度：一般**

## 空间

### 基于 Region
ISCA'06
Spatial Memory Streaming(SMS)

ICS'09
AMPM

HPCA'19
Bingo Spatial Data Prefetcher(Bingo)

MICRO'19
Dual Spatial Pattern Prefetcher(DSPatch)
机构：INTEL，ETHZ
<u>具体领域：当有更高的 DRAM 带宽可用时，当前最先进的预取器在提取更高性能方面表现不佳，预取器动态适应可用带宽，在有余量时增加预取计数和预取覆盖率 </u> **感兴趣程度：高**
sota:SPP,BOP,SMS

### 基于delta

MICRO ’15
Efficiently Prefetching Complex Address Patterns（VLDP）

MICRO‘16
Path Confidence based Lookahead Prefetching（SPP）

ISCA‘20
Bouquet of Instruction Pointers: Instruction Pointer Classifier-based Spatial Hardware Prefetching.（IPCP）
机构：INTEL 印度理工
<u>具体领域：空间预取器,从L2搬到了L1</u> **感兴趣程度：高，目前无insights** 


## 神经网络

MICRO’21
Pythia: A Customizable Hardware Prefetching Framework Using Online Reinforcement Learning
机构：ETHZ INTEL
使用在线强化学习来提高内存访问效率。Pythia通过观察多种不同的程序上下文信息，并结合当前内存带宽使用情况，来做出预取决策

ASPLOS'24
Pathfinder: Practical Real-Time Learning for Data Prefetching
神经网络预取

ASPLOS'21
A Hierarchical Neural Model of Data Prefetching(Voyager)
神经网络预取
## 未分类


MICRO 2022
Merging Similar Patterns for Hardware Prefetching
机构：中科院软件所

小总结：DPC2的冠军：BOP，MLOP是BOP的扩展
VLDP：空间预取器
SPP-PFF能有效地提供比VLDP更高的预取覆盖
BINGO：一般来说，bingo比VLDP和SPP-PFF更好的性能 在spec 2017上，但是需要更多的面积
IPCP：DPC-3的冠军。它将IP分为CS (constant stride)、CPLX (complex stride)和GS (global stream)三种类型。IPCP使用三个轻量级预取器，根据IP类发出预取请求。如果无法将IP划分为三种类型之一，则使用下一行预取器
**此预取器开源了，也提到只是在存储空间有限的情况下优于机器学习的**

Differential-Matching Prefetcher for Indirect Memory Access
西交 优于IPCP berti


HPCA 2024
A Two Level Neural Approach Combining Off-Chip Prediction with Adaptive Prefetch Filtering
华为瑞士研发中心，巴塞罗那超算，TEXA A&M

MICRO 2023
CLIP: Load Criticality based Data Prefetching for Bandwidth-constrained Many-core Systems
印度理工
在带宽受限的众核系统中，现有的硬件预取技术（如Berti L1预取器）无法提供性能改进，甚至可能导致性能下降。这是因为在DRAM带宽受限的情况下，预取器的高命中率并不能保证有效的数据访问