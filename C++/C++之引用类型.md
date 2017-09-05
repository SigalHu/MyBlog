我们知道，引用相当于给对象取一个别名，通过该别名，我们可以对该对象进行相关操作，由于引用依托于其他对象而存在，所以引用必须初始化，且一旦确定引用对象就不可修改。引用可分为左值引用与右值引用，下面先来介绍一下什么是左值与右值。

### 左值与右值

C\+\+中所有的表达式和变量要么是左值，要么是右值。通俗的左值的定义就是非临时对象，那些可以在多条语句中使用的对象。所有的变量都满足这个定义，在多条代码中都可以使用，都是左值。右值是指临时的对象，它们只在当前的语句中有效。

* 简单的赋值语句
```cpp
int i = 0;
```
在这条语句中，i是左值，0是临时值，就是右值。在下面的代码中，i可以被引用，0就不可以了。立即数都是右值。

* 右值也可以出现在赋值表达式的左边，但是不能作为赋值的对象，因为右值只在当前语句有效，赋值没有意义。
```cpp
((i>0) ? i : j) = 1;
```
在这个例子中，0作为右值出现在了=的左边。但是赋值对象是i或者 j，都是左值。

### 左值引用

左值引用又可分为非常量左值引用与常量左值引用，其声明符号都为&。

**非常量左值引用**

非常量左值引用所引用的对象必须为非临时对象，因此其本身不占存储单元，对其求址就是对该对象求址。
```cpp
int a = 1;
int *pa = &a;

int &a1 = 1;  // 错误，立即数1为临时对象
int &a2 = a;
int *&a3 = &a;  // 错误，临时对象int *tmp=&a;int *&a3=tmp;
int *&a4 = pa;
int &a5 = a++;  // 错误，临时对象int tmp=a+1;int &a5=tmp;
int &a6 = ++a;  // 正确，非临时对象a=a+1;int &a6=a;
int &a7 = a = a++;  // 正确，非临时对象a=a;int &a7=a;
int &a8 = a*a;  // 错误，临时对象int tmp=a*a;int &a8=tmp;
int &a9 = a = a*a;  // 正确，非临时对象a=a*a;int &a9=a;
unsigned int &a10 = a;  // 错误，临时对象unsigned int tmp=a;unsigned int &a10 = tmp;
```

**常量左值引用**

常量左值引用所引用的对象可以是非临时对象，也可以是临时对象。当所引用的对象为临时对象时，系统会为其分配内存空间，此时该引用不再是临时对象。经过const限定，我们无法通过该别名对所引用的对象进行修改。在下面的代码中，所有的引用都是正确的，另外，通过输出结果，我们可以直观的看到a\+\+与\+\+a在引用上的区别。
```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 1;
    int *pa = &a;

    const int &a1 = 1;
    const int &a2 = a;
    int * const &a3 = &a;
    int * const &a4 = pa;
    const int &a5 = a++;
    const int &a6 = ++a;
    const int &a7 = a*a;
    const unsigned int &a8 = a;
    cout << "a =" << a <<endl;
    cout << "a5=" << a5 << endl;
    cout << "a6=" << a6 << endl;
    return 0;
}
```
运行结果：
```
a =3
a5=1
a6=3
```

### 右值引用

右值引用也同样分为非常量右值引用与常量右值引用，其声明符号都为&&。

**非常量右值引用**

非常量右值引用所引用的对象必须为临时对象，系统会为其分配内存空间，因此，右值引用是非临时对象。
```cpp
int a = 1;
int *pa = &a;

int &&a1 = 1;
int &&a2 = a;  // 错误，a为非临时对象
int *&&a3 = &a;  // 正确，临时对象int *tmp=&a;int *&&a3=tmp;
int *&&a4 = pa;  // 错误，pa为非临时对象
int &&a5 = a++;  // 正确，临时对象int tmp=a+1;int &&a5=tmp;
int &&a6 = ++a;  // 错误，非临时对象a=a+1;int &&a6=a;
int &&a7 = a = a++;  // 错误，非临时对象a=a;int &&a7=a;
int &&a8 = a*a;  // 正确，临时对象int tmp=a*a;int &&a8=tmp;
int &&a9 = a = a*a;  // 错误，非临时对象a=a*a;int &&a9=a;
unsigned int &&a10 = a;  // 正确，临时对象unsigned int tmp=a;unsigned int &&a10 = tmp;
```

**常量右值引用**

