### 写TAGE预测器遇到的
#### 如何获取低位？
```C++
//例如要获取pc的低14位，就用想与的形式
uint64_t pc; uint_64 yu=0x3fff;
pc = pc&yu;
```

#### 数组动态大小C++怎么办？
![[Pasted image 20240517112428.png]]
数组大小必须在编译的时候就被确定，但是可以用动态内存分配，C++里面的new来解决这个问题。
**引入数组和指针的基本概念**
```C++
//指针可以理解为
(int*) ptr //ptr存的是地址，加上*代表的才是内容
int var =10; ptr=&var;//如果指向基本的int类型的数据，那么就要取地址，但是如下怎么理解呢？
int * ptr =&var//其实可以理解为
(int*) ptr=&var//这样理解就清晰了很多
cout<<*ptr;//会输出10
//下面看数组
int balance[10];//balance，即数组名，也是指针，是一个指向数组中第一个元素的常量指针
//所以可以直接
(int*) ptr=balance;//再说加1的问题
//如果定义的是int数组，那么balance+1 表示的不是地址加1，而是地址加4，如果是double则是加8
//同理，ptr加1也是加4
```
**数组左移**
左移右移需要注意，到底是s[10]=s[9]，这是左移，不能从s[2]=s[1]开始
右移是s[1]=s[2]，不能从s[9]=s[10]开始


#### 构造函数用初始化参数列表
```C++
class point{
public:
point(int xx,int yy):x(xx),y(yy){}
private: int x;int y;
}
```
**析构函数是用于在对象被删除之前的清理工作，在对象生命周期即将结束时被自动调用。(析构函数可以清理对象并且释放内存)**
下面展示一个正确的，不知道数组大小用动态分配的方法，初始化构造列表的方法，最后在析构函数中删除的C++语句
```C++
class table
{public:
uint32_t table_i; //uint16_t table_history_width;
uint32_t * history;
table(uint32_t i):table_i(i),history(new uint32_t[HISTORY_WIDTH[table_i]-1]){}
~table(){
    delete[] history;
    cout<<"调用了析构函数";}
};
```

数组适合什么样的数据结构？
