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







