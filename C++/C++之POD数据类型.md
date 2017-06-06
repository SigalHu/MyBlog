关于什么是POD数据类型，网上相关的博文很多，我们知道，POD数据类型主要用来解决C\+\+与C之间数据类型的兼容性，以实现C\+\+程序与C函数的交互。当我们想要在不同进程间传递数据时，也会考虑所使用的数据类型是不是POD的。但是不是在以上情况时，只能使用POD数据类型？肯定不是的，这些才是我想讨论的主要内容。

### POD数据类型

C++11中把POD分为了两个基本概念的集合，即：平凡的（trival）和标准布局的（standard layout）。只有满足这两个基本概念才能称为是POD类型。

一个trivial class或者struct应该符合以下定义：
* 拥有平凡的默认构造函数（trivial constructor）和析构函数（trivial destructor）
* 拥有平凡的复制构造函数（trivial copy constructor）和移动构造函数（trivial move constructor）
* 拥有平凡的复制赋值运算符（trivial assignment operator）和移动赋值运算符（trivial move operator）
* 不能包含虚拟函数和虚拟基类

trivial constructor就是说构造函数什么都不干，通常情况下，不定义类的构造函数, 编译器就会自动生成一个trivial constructor，而一旦定义了构造函数, 即使构造函数中不包含任何参数，函数体中也没有任何代码，那么该构造函数都不再是trivial的，但是**我们可以使用C++11中的新的关键字 *=defualt* 来显式的声明缺省版本的构造函数从而使了类型恢复平凡化。**第二、三个规则也类似。
```cpp
#include <iostream>
using namespace std;

class A1 {};

class A2 {
public:
    A2(){}
};

class A3 {
public:
    A3() = default;
    A3(int a3){}
};

int main() {
    cout<<boolalpha<<is_trivial<A1>::value<<endl;
    cout<<boolalpha<<is_trivial<A2>::value<<endl;
    cout<<boolalpha<<is_trivial<A3>::value<<endl;
    return 0;
}
```
运行结果：
```
true
false
true
```

一个standard layout的class或者struct应该符合以下定义：
* 所有非静态成员都有相同的访问权限（public, private, protected）
* 在class或者struct继承时，满足以下两种情况之一的class或者struct也是标准布局的：
    * 派生类中有非静态成员，且只有一个仅包含静态成员的基类
    * 基类有非静态成员，而派生类没有非静态成员
* 类中第一个非静态成员的类型与其基类不同
* 没有虚拟函数和虚基类
* 所有非静态数据成员均符合标准布局类型，其基类也符合标准布局

第三个规则实际上是C\+\+中允许优化不包含成员的基类而产生的，在C\+\+标准中，如果基类没有成员，标准允许派生类的第一个成员与基类共享地址，基类并没有占据任何实际的空间，但此时若该派生类的第一个成员类型仍然是基类，编译器仍会为基类分配1字节的空间，这是因为C\+\+标准要求类型相同的对象必须地址不同，所以C\+\+11标准强制要求POD类型的派生类的第一个非静态成员的类型必须不同于基类。
```cpp
#include <iostream>
using namespace std;

class A1 {};
class A2 {};

class B1:public A1 {
public:
    A1 a1;
    int b1;
};

class B2:public A1 {
public:
    A2 a2;
    int b2;
};

class B3:public A1 {
public:
    int b3;
    A1 a1;
};

int main() {
    B1 b1;b1.b1=0xb1;
    B2 b2;b2.b2=0xb2;
    B3 b3;b3.b3=0xb3;

    cout<<"sizeof(b1)="<<sizeof(b1)<<endl;
    cout<<"&b1   ="<<&b1<<endl;
    cout<<"&b1.a1="<<&b1.a1<<endl;
    cout<<"&b1.b1="<<&b1.b1<<endl<<endl;
    cout<<"sizeof(b2)="<<sizeof(b2)<<endl;
    cout<<"&b2   ="<<&b2<<endl;
    cout<<"&b2.a2="<<&b2.a2<<endl;
    cout<<"&b2.b2="<<&b2.b2<<endl<<endl;
    cout<<"sizeof(b3)="<<sizeof(b3)<<endl;
    cout<<"&b3   ="<<&b3<<endl;
    cout<<"&b3.b3="<<&b3.b3<<endl;
    cout<<"&b3.a1="<<&b3.a1<<endl<<endl;

    cout<<boolalpha<<is_standard_layout<B1>::value<<endl;
    cout<<boolalpha<<is_standard_layout<B2>::value<<endl;
    cout<<boolalpha<<is_standard_layout<B3>::value<<endl;
    return 0;
}
```
运行结果：
```
sizeof(b1)=8
&b1   =0x28ff28
&b1.a1=0x28ff29
&b1.b1=0x28ff2c

sizeof(b2)=8
&b2   =0x28ff20
&b2.a2=0x28ff20
&b2.b2=0x28ff24

sizeof(b3)=8
&b3   =0x28ff18
&b3.b3=0x28ff18
&b3.a1=0x28ff1c

false
true
true
```

