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






