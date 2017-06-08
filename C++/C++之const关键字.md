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

* 编译器处理方式不同。#define宏是在预编译阶段展开（字符替换）；内置整型数据类型const常量是编译运行阶段使用（常量折叠）
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

### const的进一步探究

编译器会对内置整型数据类型const常量进行常量替换，但对于浮点数、数组、结构体以及类等复杂数据类型，由于编译器不知道如何直接替换，因此必须要访问内存去获取数据。在下面的代码中，我们通过指针操作改变const常量的值，其中，常量a的引用在编译过程中就已经被编译器替换成立即数1并存于代码区，而常量数组b则没有被编译器进行优化，每次都需要从内存中读取数据，所以当我们通过指针改变了b[0]的值，常量数组的值也发生了改变。在这里，const属性的修改可以通过const_cast运算符与传统转换方式实现。
```cpp
#include <iostream>
using namespace std;

int main() {
    const int a = 1;
    const int b[] = {2};
    const float c = 1.2;
    int *pa = (int *)&a;
    int *pb = const_cast<int *>(b);
    float *pc = (float *)&c;
    cout<<a<<" "<<*pa<<"|"<<b[0]<<" "<<*pb<<"|"<<c<<" "<<*pc<<endl;
    (*pa)++;(*pb)++;*pc = 3.4;
    cout<<a<<" "<<*pa<<"|"<<b[0]<<" "<<*pb<<"|"<<c<<" "<<*pc<<endl;
    return 0;
}
```
运行结果：
```
1 1|2 2|1.2 1.2
1 2|3 3|3.4 3.4
```




原因在于对const 类型,编译器有3种不同的处理.
1. 对于直接已知值的int,long,short,char 类型以及其unsigned版本,即 const int a=2; 这种,编译器编译程序之后,程序中所有a出现的地方,全部自动替换成2. 所以,就出现了对于 *b=3 ,在 const int a=2 ;中不会修改a,而在 const int a=c; 中则会修改 a的情况.
2. 对于字符串. 类似 const char *a="abc"; 这种,同样是不能修改的,不过原因就不再是上面那个,而是因为这个 "abc" 在编译之后是放在程序的"常量段",这部分是执行文件的一部分,运行期间不可修改,如果强制修改,就会出现 内存读写错误:0x000005不可写 这种错误.
3. 就是文章提到的这种情况,会 *b=3 会修改const的限制,原因也如文章中所说一致,这个限制只是编译期间限制,运行期间不受影响.对于上面没有提到的类型(包括float,double,以及自定义类型),都会作这种处理.
```cpp
#include <iostream>
using namespace std;

int main() {
    const char a = '1';
    cout<<a<<endl;
    char *pa = (char *)&a;
    cout<<*pa<<endl;
    (*pa)++;
    cout<<*pa<<endl;
    return 0;
}
```
```cpp
#include <iostream>
using namespace std;

int main() {
    const char a[] = "123";
//    const char *a = "123";
//    char *a = "123";
    cout<<a<<endl;
    char *pa = (char *)a;
    cout<<*pa<<endl;
    (*pa)++;
    cout<<*pa<<endl;
    cout<<a<<endl;
    return 0;
}
```
```cpp
#include <iostream>
using namespace std;

class A{
public:
    const int a = 1;
    static const int b= 2;
};

int main() {
    A a;
    int b = a.a;
    int c = a.b;
    return 0;
}
```
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int a;
    int b;
    void setA(int &a){
        this->a = a;
    }
    void setA(const int &a){
        this->a = a;
    }
    void setB(int b){
        this->b = b;
    }
//    void setB(const int b){
//        this->b = b;
//    }
};

int main() {
    A a;
    return 0;
}
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int a = 1;
    const int b = a;
    int c = b;

    const int e = 2;
    int f = e;
    return 0;
}
```
```cpp
#include <iostream>
using namespace std;

int main() {
    const int c = 3;
    int b = 2;
    int a = 1;
    int *pc = (int *)&c;
    cout<<hex<<"&a="<<&a<<dec<<"   a="<<a<<endl;
    cout<<hex<<"&b="<<&b<<dec<<"   b="<<b<<endl;
    cout<<hex<<"&c="<<&c<<dec<<"   c="<<c<<endl;
    cout<<hex<<"pc="<<pc<<dec<<" *pc="<<*pc<<endl;
    *pc = 0xc;
    cout<<hex<<"pc="<<pc<<dec<<" *pc="<<*pc<<endl;
    cout<<hex<<"&c="<<&c<<dec<<"   c="<<c<<endl;
    return 0;
}
```

```cpp
#include <iostream>
using namespace std;

int main() {
    volatile const int c = 3;
    int b = 2;
    int a = 1;
    int *pc = (int *)&c;
    cout<<hex<<"&a="<<&a<<dec<<"   a="<<a<<endl;
    cout<<hex<<"&b="<<&b<<dec<<"   b="<<b<<endl;
    cout<<hex<<"&c="<<&c<<dec<<"   c="<<c<<endl;
    cout<<hex<<"pc="<<pc<<dec<<" *pc="<<*pc<<endl;
    *pc = 0xc;
    cout<<hex<<"pc="<<pc<<dec<<" *pc="<<*pc<<endl;
    cout<<hex<<"&c="<<&c<<dec<<"   c="<<c<<endl;
    return 0;
}
```


**参考链接**

[C++ const关键字总结](http://www.cnblogs.com/chogen/p/4574118.html)</br>
[C++中的const完全解析](http://www.cnblogs.com/findumars/p/3697079.html)</br>
[C++常量折叠](http://blog.csdn.net/yby4769250/article/details/7359278)</br>
[C++中const的实现机制深入分析](http://blog.csdn.net/zyw_ym_zone/article/details/10211201)</br>
