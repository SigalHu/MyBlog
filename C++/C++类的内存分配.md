### 内存对齐

**内存对齐的原因**

1. **平台原因（移植原因）：** 不是所有的硬件平台都能访问任意地址上的任意数据的；某些硬件平台只能在某些地址处取某些特定类型的数据，否则抛出硬件异常。
2. **性能原因：** 数据结构（尤其是栈）应该尽可能地在自然边界上对齐。原因在于，为了访问未对齐的内存，处理器需要作两次内存访问；而对齐的内存访问仅需要一次访问。

**对齐规则**

每个特定平台上的编译器都有自己的默认“对齐系数”（也叫对齐模数）。程序员可以通过预编译命令#pragma pack(n)，n=1,2,4,8,16来改变这一系数，其中的n就是你要指定的“对齐系数”。

1. **数据成员对齐规则：** 类（class）、结构体（struct）或联合体（union）的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员的对齐按照#pragma pack指定的数值和这个数据成员自身长度中，比较小的那个进行。
2. **整体对齐规则：** 在数据成员完成各自对齐之后，类、结构体或联合体本身也要进行对齐，对齐将按照#pragma pack指定的数值和类、结构体或联合体最大数据成员长度中，比较小的那个进行。

结合1、2可推断，当#pragma pack的n值等于或超过所有数据成员长度的时候，这个n值的大小将不产生任何效果。

### 成员变量

**测试一**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A {
};

class B {
public:
    short a;
};

class C {
public:
    short a;
    int b;
};

int main() {
    cout << "A:" << sizeof(A) << endl;
    cout << "B:" << sizeof(B) << endl;
    cout << "C:" << sizeof(C) << endl;
    return 0;
}
```
测试结果：
```
A:1
B:2
C:8
```
当为空类时，编译器会给空类分配1个字节的内存空间，这样的话，创建的实例所指向的就是有意义的内存空间。

对比类B和C，B的成员变量为short型，占2个字节，C的第一个成员变量存放在offset为0的地方，第二个成员变量占4字节，故编译器插入2个字节凑足4的整数倍，使得第二个成员变量存放在offset为4的地方。

**测试二**

测试代码：
```cpp
#include <iostream>
#include <string.h>
using namespace std;

class A1{};
class A2{};

class B1:public A1{
public:
    A1 a1;
    char ca;
};

class B2:public A1{
public:
    A2 a2;
    char ca;
};

class B3:public A1{
public:
    char ca;
    A1 a1;
};

int main() {
    cout<<sizeof(B1)<<endl;
    cout<<sizeof(B2)<<endl;
    cout<<sizeof(B3)<<endl;
    return 0;
}
```
测试结果：
```
3
2
2
```
如果基类没有成员，标准允许派生类的第一个成员与基类共享地址，基类并没有占据任何实际的空间，但此时若该派生类的第一个成员类型仍然是基类，编译器仍会为基类分配1字节的空间，这是因为C\+\+标准要求类型相同的对象必须地址不同。

**测试三**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    short a;
    int b;
    double c;
};

class B {
public:
    short a;
    double b;
    int c;
};

int main() {
    A classA;
    B classB;

    cout << "classA:" << sizeof(classA) << endl;
    cout << "classA.a:" << &classA.a << endl;
    cout << "classA.b:" << &classA.b << endl;
    cout << "classA.c:" << &classA.c << endl;

    cout << "classB:" << sizeof(classB) << endl;
    cout << "classB.a:" << &classB.a << endl;
    cout << "classB.b:" << &classB.b << endl;
    cout << "classB.c:" << &classB.c << endl;
    return 0;
}
```
测试结果：
```
classA:16
classA.a:0x28ff20
classA.b:0x28ff24
classA.c:0x28ff28
classB:24
classB.a:0x28ff08
classB.b:0x28ff10
classB.c:0x28ff18
```
类A的第一个成员变量存放在0x28ff20\~0x28ff21，第二个成员变量占4字节，offset为4，在0x28ff24\~0x28ff27，第三个成员变量占8字节，offset为8，在0x28ff28\~0x28ff2f，故类A占16字节。

类B的第一个成员变量存放在0x28ff08\~0x28ff09，第二个成员变量占8字节，offset为8，在0x28ff10\~0x28ff17，第三个成员变量占4字节，offset取4的整数倍为16，在0x28ff18\~0x28ff1b，由于类B本身也要对齐，这里最大成员变量长度为8，故类B的大小需为8的整数倍，编译器在最后插入4个字节，共占24字节。

