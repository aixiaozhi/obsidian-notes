```C++
int *x=&a;
//切记，理解成x是a的地址，而不是*x是a的地址
不能写成 *x=&a 因为这个本意是把x指向的内容赋予a的地址，只能用int**;
记住，没有**那么永远只有一层地址套壳

定义的时候
int &x=y，这时候是引用，不是取地址符
```

![[Pasted image 20240817190345.png]]
static作用的只会初始化一次，在全局区，这里要注意{}外i在全局区
算k 13行后 a=3,k=k+5=10;
15行后 k=k+4=14；a=4


### C++ const
```C++
const int  a = 7; 
int  b = a; // 正确
a = 8;
```
修饰常量可以当右值不能当左值

```C++
const int *p = 8;//8的值不能变 也就是*p是恒定的
//
int* const p=&a;
*p = 9; // 正确
int  b = 7;
p = &b; // 这就错误了，P本身的地址是不能变的
```

### C++ 堆/栈，全局变量等
1. 栈一般指的函数内存放的变量，存放参数值，局部变量
2. 堆一般是程序员分配，释放，结束时候可能由OS收回
3. 全局区(静态区)(static)
4. 文字常量区
5. 程序代码区
```C++
int a=0;//全局初始化区
char *p1;//全局未初始化区
main()
{
int b;//栈
char *p3="123456";//p3在栈，123456在常量区
static int c=0;//全局（静态）初始化区
p1=(char*)malloc(10); //10字节在堆区
}
```

### C++ for 循环
```C++
	vector<int> vec;
	for (int var : vec) {
		cout << var << endl;
	}
var直接是 vec中的值，建议直接auto

例如：
list<int> a1;
for(auto a:a1){
    cout<<a<<endl;
}

迭代器指向的地址是不变的，比如b现在指向begin，实际是指向了1，如果从头插入其实他还是指向1，而不是begin了

int main(){
list<int> a1={1,2,3,4};
LIit b =a1.begin();
a1.push_front(5);
cout<<*b<<endl;
return 0;
}
//输出的是1
```

### C++ list
```C++
splice可以用于拼接
list1.splice(iterator position, list2)
list1.splice(iterator position, list2, iterator i)

mylist.splice(mylist.begin(), mylist, needit);
//很好用，能迅速拼接到开头
如果needit指向原来的元素的话，现在指向begin了
```

### C++三大特性
**封装**：突破了C语言对于函数的限制，封装可以隐藏实现细节，从而使得代码能够模块化。
**继承**：继承可以通过扩展已存在的代码模块（类），达到代码重用的目的。原有的类称为基类或父类，产生的新类称为派生类或子类。

### 寒武纪8.19
```C++
不知道数组大小的时候就用vector，知道的时候就用 int* a=new int[10]
红黑树
基本数据结构
```
![[Pasted image 20240819184838.png|500]]
![[Pasted image 20240819185718.png|500]]
![[Pasted image 20240819185932.png|500]]

### C++模板

