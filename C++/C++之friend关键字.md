友元提供了不同类的成员函数之间、类的成员函数与一般函数之间进行数据共享的机制。通过友元，一个不同函数或另一个类中的成员函数可以访问类中的私有成员和保护成员。

友元的声明以关键字`friend`开始，只能出现在类定义的内部。**因为友元函数是类外的函数，所以它的声明可以放在类的私有段或公有段且没有区别。**

友元的正确使用能提高程序的运行效率，但同时也破坏了类的封装性和数据的隐藏性，导致程序可维护性变差。


### 普通函数友元函数

将一个普通函数声明为类的友元函数，使得该普通函数可以访问类的私有成员。
```cpp
#include <iostream>
using namespace std;

class A{
private:
    int a;
public:
    int get_a(){
        return this->a;
    }
    friend void set_a(A &a,int aa){
        a.a = aa;
    }
};

int main() {
    A a;
    set_a(a,1);
//    a.set_a(a,2); // 报错
    cout<<a.get_a()<<endl;
    return 0;
}
```
运行结果：
```
1
```

### 类`B`的一个成员函数为类`A`的友元函数

将一个类`B`的成员函数声明为类`A`的友元函数，使得可以通过该成员函数访问类`A`的私有成员。
```cpp
#include <iostream>
using namespace std;

class A;  // 前向声明

class B{
public:
    void set_a(A &a,int aa);
};

class A{
private:
    int a;
public:
    int get_a(){
        return this->a;
    }
    friend void B::set_a(A &a,int aa);
};

void B::set_a(A &a,int aa){
    a.a = aa;
}

int main() {
    A a;B b;
    b.set_a(a,1);
    cout<<a.get_a()<<endl;
    return 0;
}
```
运行结果：
```
1
```

### 友元类

将类`B`声明为类`A`的友元类，使得可以通过类B`对象访问类`A`的私有成员。
```cpp
#include <iostream>
using namespace std;

class B;  // 前向声明

class A{
private:
    int a;
public:
    int get_a(){
        return this->a;
    }
    friend B;
};

class B{
public:
    void set_a(A &a,int aa){
        a.a = aa;
    }
};

int main() {
    A a;B b;
    b.set_a(a,1);
    cout<<a.get_a()<<endl;
    return 0;
}
```
运行结果：
```
1
```

**参考链接**

[C++ 友元关系详解](http://blog.csdn.net/vincent040/article/details/8620715)</br>
[友元函数和友元类](http://www.cnblogs.com/staring-hxs/p/3432161.html)</br>
[关于C++中的友元函数的总结](http://www.cnblogs.com/BeyondAnyTime/archive/2012/06/04/2535305.html)</br>
[待补遗(6)[C++]两个类如何通过友元声明互相访问对方的非公有成员](http://blog.csdn.net/u011559205/article/details/41980205)</br>
[C++两个类使用同一个友元函数来进行相互调用](http://blog.csdn.net/pcliuguangtao/article/details/6433870)
