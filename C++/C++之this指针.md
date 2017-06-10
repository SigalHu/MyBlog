this指针是类的普通成员函数的隐含形参，只能在成员函数中使用，指向调用该成员函数的实例地址，当形参与成员变量重名时，我们可以通过this指针加以区分。
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int a;
    A* set(int a){
        this->a = a;
        return this;
    }
};

int main() {
    A a={0};
    cout<<&a<<endl;
    cout<<a.a<<endl;
    cout<<a.set(1)<<endl;
    cout<<a.a<<endl;
    return 0;
}
```
运行结果：
```
0x28ff2c
0
0x28ff2c
1
```
我们知道同一类的不同实例所调用的同一成员函数只有一份，保存在代码区，通过隐含形参this来对不同的类加以区分。在下面的代码中，我们声明一个类的空指针，并调用成员函数，此时，该指针并没有指向实例地址，即没有分配栈区内存，但我们依旧可以调用位于代码区的成员函数，只是在该函数中，this指针为NULL，若指向普通成员变量就会运行错误，但是可以指向静态成员变量，这是由于静态成员变量在程序启动时就分配空间，位于在全局数据，且静态成员变量只有一份拷贝，为所有实例共享。
```cpp
#include <iostream>
using namespace std;

class A {
public:
    int a;
    static int b;
    A* set(){
//        this->a = 1;  // 错误
        this->b = 1;
        return this;
    }
};
int A::b = 0;

int main() {
    A *pa=NULL;
    cout<<pa<<endl;
    cout<<pa->b<<endl;
    cout<<pa->set()<<endl;
    cout<<pa->b<<endl;
    return 0;
}
```
运行结果：
```
0
0
0
1
```
我们知道，在静态成员函数中不能使用this指针，也不能访问非静态成员变量，当静态成员函数的形参与静态成员变量同名，我们可以通过`<类名>::<静态成员变量>`的方式访问静态成员变量。
```cpp
#include <iostream>
using namespace std;

class A {
public:
    static int a;
    static void set(int a){
        A::a = a;
    }
};
int A::a = 0;

int main() {
    cout<<A::a<<endl;
    A::set(1);
    cout<<A::a<<endl;
    return 0;
}
```
运行结果：
```
0
1
```
