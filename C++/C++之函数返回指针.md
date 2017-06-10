当函数的返回值为指针时，我们必须确保所返回的指针指向的内存空间是有效的。

```cpp
#include <iostream>
#include <cstring>
using namespace std;

char* fun(int mod){
    switch(mod){
        case 0:{
            char a[] = "0123456";
            char *pa = a;
            pa[0]++;
            cout<<&pa<<"->"<<(void *)pa<<":"<<pa<<endl;
            return pa;
        }
        case 1:{
            static char a[] = "1234567";
            a[0]++;
            cout<<&a<<"->"<<(void *)a<<":"<<a<<endl;
            return a;
        }
        case 2:{
            char *a = "2345678";
//            a[0]++;  // 错误
            cout<<&a<<"->"<<(void *)a<<":"<<a<<endl;
            return a;
        }
        case 3:{
            char *a = new char[8]{'3','4','5','6','7','8','9','\0'};
            a[0]++;
            cout<<&a<<"->"<<(void *)a<<":"<<a<<endl;
            return a;
        }
        case 4:{
            static char *a = "4567890";
//            a[0]++;   // 错误
            cout<<&a<<"->"<<(void *)a<<":"<<a<<endl;
            return a;
        }
        case 5:{
            static char *a = new char[8]{'5','6','7','8','9','0','1','\0'};
            a[0]++;
            cout<<&a<<"->"<<(void *)a<<":"<<a<<endl;
            return a;
        }
        default:
            return NULL;
    }
}

int main() {
    char str[8];
    char *a = fun(0);
    strcpy(str,a);
    cout<<a<<" "<<str<<endl;
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    a = fun(1);
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    a = fun(2);
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    a = fun(3);
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    delete a;
    a = fun(4);
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    a = fun(5);
    cout<<&a<<"->"<<(void *)a<<":"<<a<<endl<<endl;
    delete a;
    return 0;
}
```
运行结果：
```
0x28fee4->0x28fee8:1123456
   456 1123456
0x28ff14->0x28fee8:╔

0x405004->0x405004:2234567
0x28ff14->0x405004:2234567

0x28fee0->0x40606a:2345678
0x28ff14->0x40606a:2345678

0x28fedc->0x5315d0:4456789
0x28ff14->0x5315d0:4456789

0x40500c->0x40608e:4567890
0x28ff14->0x40608e:4567890

0x408030->0x5315d0:6678901
0x28ff14->0x5315d0:6678901
```
在case 0中所声明的字符串数组保存在栈中，在函数返回时空间被释放，在测试结果中我们可以看到，函数返回后该地址处的数据还在，但随即便被其他值覆盖。

在case 1中所声明的字符串数组由于添加了static关键字，字符串被保存在全局数据区，该区域在程序结束时由系统释放，所以我们可以通过指针访问到字符串。

在case 2中的字符串存放在常量区，不可修改，同样是在程序结束时由系统释放。

在case 3中通过new申请的内存空间位于堆中，并通过delete进行释放，由于在函数中我们并没有释放该处内存，所以当函数返回后可以通过指针进行访问。

在case 4、5中字符串分别保存在常量区与堆中，而指向字符串的指针位于全局数据区，所以在gdb中我们可以通过存放在全局数据区的字符串指针来访问字符串
```cpp
Breakpoint 2, main () at E:\Code\Cpp\test\test.cpp:62
62	    delete a;
(gdb) p (char *)0x405004
$1 = 0x405004 <fun(int)::a> "2234567"
(gdb) p (char *)*(long *)0x40500c
$2 = 0x40608e <std::piecewise_construct+42> "4567890"

Breakpoint 3, main () at E:\Code\Cpp\test\test.cpp:67
67	    delete a;
(gdb) p (char *)*(long *)0x408030
$3 = 0x6216a0 "6678901"
```
