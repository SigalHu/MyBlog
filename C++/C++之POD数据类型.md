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

### 非POD数据类型与POD数据类型的转换

我们知道POD数据类型是为了解决C\+\+与C之间数据类型的兼容性，但并不是说非POD数据类型在C\+\+与C之间就不兼容。在调用C函数时，我们所传的实参也可以是非POD的，例如在如下代码中，我们可以将指向类A实例的指针转换为B类型并取值来实现非POD到POD类型的赋值，换句话说，我们完全可以以类B的内存分配方式去读取类A实例在栈中的内存空间。
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
    cout<<boolalpha<<is_pod<A>::value<<endl;
    cout<<boolalpha<<is_pod<B>::value<<endl;
    cout<<hex<<a.getA1()<<" "<<b.b<<dec<<endl;
    b = *((B *)&a+1);
    cout<<hex<<a.a2<<" "<<b.b<<dec<<endl;
    a = *(A *)&b;
    cout<<hex<<a.getA1()<<" "<<b.b<<dec<<endl;
    return 0;
}
```
测试结果：
```
false
true
a1 a1
a2 a2
a2 a2
```
从测试结果可以看到，这种方式会破坏类的封装特性，而且当类A与B的内存布局不相同时，也不会报错，说白了就是将该指针所指向的栈中的数据以某种数据类型的方式取值。当然，我们也可以通过合理地重载类A的赋值运算符来实现相应的需求。

当类中包含虚函数时，情况又会有些不同。我们知道，当类中包含虚函数时，在该类实例的所占内存的起始位置会存放一个指向该类虚函数表的指针，例如在以下代码中，我们将指向实例a的指针转换为B1的指针并取值，得到的b1.a是a中指向类A虚函数表的指针，我们也通过存放在虚函数表中的函数指针加以验证，b1.b的值等于a.a，这些都是符合类的内存分配方式的。
```cpp
#include <iostream>
#include <string.h>
using namespace std;

class A{
public:
    int a;
    A(int a,int b):a(a),b(b){}
private:
    int b;
    virtual void funA1(){
        cout<<"funA1"<<endl;
    }
    virtual void funA2(){
        cout<<"funA2"<<endl;
    }
};

class B1{
public:
    int a;
    int b;
};

class B2{
public:
    int a;
    int b;
    virtual void funB1(){
        cout<<"funB1"<<endl;
    }
    virtual void funB2(){
        cout<<"funB2"<<endl;
    }
};

int main() {
    cout<<boolalpha<<is_pod<A>::value<<endl;
    cout<<boolalpha<<is_pod<B1>::value<<endl;
    cout<<boolalpha<<is_pod<B2>::value<<endl<<endl;

    A a(0xa,0xb);B1 b1;B2 b2;
    b1 = *(B1 *)&a;
    cout<<hex<<b1.a<<" "<<b1.b<<dec<<endl;
    typedef void (*Fun)();
    ((Fun)*(unsigned int *)b1.a)();
    ((Fun)*((unsigned int *)b1.a+1))();
    b1 = *(B1 *)((unsigned int*)&a+1);
    cout<<hex<<b1.a<<" "<<b1.b<<dec<<endl<<endl;

    b2 = *(B2 *)&a;
    cout<<hex<<b2.a<<" "<<b2.b<<dec<<endl;
    ((Fun)*(unsigned int *)*(unsigned int *)&b2)();
    b2.funB2();
    B2 *pb2=&b2;
    pb2->funB2();
    cout<<endl;

    memcpy(&b2,&a,sizeof(b2));
    cout<<hex<<b2.a<<" "<<b2.b<<dec<<endl;
    ((Fun)*(unsigned int *)*(unsigned int *)&b2)();
    b2.funB2();
    pb2=&b2;
    pb2->funB2();
    return 0;
}
```
测试结果：
```
false
true
false

406170 a
funA1
funA2
a b

a b
funB1
funB2
funB2

a b
funA1
funB2
funA2
```
在上面的代码中，将实例a赋值给b2调用了类B2的默认赋值运算符函数，故b2中指向虚函数表中的指针不变，成员变量被赋值。而使用memcpy()则将类B2大小的数据整个复制到b2的内存空间，此时b2中的指向类B2的虚函数表指针被替换成指向类A的虚函数表指针。此时，通过实例b2仍然可以正确的调用虚函数B2::funB2()，而通过类B2的指针却调用的是A::funA2()。通过gdb查看这部分的汇编代码，使用实例b2调用虚函数时，直接通过函数指针调用函数，并没有通过虚函数表；而使用指针调用虚函数时，则是在调用时从虚函数表的确定位置处获取函数指针。
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main():
37	int main() {
...
62	    b2.funB2();
   0x00401540 <+224>:	lea    -0x3c(%ebp),%eax
   0x00401543 <+227>:	mov    %eax,%ecx
   0x00401545 <+229>:	call   0x403d50 <B2::funB2()>

63	    pb2=&b2;
   0x0040154a <+234>:	lea    -0x3c(%ebp),%eax
   0x0040154d <+237>:	mov    %eax,-0x1c(%ebp)

64	    pb2->funB2();
   0x00401550 <+240>:	mov    -0x1c(%ebp),%eax
   0x00401553 <+243>:	mov    (%eax),%eax
   0x00401555 <+245>:	add    $0x4,%eax
   0x00401558 <+248>:	mov    (%eax),%eax
   0x0040155a <+250>:	mov    -0x1c(%ebp),%edx
   0x0040155d <+253>:	mov    %edx,%ecx
   0x0040155f <+255>:	call   *%eax
...
End of assembler dump.
```

### 总结

1. 使用memcpy()通过类实例指针操作的是类实例在栈中的数据，所以只要搞清楚类的内存分配，我们总是可以获取自己想要的数据。
2. 当类中包含指针时，对其进行赋值或者memcpy()操作需注意内存的释放与野指针的产生
3. 当类中包含虚函数或虚继承时，类实例的内存空间中会包含指向虚函数表的指针
4. 相同的指针（虚拟地址）被不同的进程映射到的物理地址是不同的，也就是说，在不同进程间传递指针是没有意义的

**参考链接**

[C++11：POD数据类型](http://blog.csdn.net/aqtata/article/details/35618709)</br>
[C++中POD是什么？](http://blog.csdn.net/eijnew/article/details/7369253)</br>
[C++11中的POD和Trivial](http://blog.csdn.net/dashuniuniu/article/details/50042341)</br>
[C++11中的POD类型](http://www.codeweblog.com/c-11中的pod类型/)</br>
[C++11 POD类型](http://www.cnblogs.com/DswCnblog/p/6371071.html)