**模板的出现是为了解决频繁书写函数重载的问题**
1. 函数模板：使用泛型参数的函数
2. 类模板：使用泛型参数的类
#### 函数模板
```C++
 template<typename T1, typename T2,......,typename Tn>
 返回值类型 函数名(参数列表)
 {   
 //……
 }    
 注意：typename是用来定义模板参数关键字，也可以使用class
 例如：
   
 template<typename T>
void Swap(T& left, T& right)
{
T temp = left;
left = right;
right = temp;
}
使用的时候分为显式实例化和隐式实例化
比如 Swap(a,b)就是隐式
Swap<int>(a,b)就是显示
```
#### 类模板
```C++
类模板是对成员数据类型不同的类的抽象，它说明了类的定义规则，一个类模板可以生成多种具体的类。与函数模板的定义形式类似， 类模板也是使用template关键字和尖括号“<>”中的模板形参进行说明，类的定义形式与普通类相同。
template<class T1, class T2, ..., class Tn>
class 类模板名
{  
// 类内成员定义
}; 
   
   
template<class T1>
class Stack
{   
public:
	// 构造函数
	Stack(int capacity = 4)
		:_a(new T1[capacity])
		,_capacity(capacity)
		,_size(0)
	{}
     
	void Push(T1 data)
	{
		_a[_size] = data;
		_size++;
	}
	// ...其他方法
	// 析构函数
	~Stack()
	{   
		delete[]_a;
		_a = nullptr;
		_capacity = _size = 0;
	}
private:
	T1* _a;
	int _capacity;
	int _size;
}; 
   
int main()
{    
	Stack<int> s1;
	Stack<double> s2;
	return 0;
}
```
类模板实例化需要在类模板名字后跟**`<>`**(显式例化）**，然后将实例化的类型放在中即可，**类模板名字不是真正的类，而实例化的结果才是真正的类**
**类模板，是不支持，声明，定义，测试分开写的，会出现链接编译错误**，定义和测试放在一起写就好了

### C++ . 和 ->
总的来说.的左边必须是实体，->的左边是指针，不能星指针.

### C++ unordered_map
底层是一个哈希表，通过给定的关键字访问到具体值的一个数据结构
1. 直接寻址法：某个线性函数
2. 数字分析法：通过分析数据找到冲突较小的部分
3. 除留取余
哈希冲突了怎么办？第一，可以继续哈希。第二，可以
![[Pasted image 20240829222431.png|500]]
```C++
//查找值的时间复杂度是O(1)，是hashmap
#include<unordered_map>
unordered_map<int,node*> map;
//查找直接用
if(map.find(addr)!=map.end()
//unordered_map的好处是 其查找的复杂度是O(1),无论查找什么key
```


### C++虚拟地址空间是什么，有什么好处
1. 进程无法访问到操作系统禁止访问的物理地址，也不能访问别的进程的地址空间，大大增强了程序的安全性

### C++虚函数
C++一个重要特性，继承要搭配虚函数来使用，实现多态，子类可以调用父类的函数，这个叫继承了调用权，有三种函数，非虚函数是不希望子类重新定义它，虚函数是你希望派生类重新定义他，并且你有一个默认定义。纯虚函数是你希望子类一定要去定义它，他没有默认的定义
```C++
class A{
virtual void draw()=0;//=0为纯虚函数
virtual void A(int a){};//虚函数
int A(int a){};//非虚函数
}
```
函数重载只是参数列表不用，定义的是我在使用相同函数的时候如果参数列表是不用的，会达到不同的效果，比如说构造函数是赋予全值还是默认值。

### C++堆栈内存空间
堆：存在于某一作用域的内存空间，例如调用函数，函数本身会形成一个堆来放置其所接受的参数，及返回地址，在函数本身内声明的任何变量，其所使用的内存堆都来自上述堆
栈：操作系统提供的一块global的空间，程序可以动态分配某些区块（new)，new的时候自己有责任去释放这一块内存空间.new之后 当作用域结束，指针p的生命周期结束的，但是分配的这一块空间还是在这，这就叫**内存泄漏**
堆释放会调用析构函数
```C++
class complex{...};

{
Complex c1(1,2);//c1的空间来自于堆
Complex* p=new complex(1,2) //p的空间来自于栈
static complex c2(1,2) //生命在作用域结束之后仍然存在，直到整个程序结束
}

//下面新起的
class complex{};
complex c3(1,2);//写在任何作用域之外，是全局对象
int main(){
...
}

{
complex *p=new complex;
}
```

### C++ struct 和 class的区别
1. struct默认的访问权限是public，class的默认权限是private
2. 都可以有成员函数，但是类偏向实现类的行为和功能，结构体是为了实现一些简单的操作
3. 使用场景：结构体一般是存储一组相关的数据，没有复杂的操作和逻辑，轻量容器；类一般是实现更复杂的数据结构，需要封装行为啊