### static关键字

测试代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    static short a;
    int b;
    double c;
};
short A::a = 0;

int main() {
    A classA;

    cout << "A:" << sizeof(A) << endl;
    cout << "A::a:" << &A::a << endl;
    cout << "classA:" << sizeof(classA) << endl;
    cout << "classA.a:" << &classA.a << endl;
    cout << "classA.b:" << &classA.b << endl;
    cout << "classA.c:" << &classA.c << endl;
    return 0;
}
```
测试结果：
```
A:16
A::a:0x407020
classA:16
classA.a:0x407020
classA.b:0x28ff20
classA.c:0x28ff28
```
类A为16字节，这是由于在该测试代码中，其静态成员变量存储在全局数据区，独立于类的实例，为所有该类实例所共享。

### #pragma pack字节对齐

测试代码：
```cpp
#include <iostream>
using namespace std;

#pragma pack(1)
class A {
public:
    char a;
    short b;
    double c;
};
#pragma pack()

#pragma pack(4)
class B1 {
public:
    char a;
    short b;
    double c;
};
#pragma pack()

#pragma pack(4)
class B2 {
public:
    char a;
    double b;
    short c;
};
#pragma pack()

#pragma pack(16)
class C {
public:
    char a;
    short b;
    double c;
};
#pragma pack()

int main() {
    A classA;
    B1 classB1;
    B2 classB2;
    C classC;

    cout << "classA:" << sizeof(classA) << endl;
    cout << "classA.a:" << (void *)&classA.a << endl;
    cout << "classA.b:" << &classA.b << endl;
    cout << "classA.c:" << &classA.c << endl;

    cout << "classB1:" << sizeof(classB1) << endl;
    cout << "classB1.a:" << (void *)&classB1.a << endl;
    cout << "classB1.b:" << &classB1.b << endl;
    cout << "classB1.c:" << &classB1.c << endl;

    cout << "classB2:" << sizeof(classB2) << endl;
    cout << "classB2.a:" << (void *)&classB2.a << endl;
    cout << "classB2.b:" << &classB2.b << endl;
    cout << "classB2.c:" << &classB2.c << endl;

    cout << "classC:" << sizeof(classC) << endl;
    cout << "classC.a:" << (void *)&classC.a << endl;
    cout << "classC.b:" << &classC.b << endl;
    cout << "classC.c:" << &classC.c << endl;
    return 0;
}
```
测试结果：
```
classA:11
classA.a:0x28ff25
classA.b:0x28ff26
classA.c:0x28ff28
classB1:12
classB1.a:0x28ff18
classB1.b:0x28ff1a
classB1.c:0x28ff1c
classB2:16
classB2.a:0x28ff08
classB2.b:0x28ff0c
classB2.c:0x28ff14
classC:16
classC.a:0x28fef8
classC.b:0x28fefa
classC.c:0x28ff00
```
类A的自定义字节对齐n=1，小于A的所有成员变量所占字节数，故A的所有成员变量依次存放，成员变量之间没有插入多余字节。类C的自定义字节对齐n=16，大于C的所有成员变量，此时不产生任何效果。

类B1的自定义字节对齐n=4，大于第二个成员变量所占字节数2，故该成员变量存放在offset为2的地方，而第三个成员变量所占字节数为8，大于n，故offset取4的整数倍为4。类B2的成员变量存放位置的确定与B1类似，但此时由于类B2所占内存为14字节不能被4（取n与最大成员变量长度（此处为8）的最小值）整除，故编译器在最后插入2个字节，使其所占内存数为16字节。

### 虚函数

**测试一**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int a;
    void fun(){}
};

class B{
public:
    int a;
    virtual void fun(){}
};

class C{
public:
    int a;
    virtual void fun1(){}
    virtual void fun2(){}
};

int main() {
    A a;a.a=1;
    B b;b.a=2;
    C c;c.a=3;
    cout<<"sizeof(a)="<<sizeof(a)<<endl;
    cout<<"sizeof(b)="<<sizeof(b)<<endl;
    cout<<"&b  ="<<&b<<endl;
    cout<<"&b.a="<<&b.a<<endl;
    cout<<"sizeof(c)="<<sizeof(c)<<endl;
    cout<<"&c  ="<<&c<<endl;
    cout<<"&c.a="<<&c.a<<endl;
    return 0;
}
```
测试结果：
```
sizeof(a)=4
sizeof(b)=8
&b  =0x28ff24
&b.a=0x28ff28
sizeof(c)=8
&c  =0x28ff1c
&c.a=0x28ff20
```
由于类A的成员函数存放在代码区，测试代码中的A在栈区的大小等于它的成员变量所占字节数。类B与类C中的虚函数也同样存放在代码区，但同时在offset为0的位置必须包含一个大小为4字节，指向各自虚函数表的指针。
```cpp
(gdb) p /a (*(unsigned int*)&a)@4
$1 = {0x1, 0x401b60 <__do_global_dtors>, 0x28ff50, 0x28ff94}

(gdb) p /a (*(unsigned int*)&b)@4
$2 = {0x405190 <_ZTV1B+8>, 0x2, 0x1, 0x401b60 <__do_global_dtors>}

(gdb) p /a (*(unsigned int*)0x405190)@4
$3 = {0x403d38 <B::fun()>, 0x0, 0x405178 <_fu7___ZTVN10__cxxabiv117__class_type_infoE>, 0x403d5c <C::fun1()>}

(gdb) p /a (*(unsigned int*)&c)@4
$4 = {0x40519c <_ZTV1C+8>, 0x3, 0x405190 <_ZTV1B+8>, 0x2}

(gdb) p /a (*(unsigned int*)0x40519c)@4
$5 = {0x403d5c <C::fun1()>, 0x403d68 <C::fun2()>, 0x3a434347, 0x4e472820}
```

