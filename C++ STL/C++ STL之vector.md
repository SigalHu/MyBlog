### 介绍

`vector`是表示可变大小数组的序列容器。

就像数组一样，`vector`也采用的连续存储空间来存储元素。也就意味着使用迭代器、下标或指针偏移的方式对`vector`的元素进行访问，和数组一样高效。但是又不像数组，它的大小是可以动态改变的，而且它的大小会被容器自动处理。

本质讲，`vector`使用动态分配数组来存储它的元素。当新元素插入时候，这个数组需要被重新分配大小为了增加存储空间。其做法是，分配一个新的数组，然后将全部元素移到这个数组。就时间而言，这是一个相对代价高的任务，因为每当一个新的元素加入到容器的时候，`vector`并不会每次都重新分配大小。

`vector`分配空间策略：`vector`会分配一些额外的空间以适应可能的增长，因为存储空间比实际需要的存储空间更大。不同的库采用不同的策略权衡空间的使用和重新分配。但是无论如何，重新分配都应该是对数增长的间隔大小，以至于在末尾插入一个元素的时候是在常数时间的复杂度完成的。

所以，`vector`的大小与`vector`的容量是有区别的，大小是指元素的个数，容量是指分配的内存大小。`vector::size()`返回`vector`的大小，`vector::capacity()`返回容量值。若要自己指定分配的容量大小，则可以使用`vector::reserve()`，但规定的值要大于`size()`值。

因此，`vector`占用了更多的存储空间，为了获得管理存储空间的能力，并且以一种有效的方式动态增长。

与其它动态序列容器相比（deques, lists and forward_lists）， `vector`在访问元素的时候更加高效，在末尾添加和删除元素相对高效。对于其它不在末尾的删除和插入操作，效率更低。比起lists和forward_lists统一的迭代器和引用更好。

### vector的用法

#### 头文件
```cpp
#include <vector>
```
`vector`属于`std`命名域的，建议使用全局的命名域方式：
```cpp
using namespace std;
```

#### 构造函数与析构函数

| 操作 | 效果 |
| -- | -- |
| `vector<T> c` | Default构造函数，产生一个空`vector`，没有任何元素 |
| `vector<T> c(c2)` | Copy构造函数，建立`c2`的同型`vector`并成为`c2`的一份拷贝（所有元素都被复制） |
| `vector<T> c=c2` | Copy构造函数，建立一个新的`vector`作为`c2`的拷贝（每个元素都被复制） |
| `vector<T> c(rv)` | Move构造函数，建立一个新的`vector`，取`rvalue rv`的内容（始自C++11） |
| `vector<T> c=rv` | Move构造函数，建立一个新的`vector`，取`rvalue rv`的内容（始自C++11） |
| `vector<T> c(n)` | 利用元素的Default构造函数生成一个大小为`n`的`vector` |
| `vector<T> c(n,val)` | 建立一个大小为`n`的`vector`，每个元素值都是`val` |
| `vector<T> c(beg,end)` | 建立一个`vector`，以区间`[begin,end)`作为元素初值 |
| `vector<T> c(initlist)` | 建立一个`vector`，以初值列`initlist`的元素为初值（始自C++11） |
| `vector<T> c=initlist` | 建立一个`vector`，以初值列`initlist`的元素为初值（始自C++11） |
| `c.~vector()` | 销毁所有元素，释放内存 |


```cpp
// v1是一个空vector，它潜在的元素是int类型的，执行默认初始化
vector<int> v1;
// v2中包含有v1所有元素的副本
vector<int> v2(v1);
// 等价于v3(v2)，v3中包含有v2所有元素的副本
vector<int> v3 = v2;
// v4中包含了2个重复元素，每个元素的值都为4
vector<int> v4(2,4);
// v5中包含了2个重复地执行了值初始化的对象
vector<int> v5(2);
// v6包含了初始值个数的元素，每个元素被赋予相应的初始值
vector<int> v6{5,6};
// 等价于v7{6,7}
vector<int> v7 = {6,7};
// 用向量v7的第0个到第1个值初始化v8
vector<int> v8(v7.begin(),v7.begin()+2);
// 用数组a的第0个到第1个值初始化v9
array<int,2> a = {1,2};
vector<int> v9(a.begin(),a.begin()+2);
// 用数组b的第0个到第1个值初始化v10与v11
int b[] = {3,4};
vector<int> v10(b,b+2);
vector<int> v11(&b[0],&b[2]);
```
#### 非更易型操作

#### 算法

**遍历**

**查找**

**排序**

**参考链接**

[Vector的用法](http://blog.csdn.net/lsh_2013/article/details/21191289)</br>
[C++ STL之vector用法总结](http://www.cnblogs.com/zhonghuasong/p/5975979.html)</br>
[C++STL中vector容器的用法](http://www.cnblogs.com/ziyi--caolu/archive/2013/07/04/3170928.html)
