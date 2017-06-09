### const关键字的使用

const是一个C++语言的限定符，它限定一个变量不允许被改变。

* **修饰常量**

用const修饰的变量是不可变的，以下两种定义形式在本质上是一样的：
```cpp
const int a = 10;
int const a = 10;
```

* **修饰指针**

如果const位于\*的左侧，则const就是用来修饰指针所指向的变量，即指针指向为常量；如果const位于\*的右侧，const就是修饰指针本身，即指针本身是常量。
```cpp
int a = 10;
const int* p = &a;            // 指针指向的内容不能变
int const* p = &a;            // 同上
int* const p = &a;            // 指针本身不能变
const int* const p = &a;      // 两者都不能变
int const* const p = &a;      // 同上
```

* **修饰引用**

以下两种定义形式在本质上是一样的：
```cpp
int a = 10;
const int& b = a;
int const& b = a;
```

* **修饰函数参数**

用const修饰函数参数，传递过来的参数在函数内不可以改变。
```cpp
void func (const int& n) {
     n = 10;        // 编译错误
}
```

* **修饰函数返回值**

用const修饰函数返回值的含义和用const修饰普通变量以及指针的含义基本相同。
```cpp
const int* func() {  // 返回的指针所指向的内容不能修改
    // return p;
}
```

* **修饰类成员变量**

用const修饰的类成员变量，**只能在类的构造函数初始化列表中赋值，不能在类构造函数体内赋值。**
```cpp
class A {
public：
    A(int x) : a(x) { // 正确
         //a = x;     // 错误
    }
private：
    const int a;
};
```

* **修饰类成员函数**

用const修饰的类成员函数，**在该函数体内不能改变该类对象的任何成员变量, 也不能调用类中任何非const成员函数。**
```cpp
class A {
public:
    int& getValue() const {
        // a = 10;    // 错误
        return a;
    }
private:
    int a;            // 非const成员变量
};
```

* **修饰类对象**

用const修饰的类对象，该对象内的任何成员变量都不能被修改。
因此**不能调用该对象的任何非const成员函数，因为对非const成员函数的调用会有修改成员变量的企图。**
```cpp
class A {
 public:
    void funcA() {}
    void funcB() const {}
};

int main {
    const A a;
    a.funcB();    // 可以
    a.funcA();    // 错误

    const A* b = new A();
    b->funcB();    // 可以
    b->funcA();    // 错误
}
```

* **在类内重载成员函数**
```cpp
class A {
public:
    void func() {}
    void func() const {}   // 重载
};
```

### const常量与#define的区别

* 编译器处理方式不同。#define宏是在预编译阶段展开（字符替换）；内置数据类型const常量是编译运行阶段使用（常量折叠）
* 类型和安全检查不同。#define宏没有类型，不做任何类型检查，仅仅是进行字符替换；const常量有具体的类型，在编译阶段会执行类型检查
* 存储方式不同。#define宏仅仅是展开，有多少地方使用，就展开多少次，不会分配内存；const常量会在内存中分配（栈区或者全局数据区）
* 另外，定义的const常量只在作用域内有效，而#define宏则是在定义范围内有效，例如可以将类的成员变量定义为const常量，此外，const常量可以是数组、类以及结构体等复杂数据类型，这些都是#define宏无法做到的。