**测试二**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A1{
public:
    int a;
    virtual void fun(){}
};

class A2{
public:
    int b;
    virtual void fun(){}
};

class B:public A1,public A2{
public:
    int c;
    virtual void fun1(){}
};

int main() {
    B b;
    b.a=1;b.b=2;b.c=3;
    cout<<"sizeof(b)="<<sizeof(b)<<endl;
    cout<<"&b  ="<<&b<<endl;
    cout<<"&b.a="<<&b.a<<endl;
    cout<<"&b.b="<<&b.b<<endl;
    cout<<"&b.c="<<&b.c<<endl;
    return 0;
}
```
测试结果：
```
sizeof(b)=20
&b  =0x28ff1c
&b.a=0x28ff20
&b.b=0x28ff28
&b.c=0x28ff2c
```
父类A1与A2在子类B中的存放位置如下，此时，子类B中的虚函数fun1()存放在第一个父类A1的虚函数表中，所以子类B所占字节数为20。

| 起始地址 | 结束地址 | 说明 |
| ------- | -------- | ---- |
| 0x28ff1c | 0x28ff1f | 存放指向A1虚函数表的指针 |
| 0x28ff20 | 0x28ff23 | 存放A1的成员变量a |
| 0x28ff24 | 0x28ff27 | 存放指向A2虚函数表的指针 |
| 0x28ff28 | 0x28ff2b | 存放A2的成员变量b |
| 0x28ff2c | 0x28ff2f | 存放B的成员变量c |

从下面的gdb输出信息可以看到，在A1的虚函数表中除了存放A1的虚函数指针外，还存放着B继承的下一个父类A2在B中存放位置的偏移字节数的补码（offset=~0xfffffff8+1=8）。

```cpp
(gdb) p /a (*(unsigned int*)&b)@8
$1 = {0x40519c <_ZTV1B+8>, 0x1, 0x4051ac <_ZTV1B+24>, 0x2, 0x3, 0x401af0 <__do_global_dtors>, 0x28ff50, 0x28ff94}