常量右值引用与常量左值引用不同，只能引用临时对象。系统会为其分配内存空间，此时该引用不再是临时对象。经过const限定，我们无法通过该别名对所引用的对象进行修改。
```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 1;
    int *pa = &a;

    const int &&a1 = 1;
//    const int &&a2 = a;  // 错误，a为非临时对象
    int * const &&a3 = &a;  // 正确，临时对象int *tmp=&a;int * const &&a3=tmp;
//    int * const &&a4 = pa;  // 错误，pa为非临时对象
    const int &&a5 = a++;  // 正确，临时对象int tmp=a+1;const int &&a5=tmp;
//    const int &&a6 = ++a;  // 错误，非临时对象a=a+1;const int &&a6=a;
    const int &&a7 = a*a;  // 正确，临时对象int tmp=a*a;const int &&a7=tmp;
    const unsigned int &&a8 = a;  // 正确，临时对象unsigned int tmp=a;const unsigned int &&a8 = tmp;
    cout << "a =" << a <<endl;
    cout << "a5=" << a5 << endl;
//    cout << "a6=" << a6 << endl;
    return 0;
}
```
运行结果：
```
a =2
a5=1
```

### 补充

* 在常量左值引用时，我们虽然不能通过别名改变所引用的非临时对象的值，但可以直接通过非临时对象的变量名进行修改
```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 1;
    const int &b = a;
    cout<<"&a="<<&a<<" a="<<a<<endl;
    cout<<"&b="<<&b<<" b="<<b<<endl<<endl;
//    b++;  // 错误
    a++;
    cout<<"&a="<<&a<<" a="<<a<<endl;
    cout<<"&b="<<&b<<" b="<<b<<endl;
    return 0;
}
```
运行结果：
```
&a=0x28ff28 a=1
&b=0x28ff28 b=1

&a=0x28ff28 a=2
&b=0x28ff28 b=2
```

* 可以给数组建立引用，因为数组名本质上就是地址（立即数）
```cpp
int a[] = { 1, 2 };
int *pa = a;

int *&b = a;  // 错误
int *&c = pa;
int * const &d = a;
int *&&e = a;
int * const &&f = a;
```

* 可以返回函数内部new分配的内存的引用，但不建议这么做，因为一不小心就会造成内存泄漏。另外，返回指针引用在一些编译器下会导致运行错误
```cpp
#include <iostream>
using namespace std;

int& fun(){
    int *a = new int(1);
    return *a;
}

int main() {
    int &a1 = fun();
    int a2 = fun();
    cout << "&a1=" << &a1 << " | a1=" << a1 << endl;
    cout << "&a2=" << &a2 << " | a2=" << a2 << endl;
    delete &a1;
//    delete &a2;
    cout << "&a1=" << &a1 << " | a1=" << a1 << endl;
    cout << "&a2=" << &a2 << " | a2=" << a2 << endl;
    return 0;
}
```
运行结果：
```
&a1=0x4915d0 | a1=1
&a2=0x28ff28 | a2=1
&a1=0x4915d0 | a1=4789744
&a2=0x28ff28 | a2=1
```

* 类成员变量也可以是引用类型，但必须在构造函数的初始化列表中进行初始化，同样不建议这么做，因为很容易出错，而且右值引用在有些编译器下赋值不成功，读取实例内存可能导致电脑蓝屏。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int &a;
    int &&b;
    A(const int &a, int b) :a(const_cast<int &>(a)), b(b++){}
};

int main() {
    A a(0xa, 0xb);
    cout << sizeof(a) << endl;
    cout << &a.a << ":" << a.a << endl;
    cout << &a.b << ":" << a.b << endl;
    return 0;
}
```
运行结果：
```
8
0x28ff2c:10
0x28ff04:11
```

* 引用是除指针外另一个可以产生多态效果的手段。这意味着，一个基类的引用可以指向它的派生类实例。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    virtual void fun(){
        cout<<"A::fun()"<<endl;
    }
};

class B:public A{
public:
    virtual void fun(){
        cout<<"B::fun()"<<endl;
    }
};

int main() {
    A &&b = B();
    b.fun();
    return 0;
}
```
运行结果：
```
B::fun()
```

**参考链接**

[C++11 标准新特性: 右值引用与转移语义](https://www.ibm.com/developerworks/cn/aix/library/1307_lisl_c11/)</br>
[C++引用详解](http://www.cnblogs.com/gw811/archive/2012/10/20/2732687.html)