换句话说，宏是字符常量，在预编译完宏替换完成后，该宏名字会消失，所有对宏的引用已经全部被替换为它所对应的值，编译器当然没有必要再维护这个符号。而常量折叠发生的情况是，对const常量的引用全部替换为该常量的值，但是，常量并不会消失，编译器会将其放入到符号表中，同时，会为该常量分配空间。
```cpp
#include <iostream>
using namespace std;

int main() {
#define DATA 1
    const int Data = 2;
    int a = DATA;
    int b = Data;
    return 0;
#undef DATA
}
```
在下面的汇编代码可以看到，#define宏与const常量在汇编中都被替换成立即数，与代码一起存于代码区。
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main():
4	int main() {
   0x00401460 <+0>:	push   %ebp
   0x00401461 <+1>:	mov    %esp,%ebp
   0x00401463 <+3>:	and    $0xfffffff0,%esp
   0x00401466 <+6>:	sub    $0x10,%esp
   0x00401469 <+9>:	call   0x401a20 <__main>

5	#define DATA 1
6	    const int Data = 2;
=> 0x0040146e <+14>:	movl   $0x2,0xc(%esp)

7	    int a = DATA;
   0x00401476 <+22>:	movl   $0x1,0x8(%esp)

8	    int b = Data;
   0x0040147e <+30>:	movl   $0x2,0x4(%esp)

9	    return 0;
   0x00401486 <+38>:	mov    $0x0,%eax

10	#undef DATA
11	}   0x0040148b <+43>:	leave
   0x0040148c <+44>:	ret

End of assembler dump.
```

### const常量在C与C++中的区别

在C++中，编译器会对内置整型数据类型的const常量进行常量替换，而在C中，则编译器没有做这部分优化，每次都要在const常量所在内存进行读取。
```cpp
int main(){
    const int a = 1;
    int b = a;
    return 0;
}
```
C版本汇编代码：
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main:
6	int main(){
   0x00401460 <+0>:	push   %ebp
   0x00401461 <+1>:	mov    %esp,%ebp
   0x00401463 <+3>:	and    $0xfffffff0,%esp
   0x00401466 <+6>:	sub    $0x10,%esp
   0x00401469 <+9>:	call   0x4019a0 <__main>

7	    const int a = 1;
=> 0x0040146e <+14>:	movl   $0x1,0xc(%esp)

8	    int b = a;
   0x00401476 <+22>:	mov    0xc(%esp),%eax
   0x0040147a <+26>:	mov    %eax,0x8(%esp)

9	    return 0;
   0x0040147e <+30>:	mov    $0x0,%eax

10	}   0x00401483 <+35>:	leave
   0x00401484 <+36>:	ret

End of assembler dump.
```
C++版本汇编代码：
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main():
1	int main(){
   0x00401460 <+0>:	push   %ebp
   0x00401461 <+1>:	mov    %esp,%ebp
   0x00401463 <+3>:	and    $0xfffffff0,%esp
   0x00401466 <+6>:	sub    $0x10,%esp
   0x00401469 <+9>:	call   0x4019a0 <__main>

2	    const int a = 1;
=> 0x0040146e <+14>:	movl   $0x1,0xc(%esp)

3	    int b = a;
   0x00401476 <+22>:	movl   $0x1,0x8(%esp)

4	    return 0;
   0x0040147e <+30>:	mov    $0x0,%eax

5	}   0x00401483 <+35>:	leave
   0x00401484 <+36>:	ret

End of assembler dump.
```

### 跟const有关的关键字

**volatile关键字**

volatile关键字是一种类型修饰符，用它声明的类型变量表示可能被某些编译器未知的因素更改，比如操作系统、硬件或者其它线程等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。

当要求使用volatile声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。

在下面代码中，const常量b通过volatile声明后，编译器不会对其进行常量替换，每次访问b，都从它所在的内存读取，所以通过指针改变b所在内存的数据，const常量b也会发生改变。
```cpp
#include <iostream>
using namespace std;

int main() {
    const int a = 1;
    volatile const int b = 2;
    int *pa = (int *)&a;
    int *pb = (int *)&b;
    cout<<a<<" "<<*pa<<"|"<<b<<" "<<*pb<<endl;
    (*pa)++;(*pb)++;
    cout<<a<<" "<<*pa<<"|"<<b<<" "<<*pb<<endl;
    return 0;
}
```
运行结果：
```
1 1|2 2
1 2|3 3
```

**mutable关键字**

在C++中，mutable关键字也是为了突破const的限制而设置的。被mutable修饰的变量（**mutable只能由于修饰类的非静态数据成员**），将永远处于可变的状态，即使在一个const函数中或者类的实例为const常量。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int a;
    mutable int b;
    void set(int a,int b)const{
//        this->a = a;  // 报错
        this->b = b;
    }
};

int main() {
    const A a={1,2};
    cout<<a.a <<" "<<a.b<<endl;
//    a.a = 2;  // 报错
    a.b = 3;
    cout<<a.a <<" "<<a.b<<endl;
    a.set(4,4);
    cout<<a.a <<" "<<a.b<<endl;
    return 0;
}
```