(gdb) p /a (*(unsigned int*)0x40519c)@4
$2 = {0x403d0c <A1::fun()>, 0x403cc8 <B::fun1()>, 0xfffffff8, 0x405158 <_fu5___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x4051ac)@4
$3 = {0x403d30 <A2::fun()>, 0x0, 0x405178 <_fu7___ZTVN10__cxxabiv117__class_type_infoE>, 0x403d0c <A1::fun()>}
```

**测试三**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A1{
public:
    virtual void fun(){}
};

class A2{
public:
    virtual void fun(){}
};

class B1:public A1,public A2{
};

class B2:public A1,public A2{
    virtual void fun(){}
};

class B3:public A1,public A2{
    virtual void fun1(){}
};

int main() {
    cout<<"B1:"<<sizeof(B1)<<endl;
    cout<<"B2:"<<sizeof(B2)<<endl;
    cout<<"B3:"<<sizeof(B3)<<endl;
    return 0;
}
```
测试结果：
```
B1:8
B2:8
B3:8
```
子类继承的多个父类都有自己的虚函数列表，故B1所占字节数为8。子类B2覆盖了父类A1与A2的虚函数，各父类虚函数表中的fun()的位置被替换成了子类的函数指针；而子类B3中的虚函数存放在第一个父类A1的虚函数表中，所以B1与B2所占字节数都为8。
```cpp
(gdb) p /a (*(unsigned int*)&b1)@4
$1 = {0x4051d0 <_ZTV2B1+8>, 0x4051dc <_ZTV2B1+20>, 0x401a80 <__do_global_dtors>, 0x28ff50}

(gdb) p /a (*(unsigned int*)0x4051d0)@4
$2 = {0x403c58 <A1::fun()>, 0xfffffffc, 0x405154 <_fu5___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403c64 <A2::fun()>}

(gdb) p /a (*(unsigned int*)0x4051dc)@4
$3 = {0x403c64 <A2::fun()>, 0x0, 0x405174 <_fu4___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403c70 <B2::fun()>}

(gdb) p /a (*(unsigned int*)&b2)@4
$4 = {0x4051e8 <_ZTV2B2+8>, 0x4051f4 <_ZTV2B2+20>, 0x4051d0 <_ZTV2B1+8>, 0x4051dc <_ZTV2B1+20>}

(gdb) p /a (*(unsigned int*)0x4051e8)@4
$5 = {0x403c70 <B2::fun()>, 0xfffffffc, 0x405174 <_fu4___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403c88 <_ZThn4_N2B23funEv>}

(gdb) p /a (*(unsigned int*)0x4051f4)@4
$6 = {0x403c88 <_ZThn4_N2B23funEv>, 0x0, 0x405194 <_fu3___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403c58 <A1::fun()>}

(gdb) p /a (*(unsigned int*)&b3)@4
$7 = {0x405200 <_ZTV2B3+8>, 0x405210 <_ZTV2B3+24>, 0x4051e8 <_ZTV2B2+8>, 0x4051f4 <_ZTV2B2+20>}

(gdb) p /a (*(unsigned int*)0x405200)@4
$8 = {0x403c58 <A1::fun()>, 0x403c7c <B3::fun1()>, 0xfffffffc, 0x405194 <_fu3___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x405210)@4
$9 = {0x403c64 <A2::fun()>, 0x3a434347, 0x4e472820, 0x35202955}
```
通过gdb的输出信息我们可以看到，子类B2中父类A1与A2的虚函数表中的fun()确实被置为新值，可是如果是被替换成子类B2的函数指针的话，这两个新值不应该相等吗？这里我也不太理解，我们可以写个程序测试一下，将测试三的代码做如下修改：
```cpp
#include <iostream>
using namespace std;

class A1{
public:
    virtual void fun(){
        cout<<"A1::fun"<<endl;
    }
};

class A2{
public:
    virtual void fun(){
        cout<<"A2::fun"<<endl;
    }
};

class B2:public A1,public A2{
public:
    virtual void fun(){
        cout<<"B2::fun"<<endl;
    }
};

int main() {
    B2 b2;
    typedef void (*Fun)();

    Fun fun = (Fun)*(unsigned int*)*(unsigned int*)&b2;
    cout<<(unsigned int*)fun<<endl;
    fun();

    fun = (Fun)*(unsigned int*)*((unsigned int*)&b2+1);
    cout<<(unsigned int*)fun<<endl;
    fun();
    return 0;
}
```
运行结果：
```
0x403bf8
B2::fun
0x403c2c
B2::fun
```
我们可以看到，尽管函数指针不相等，但通过这两个函数指针调用的都是B2中的fun()。
```cpp
(gdb) p /a 0x403bf8
$1 = 0x403bf8 <B2::fun()>
(gdb) p /a 0x403c2c
$2 = 0x403c2c <_ZThn4_N2B23funEv>
```