### 调用C函数时非POD数据类型与POD数据类型的转换

我们知道POD数据类型是为了解决C\+\+与C之间数据类型的兼容性，但并不是说非POD数据类型在C\+\+与C之间就不兼容。在调用C函数时，我们所传的实参也可以是非POD的，例如在如下代码中，我们可以将指向类A实例的指针转换为B类型并取值来实现非POD到POD类型的赋值，换句话说，我们完全可以以类B的内存布局去读取类A实例在栈中的内存空间。
```cpp
#include <iostream>
using namespace std;

class A {
private:
    int a1;
public:
    int a2;
    A(int a1,int a2) : a1(a1),a2(a2) {}
    int getA1() { return a1; }
};

class B {
public:
    int b;
};

int main() {
    A a(0xa1,0xa2);
    B b = *(B *)&a;
    cout << boolalpha << is_pod<A>::value << endl;
    cout << boolalpha << is_pod<B>::value << endl;
    cout << hex << a.getA1() << dec << endl;
    cout << hex << b.b << dec << endl;
    cout << hex << a.a2 << dec << endl;
    b = *((B *)&a+1);
    cout << hex << b.b << dec << endl;
    return 0;
}
```
测试结果：
```
false
true
a1 a1
a2 a2
```



很多博文都提到可以使用memcpy()和memset()对POD数据类型进行相关操作，但并不是说非POD数据类型就不可以了。

**同一进程中**



**不同进程间**

```cpp
#include <iostream>
#include "test1.h"
#include <string.h>
using namespace std;

class A{
public:
    A(int a,int b){this->a=a;this->b=b;}
    A(int a){this->a=a;}
    int a;
    int getB(){return b;}
    void setB(int b){this->b=b;}
    virtual void fun(){
        cout<<"classA"<<endl;
    }
private:
    int b;
};

class B{
public:
    B(int a,int b){this->a=a;this->b=b;}
    int a;
    int getB(){return b;}
    void setB(int b){this->b=b;}
    int b;
    virtual void fun(){
        cout<<"classB"<<endl;
    }
};

int main() {
    cout<<is_pod<A>::value<<endl;
    cout<<is_pod<B>::value<<endl;
    A a(0xa,0xb);
    cout<<sizeof(a)<<endl;
    char *b = new char[sizeof(a)];
    memcpy(b,&a,sizeof(a));
//    a.fun();
    ((B *)b)->fun();
    cout<<a.a<<endl;
    cout<<((B *)b)->a<<endl;
    cout<<a.getB()<<endl;
    cout<<((B *)b)->b<<endl;
    delete b;
    return 0;
}
```

**参考链接**

[C++11：POD数据类型](http://blog.csdn.net/aqtata/article/details/35618709)</br>
[C++中POD是什么？](http://blog.csdn.net/eijnew/article/details/7369253)</br>
[C++11中的POD和Trivial](http://blog.csdn.net/dashuniuniu/article/details/50042341)</br>
[C++11中的POD类型](http://www.codeweblog.com/c-11中的pod类型/)</br>
[C++11 POD类型](http://www.cnblogs.com/DswCnblog/p/6371071.html)
