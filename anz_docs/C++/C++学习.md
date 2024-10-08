## 基础知识
### 构造函数
这一块暂时可以不深究，构造函数分带参和不带参的，带参的可以用初始化参数列表的形式来写：
![[Pasted image 20240418171149.png]]
**注意**：如果在一个类里，调用另一个类的对象的构造函数，只能写入初始化参数列表
[C++构造函数的各种用法全面解析（C++初学面向对象编程）_c++ 构造函数-CSDN博客](https://blog.csdn.net/Viewinfinitely/article/details/115017678)
[c++中在一个类中定义另一个带参数构造函数的类的对象_c++构造函数中创建对象时创建另一个对象但传递的参数还没获得怎么做-CSDN博客](https://blog.csdn.net/weixin_41038905/article/details/81043553)

### 初始化
C++ 11可以使用{}列表初始化
``` C++
 vector <int>a={1,2,3};//a初始化为1，2，3
 vector <int>a{3}；//a初始化为一个值3

```

### 遍历
```C++
//第一种用法
void showvec(const vector<int>& line) {
  for (auto iter = line.cbegin(); iter != line.cend(); iter++) {
    cout << (*iter) << endl;
  }
}

//第二种用法
for (auto lin : line) {
    cout << lin;
  }
//可以用auto直接遍历
```
auto 即 for(auto x: range) 这样会拷贝一份 range 元素，而不会改变 range 中元素;
当需要修改range中元素，用 for(auto& x: range);
当只想读取 range 中元素时，使用 const auto&，如：for(const auto&x:range)，它不会进行拷贝，也不会修改range;
当需要拷贝元素，但不可修改拷贝出来的值时，使用 for(const auto x:range)，避免拷贝开销

[C++:指针和new，delete详解_delete 指针-CSDN博客](https://blog.csdn.net/qq_45853229/article/details/116499668)
new一个东西必须得配上指针