从上面代码我们可以看到，虚函数在一定程度上破坏了类的封装特性，例如：
```cpp
#include <iostream>
using namespace std;

class A{
private:
    virtual void fun(){
        cout<<"A::fun"<<endl;
    }
};

int main() {
    A a;
    typedef void (*Fun)();

    Fun fun = (Fun)*(unsigned int*)*(unsigned int*)&a;
    fun();
    return 0;
}
```
运行结果：
```
A::fun
```
更进一步，在实现类的多态的过程中，子类的虚函数覆盖了父类在虚函数表上的相应虚函数，此时函数指针已经改变，但子类依旧能找到覆盖后的虚函数。也就是说，虚函数在调用时只是调用虚函数表中相应位置的虚函数，并不会判断此处的虚函数是否已发生变化。利用这一点，我们可以做一些有趣的事。
```cpp
#include <iostream>
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

class B{
public:
    virtual void funB1(){
        cout<<"funB1"<<endl;
    }
    virtual void funB2(){
        cout<<"funB2"<<endl;
    }
};

int main() {
    A a(0xa,0xb);
    ((B *)&a)->funB1();
    ((B *)&a)->funB2();
    return 0;
}
```
测试结果：
```
funA1
funA2
```

### 虚基类

**测试一**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A1 {
public:
    int a1;
    virtual void fun() {}
};

class A2 {
public:
    int a2;
    virtual void fun() {}
};

class B1 : public A1, virtual public A2 {
public:
    int b;
    virtual void funB() {}
};

class B2 : virtual public A1, virtual public A2 {
public:
    int b;
};

int main() {
    B1 b1; b1.a1 = 0x11; b1.a2 = 0x12; b1.b = 0x13;
    B2 b2; b2.a1 = 0x21; b2.a2 = 0x22; b2.b = 0x23;

    cout << "sizeof(b1)=" << sizeof(b1) << endl;
    cout << "&b1   =" << &b1 << endl;
    cout << "&b1.a1=" << &b1.a1 << endl;
    cout << "&b1.a2=" << &b1.a2 << endl;
    cout << "&b1.b =" << &b1.b << endl << endl;
    cout << "sizeof(b2)=" << sizeof(b2) << endl;
    cout << "&b2   =" << &b2 << endl;
    cout << "&b2.a1=" << &b2.a1 << endl;
    cout << "&b2.a2=" << &b2.a2 << endl;
    cout << "&b2.b =" << &b2.b << endl;
    return 0;
}
```
测试结果：
```
sizeof(b1)=20
&b1   =0x28ff1c
&b1.a1=0x28ff20
&b1.a2=0x28ff2c
&b1.b =0x28ff24

sizeof(b2)=24
&b2   =0x28ff04
&b2.a1=0x28ff10
&b2.a2=0x28ff18
&b2.b =0x28ff08
```
虚继承的子类的内存结构和普通继承完全不同。在虚继承的子类中，虚基类按照在子类的虚继承的顺序存放在子类的最后。另外，在子类的虚函数表中除了存放普通父类虚函数与子类自己的虚函数外，还存放着一个在子类内存中偏移字节数的补码（若该偏移处存放指向虚基类虚函数表的指针，则此处还需以一个四个字节的0x00000000来作为分界），在此偏移处存放着下一个指向普通继承的父类或虚基类虚函数表的指针，以此类推。
```cpp
(gdb) p /a (*(unsigned int*)&b1)@7
$1 = {0x405228 <_ZTV2B1+12>, 0x11, 0x13, 0x40523c <_ZTV2B1+32>, 0x12, 0x401c30 <__do_global_dtors>, 0x28ff50}

(gdb) p /a (*(unsigned int*)0x405228)@5
$2 = {0x403e08 <A1::fun()>, 0x403e50 <B1::funB()>, 0x0, 0xfffffff4, 0x4051a0 <_fu11___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x40523c)@5
$3 = {0x403e2c <A2::fun()>, 0x10, 0x8, 0x0, 0x4051c0 <_fu10___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)&b2)@7
$4 = {0x405250 <_ZTV2B2+16>, 0x23, 0x40525c <_ZTV2B2+28>, 0x21, 0x40526c <_ZTV2B2+44>, 0x22, 0x405228 <_ZTV2B1+12>}

