**struct与class使用{}初始化**

* struct与class若是定义了构造函数，则都不能用大括号进行初始化
* struct若没有定义构造函数，可以用大括号初始化
* class若没有定义构造函数，且所有成员变量都为public，可以用大括号初始化

**struct在C与C++中的区别**

* 在C中，struct是用户自定义数据类型（UDT）；在C++中，struct是抽象数据类型（ADT），支持成员函数的定义
* C中的struct是没有权限设置的，可以封装数据却不可以隐藏数据，而且成员不可以是函数；C++中struct增加了访问权限，且可以和类一样有成员函数
* 在C中struct不可以继承；C++中struct可以进行复杂的继承甚至多重继承，一个struct可以继承自一个class，反之亦可
* 在定义结构体与定义结构体变量时的区别：
```cpp
// 定义C结构体
struct A{
  int a;
};
// 定义C结构体变量
struct A aa;

// 或者
// 定义C结构体
typedef struct{
  int a;
}A;
// 定义C结构体变量
A aa;

// 定义C++结构体
struct A{
  int a;
};
// 定义C++结构体变量
A aa;
```

**C++中struct与class的区别**

* class中成员的默认访问权限与默认继承方式都是private的，而struct中则是public的
* class作为关键字还用于定义模板参数，就像typename，但关键字struct不用于定义模板参数

除了这两点，struct和class基本就是一个东西，使用上没有任何其它区别。

**参考链接**

[C++的类和C里面的struct有什么区别](http://blog.csdn.net/wfq_1985/article/details/5390398)</br>
[struct 区别 在C 和C++ 中](http://www.cnblogs.com/chip/p/4955137.html)
