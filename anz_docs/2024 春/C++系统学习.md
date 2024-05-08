### 前三章
``` C++
//关于string
include <string>
a=hw.size()返回大小
//如果while (cin<<b)想确定终止，可以用
if(cin.get() =='\n') break;//注意直接回车，不要空格

//vector
include <vector>; include<algorithm>;vector<string> a;
推入vector尾部：a.push_back(x)
```
#### vector
可以用max_element和min_element求最大最小值的位置，用星号求最大最小值
![[Pasted image 20240325154732.png|600]]
**注意：对于vector类型的判断for(i=0;i<=a.size();i++),不可以用<=，因为比如一共有1个元素，但此时是访问的a[0]，可用i != a.size()**
**sort**函数：默认从小到大，也可以从大到小
![[Pasted image 20240329144345.png|500]]
**如果用while(cin>>x)的话** 如果操作成功，x将保存刚刚读到的值，同时while语句的测试也会成功，无论是输入已读完还是遇到了对x的类型的无效输入，都算失败

#### domain_error
在头文件”stdexcept“中

#### string
当使用cin string时候，遇到空白就会停止，只有使用while（getline(cin,name)
)的时候遇到空格不会终止，只有遇到回车会终止

#### 指针
指针是一种变量。它的内存单元中保存的是一个标识的地址，在64位平台上用sizeof测试为8。
**字符串**在C++里面存的是首地址，所以char 星 a="hello"是没问题的，string就等于char*，string的底层是用char星实现的，但是int 星不行，猜测是加1的区别

### 星星项目
基类的函数调用如果有virtual则根据多态性调用派生类的，如果没有virtual则是正常的静态函数调用，还是调用基类的。
[C++中virtual关键字的用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/147601339)

### 成员访问运算符有.和->
点运算符“.” 其中点运算获取类对象的一个成员
（1）如果成员所属的对象是左值，则结果是左值
（2）如果成员所属的对象是右值，则结果是右值
箭头运算符“->”， ，箭头运算获取指针指向对象的成员,箭头运算符作用一个指针的运算对象，结果为左值
表达式ptr->men等价于(星ptr).mem：

### 构造函数
这一块暂时可以不深究，构造函数分带参和不带参的，带参的可以用初始化参数列表的形式来写：
![[Pasted image 20240418171149.png]]
**注意**：如果在一个类里，调用另一个类的对象的构造函数，只能写入初始化参数列表
[C++构造函数的各种用法全面解析（C++初学面向对象编程）_c++ 构造函数-CSDN博客](https://blog.csdn.net/Viewinfinitely/article/details/115017678)
[c++中在一个类中定义另一个带参数构造函数的类的对象_c++构造函数中创建对象时创建另一个对象但传递的参数还没获得怎么做-CSDN博客](https://blog.csdn.net/weixin_41038905/article/details/81043553)