(gdb) p /a (*(unsigned int*)0x405250)@5
$5 = {0x0, 0xfffffff8, 0x4051c0 <_fu10___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403e08 <A1::fun()>, 0x0}

(gdb) p /a (*(unsigned int*)0x40525c)@5
$6 = {0x403e08 <A1::fun()>, 0x0, 0xfffffff0, 0x4051c0 <_fu10___ZTVN10__cxxabiv121__vmi_class_type_infoE>, 0x403e2c <A2::fun()>}

(gdb) p /a (*(unsigned int*)0x40526c)@5
$7 = {0x403e2c <A2::fun()>, 0x3a434347, 0x4e472820, 0x35202955, 0x302e332e}
```

**测试二**

测试代码：
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int a;
    virtual void fun() {}
};

class VB1:virtual public A {};

class VB2:virtual public A {};

class B1: public A {};

class B2: public A {};

class C1:public VB1,public VB2 {};

class C2:public B1,public B2{};

int main() {
    C1 c1;c1.VB1::a=0x11;c1.VB2::a=0x12;
    C2 c2;c2.B1::a=0x21;c2.B2::a=0x22;
    cout<<"sizeof(c1)="<<sizeof(c1)<<endl;
    cout<<"&c1="<<&c1<<endl;
    cout<<"&c1.VB1::a="<<&c1.VB1::a<<endl;
    cout<<"&c1.VB2::a="<<&c1.VB2::a<<endl<<endl;
    cout<<"sizeof(c2)="<<sizeof(c2)<<endl;
    cout<<"&c2="<<&c2<<endl;
    cout<<"&c2.B1::a="<<&c2.B1::a<<endl;
    cout<<"&c2.B2::a="<<&c2.B2::a<<endl;
    return 0;
}
```
测试结果：
```
sizeof(c1)=16
&c1=0x28ff20
&c1.VB1::a=0x28ff2c
&c1.VB2::a=0x28ff2c

sizeof(c2)=16
&c2=0x28ff10
&c2.B1::a=0x28ff14
&c2.B2::a=0x28ff1c
```
在菱形继承中，若使用普通继承则会在子类C2中依次存放来自父类B1与B2的A，而若使用虚继承则只在子类C1内存空间的最后保存一份A的拷贝。所以，尽管C1和C2所占字节数一样，但其中存放的内容却并不相同。
```cpp
(gdb) p /a (*(unsigned int*)&c1)@5
$1 = {0x4052c0 <_ZTV2C1+12>, 0x4052cc <_ZTV2C1+24>, 0x4052d8 <_ZTV2C1+36>, 0x12, 0x401bb0 <__do_global_dtors>}

(gdb) p /a (*(unsigned int*)0x4052c0)@3
$2 = {0x4, 0xfffffffc, 0x4051e0 <_fu9___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x4052cc)@3
$3 = {0x0, 0xfffffff8, 0x4051e0 <_fu9___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x4052d8)@3
$4 = {0x403d88 <A::fun()>, 0x0, 0x405200 <_fu8___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)&c2)@5
$5 = {0x4052e4 <_ZTV2C2+8>, 0x21, 0x4052f0 <_ZTV2C2+20>, 0x22, 0x4052c0 <_ZTV2C1+12>}

(gdb) p /a (*(unsigned int*)0x4052e4)@3
$6 = {0x403d88 <A::fun()>, 0xfffffff8, 0x405200 <_fu8___ZTVN10__cxxabiv121__vmi_class_type_infoE>}

(gdb) p /a (*(unsigned int*)0x4052f0)@3
$7 = {0x403d88 <A::fun()>, 0x3a434347, 0x4e472820}
```

**参考链接**

[c/c++内存分配与内存对齐全面探讨](http://blog.csdn.net/cuibo1123/article/details/2547442)</br>
[C++类的实例化对象的大小之sizeof()](http://blog.csdn.net/houqd2012/article/details/40264943)</br>
[stm32中使用#pragma pack（非常有用的字节对齐用法说明）](http://www.cnblogs.com/King-Gentleman/p/5297355.html)</br>
[C++ 虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051/)</br>
[C++虚继承和虚基类](http://c.biancheng.net/cpp/biancheng/view/238.html)</br>
[c++类实例在内存中的分配 (转)](http://www.cnblogs.com/bizhu/archive/2012/09/25/2701691.html)</br>
