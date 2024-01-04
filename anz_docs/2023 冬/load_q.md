- 10.12
tlb好像会出来两个接口，同时通知load_q，还没写进去
store-load违例是重新从**取指**执行

- 10.16
**部分重构**：起因是发现TLB MISS之后发给load_q的pa，无法直接从load流水线的第二级开始运行，因为需要load流水线第一级用va 访问dcache拿到tag，然后第二级再进行pa和va的对比。
 $~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$$\Downarrow$
发现这样的话，TLB回填回来的PA就不用占用LOAD前递选择了，当作正常指令重新走一遍流水线，只是不访问TLB。发现前递的store data invalid这种情况亦不可以直接发送给前递选择，因为如果前递是部分覆盖的话，还是要访问dcache拿到dcache里面的结果，所以决定对这种情况，也直接重新走一遍流水线。
 $~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~$    $\Downarrow$
而对于dcache bank conflict和mshr full这两种情况，dcache bank conflict可能是tag访问冲突，所以也要重头走流水线，mshr full就不用了，肯定是miss，重新发给dcache就够了
4个重发，tlb回填（立即重发，从头），bank conflict（立即重发，从头），mshr full（立即重发，从头），store_data_invalid（等待重发，从头）

问mmio最后通过什么指示ldqidx