### const的进一步探究

结合代码与汇编指令，浮点型const常量Pi初始化时，编译器并没有将3.14以立即数的形式保存在代码区，而是将其存放在常量区，而且Pi、a与c都是在同一地址获取数据，这与字符串指针的初始化很像。正因为如此，每次访问Pi都需要从内存中读取数据，所以当我们可以通过指针改变Pi的值。在这里，const属性的修改可以通过const_cast运算符与传统转换方式实现。
```cpp
#include <iostream>
using namespace std;
#define PI 3.14

int main() {
    const float Pi = 3.14;
    float a = PI;
    float b = Pi;
    float c = 3.14;
    float *pb = const_cast<float *>(&b);
    cout<<b<<" "<<*pb<<endl;
    (*pb)++;
    cout<<b<<" "<<*pb<<endl;
    return 0;
}
```
运行结果：
```
3.14 3.14
4.14 4.14
```
部分汇编代码：
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main():
···
6	    const float Pi = 3.14;
=> 0x00401476 <+22>:	flds   0x405068
   0x0040147c <+28>:	fstps  -0xc(%ebp)

7	    float a = PI;
   0x0040147f <+31>:	flds   0x405068
   0x00401485 <+37>:	fstps  -0x10(%ebp)

8	    float b = Pi;
   0x00401488 <+40>:	flds   -0xc(%ebp)
   0x0040148b <+43>:	fstps  -0x1c(%ebp)

9	    float c = 3.14;
   0x0040148e <+46>:	flds   0x405068
   0x00401494 <+52>:	fstps  -0x14(%ebp)
···
End of assembler dump.
```
编译器会对内置整型数据类型const常量进行常量替换，但对于数组、结构体以及类等复杂数据类型，由于编译器不知道如何直接替换，因此必须要访问内存去获取数据。在下面的代码中，我们通过指针操作改变const常量的值，其中，常量a的引用在编译过程中就已经被编译器替换成立即数1并存于代码区，而常量数组b则没有被编译器进行优化，每次都需要从内存中读取数据，所以当我们通过指针改变了b[0]的值，常量数组的值也发生了改变。另外，我们也注意到，当const常量的初始化值为变量时，编译器也不会对其进行常量替换。
```cpp
#include <iostream>
using namespace std;

int main() {
    const int a = 1;
    const int b[] = {2};
    int c = 3;
    const int d = c;
    int *pa = (int *)&a;
    int *pb = const_cast<int *>(b);
    int *pd = (int *)&d;
    cout<<a<<" "<<*pa<<"|"<<b[0]<<" "<<*pb<<"|"<<d<<" "<<*pd<<endl;
    (*pa)++;(*pb)++;(*pd)++;
    cout<<a<<" "<<*pa<<"|"<<b[0]<<" "<<*pb<<"|"<<d<<" "<<*pd<<endl;
    return 0;
}
```
运行结果：
```
1 1|2 2|3 3
1 2|3 3|4 4
```
在下面的代码中，字符串指针c与d的初始化字符串保存在常量区，数据不可进行修改，如果强制修改，就会出现内存读写错误，与const无关。
```cpp
#include <iostream>
using namespace std;

