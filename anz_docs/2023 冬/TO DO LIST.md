- 10.12
- [x] 将前递重发逻辑加入load_q的文档中
- [x] 确定store_q发生前递失败了怎么处理
- [x] 对上香山dcache接口
   - [x] refill的接口是256怎么整
   - [x] miss怎么通知，
   - 直接通过流水线给miss_valid信号，再给ldq一个ldqidx，什么时候通知对性能没影响
   - [x] released解决
   - [x] dcache bank conflict怎么通知
   - [x] mshr fail怎么通知
- [x] sgh 异常，mmio怎么处理？
- [x] released解决

- 10.24
- [x] 原子指令是啥
- [x] 原子指令怎么运转
- [x] 看一篇论文
- [x] 讲store_q

- 10.25                                        
- [x] **建立基本的load流水线，搞清楚哪里还存疑的**
- [x] **看那篇2017年的高通论文**
- [x] LSU里面异常的详细阐述
- [x] 线程和进程的区别 ？
- [x] 原子指令怎么运转