### 修饰变量

使用extern关键字声明变量可以告诉编译器，该变量如果在当前文件的当前编译语句的之前未被定义，那么就会在当前文件的后面或者其它文件中定义。

这是由于在编译阶段，我们所定义的全局变量的可见性仅局限于所在文件的定义位置之后，而其可见性扩展到整个程序则是在链接完成之后。

在下面的代码中，使用static关键字与const关键字修饰的全局变量a与c被分配到不同空间，其作用域都只局限于各自所在文件，但是使用static关键字修饰的全局变量a不能再使用extern关键字进行变量声明，而使用const关键字修饰的全局变量d则可以通过使用extern关键字进行变量声明。

test.cpp文件：
```cpp
#include <iostream>
using namespace std;

void print_test1();

//extern int a;  // 错误
static int a = 0xa0;
extern int b;
const int c = 0xc0;
extern const int d;

void print_test(){
    cout<<hex<<"&a="<<&a<<" | "<<"a="<<a<<dec<<endl;
    cout<<hex<<"&b="<<&b<<" | "<<"b="<<b<<dec<<endl;
    cout<<hex<<"&c="<<&c<<" | "<<"c="<<c<<dec<<endl;
    cout<<hex<<"&d="<<&d<<" | "<<"d="<<d<<dec<<endl;
}

int main() {
    print_test();
    cout<<endl;
    print_test1();
    return 0;
}
```
test1.cpp文件：
```cpp
#include <iostream>
using namespace std;

static int a = 0xa1;
int b = 0xb;
const int c = 0xc1;
extern const int d = 0xd;

void print_test1(){
    cout<<hex<<"&a="<<&a<<" | "<<"a="<<a<<dec<<endl;
    cout<<hex<<"&b="<<&b<<" | "<<"b="<<b<<dec<<endl;
    cout<<hex<<"&c="<<&c<<" | "<<"c="<<c<<dec<<endl;
    cout<<hex<<"&d="<<&d<<" | "<<"d="<<d<<dec<<endl;
}
```
运行结果：
```
&a=0x405004 | a=a0
&b=0x40500c | b=b
&c=0x406068 | c=c0
&d=0x406094 | d=d

&a=0x405008 | a=a1
&b=0x40500c | b=b
&c=0x406090 | c=c1
&d=0x406094 | d=d
```

### 修饰函数

extern关键字也可以对函数进行原型声明，以下两种形式的声明都是可以的。
```cpp
extern void fun();
// 或者
void fun();
```

### extern "C"

由于C\+\+支持多态性，也就是具有相同函数名的函数可以完成不同的功能，所以C\+\+通常是通过参数区分具体调用的是哪一个函数。在编译的时候，C\+\+编译器会将参数类型和函数名连接在一起。但是在C语言中，由于完全没有多态性的概念，C编译器在编译时除了会在函数名前面添加一个下划线之外，什么也不会做（至少很多编译器都是这样干的）。因此，C\+\+方式编译后的变量与函数名与C方式编译结果不同，所以在C\+\+与C的混合编程中需要使用extern “C”关键字。

**C++调用C函数**

test.cpp文件：
```cpp
#include <iostream>
using namespace std;

extern "C"{
    extern int a;
    void fun_testc();
}

void fun_test(){
    cout << "come from C++" << endl;
    cout << hex << "a=" << a << dec << endl;
}

int main() {
    fun_test();
    fun_testc();
    return 0;
}
```
testc.c文件：
```c
#include <stdio.h>

int a = 0xa;

void fun_testc(){
    printf("come from C\n");
    printf("a=%x\n", a);
}
```
运行结果：
```
come from C++
a=a
come from C
a=a
```
为了方便C\+\+或C调用，我们可以将testc.c的变量与函数声明到头文件中，并通过define一个跟文件同名大写的宏来防止重定义，其中，__cplusplus宏在C++中定义，C中没有该定义。

testc.h文件：
```cpp
#ifndef TESTC_H
#define TESTC_H

#ifdef __cplusplus
extern "C"{
#endif /* __cplusplus */

extern int a;
extern void fun_testc();

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif //TESTC_H
```

**C调用C++函数**

testc.c文件：
```c
#include <stdio.h>
#include "test.h"

void fun_testc(){
    printf("come from C\n");
    printf("a=%x\n", a);
}

int main() {
    fun_test();
    fun_testc();
    return 0;
}
```
test.h文件：
```cpp
#ifndef TEST_H
#define TEST_H

#ifdef __cplusplus
extern "C"{
#endif /* __cplusplus */

extern int a;
void fun_test();

#ifdef __cplusplus
}
#endif /* __cplusplus */

#endif //TEST_H
```
test.cpp文件：
```cpp
#include <iostream>
#include "test.h"
using namespace std;

int a = 0xa;
void fun_test(){
    cout << "come from C++" << endl;
    cout << hex << "a=" << a << dec << endl;
}
```
运行结果：
```
come from C++
a=a
come from C
a=a
```

**参考文献**

[解析“extern”](http://blog.csdn.net/keensword/article/details/401114)</br>
[C++中extern关键字使用](http://blog.csdn.net/sruru/article/details/7951019)</br>
[关于C++的extern关键字](http://www.cnblogs.com/ForFreeDom/archive/2012/03/21/2409950.html)</br>
[C和C++混合编程(__cplusplus 与 extern "c" 的使用)](http://www.cnblogs.com/x_wukong/p/5630143.html)