int main() {
    const char a = '1';
    const char b[] = "2";
    const char *c = "3";
    char *d = "4";
    char *pa = (char *)&a;
    char *pb = (char *)b;
    char *pc = (char *)c;
    char *pd = (char *)d;

    cout<<a<<" "<<b<<" "<<c<<" "<<d<<endl;
    (*pa)++;
    (*pb)++;
//    (*pc)++;  // 运行错误
//    (*pd)++;  // 运行错误
    cout<<a<<" "<<b<<" "<<c<<" "<<d<<endl;
    return 0;
}
```
运行结果：
```
1 2 3 4
1 3 3 4
```
在下面的代码与汇编中，类A的常量数据成员a在实例定义时初始化，而静态常量数据成员b保存在全局数据区，为所有类A实例所共享，在编译时由编译器进行常量替换，而且，通过类实例访问的b也同样被优化。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    const int a = 1;
    static const int b = 2;
};

int main() {
    A a;
    int b = a.a;
    int c = A::b;
    int d = a.b;
    return 0;
}
```
汇编代码：
```cpp
(gdb) disassemble /m main
Dump of assembler code for function main():
10	int main() {
   0x00401460 <+0>:	push   %ebp
   0x00401461 <+1>:	mov    %esp,%ebp
   0x00401463 <+3>:	and    $0xfffffff0,%esp
   0x00401466 <+6>:	sub    $0x10,%esp
   0x00401469 <+9>:	call   0x401a10 <__main>

11	    A a;
=> 0x0040146e <+14>:	movl   $0x1,(%esp)

12	    int b = a.a;
   0x00401475 <+21>:	mov    (%esp),%eax
   0x00401478 <+24>:	mov    %eax,0xc(%esp)

13	    int c = A::b;
   0x0040147c <+28>:	movl   $0x2,0x8(%esp)

14	    int d = a.b;
   0x00401484 <+36>:	movl   $0x2,0x4(%esp)

15	    return 0;
   0x0040148c <+44>:	mov    $0x0,%eax

16	}   0x00401491 <+49>:	leave
   0x00401492 <+50>:	ret

End of assembler dump.
```
当类成员函数的形参为引用类型时，可以通过对该形参添加const限定重载成员函数，否则在编译时会报重定义错误。对于setA()的调用，我们可以假设对应实参为const常量，此时若不通过指针操作且该函数未被重载，编译时就会报错。而对于setB()的调用，不管形参有没有加const限定，都是将实参的数据复制到形参对其进行初始化，编译器无法区分这两个函数。此外，也可以通过对成员函数加const限定来重载成员函数，此时，若该类的实例的const常量，则只能调用加const限定的成员函数。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int a;
    int b;
    int c;
    void setA(A &a){
        this->a = a.a;
        cout<<"void setA(A &a)"<<endl;
    }
    void setA(const A &a){
        this->a = a.a;
        cout<<"void setA(const A &a)"<<endl;
    }
    void setB(int b){
        this->b = b;
    }
//    void setB(const int b){   // 报错
//        this->b = b;
//    }
    void setC(int a){
        cout<<"void setC(int a)"<<endl;
    }
    void setC(int a) const{
        cout<<"void setC(int a) const"<<endl;
    }
};

int main() {
    A a;
    const A b = {1,2,3};
    a.setA(a);
    a.setA(b);
    a.setC(4);
    b.setC(5);
    return 0;
}
```
运行结果：
```
void setA(A &a)
void setA(const A &a)
void setC(int a)
void setC(int a) const
```

**参考链接**

[C++ const关键字总结](http://www.cnblogs.com/chogen/p/4574118.html)</br>
[C++中的const完全解析](http://www.cnblogs.com/findumars/p/3697079.html)</br>
[C++常量折叠](http://blog.csdn.net/yby4769250/article/details/7359278)</br>
[C++中const的实现机制深入分析](http://blog.csdn.net/zyw_ym_zone/article/details/10211201)</br>
[C++中const、volatile、mutable、explicit的用法](http://blog.csdn.net/wangtaoking1/article/details/48058415)</br>
[C/C++要点全掌握（五）——mutable、volatile](http://blog.csdn.net/tht2009/article/details/6920511)
