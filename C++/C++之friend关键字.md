友元提供了不同类的成员函数之间、类的成员函数与一般函数之间进行数据共享的机制。通过友元，一个不同函数或另一个类中的成员函数可以访问类中的私有成员和保护成员。

友元的声明以关键字`friend`开始，只能出现在类定义的内部。**因为友元函数是类外的函数，所以它的声明可以放在类的私有段或公有段且没有区别。**

友元的正确使用能提高程序的运行效率，但同时也破坏了类的封装性和数据的隐藏性，导致程序可维护性变差。


### 普通函数友元函数

友元函数是可以直接访问类的私有成员的非成员函数。它是定义在类外的普通函数，它不属于任何类，但需要在类的定义中加以声明，声明时只需在友元的名称前加上关键字`friend`，其格式如下：
```cpp
friend 类型 函数名(形式参数);
```
友元函数的声明可以放在类的私有部分，也可以放在公有部分，它们是没有区别的，都说明是该类的一个友元函数。

友元函数的调用与一般函数的调用方式和原理一致。
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
**一个函数可以是多个类的友元函数，只需要在各个类中分别声明。**
```cpp
#include <iostream>
using namespace std;

class B;

class A{
private:
    int a;
public:
    int get_a(){
        return this->a;
    }
    friend void set_ab(A &a,int aa,B &b,int bb);
};

class B{
private:
    int b;
public:
    int get_b(){
        return this->b;
    }
    friend void set_ab(A &a,int aa,B &b,int bb);
};

void set_ab(A &a,int aa,B &b,int bb){
    a.a = aa;
    b.b = bb;
}

int main() {
    A a;B b;
    set_ab(a,1,b,2);
    cout<<a.get_a()<<" "<<b.get_b()<<endl;
    return 0;
}
```
运行结果：
```
1 2
```

### 一个类的函数是另一个类的友元

将一个类`B`的成员函数声明为类`A`的友元函数，使得可以通过该成员函数访问类`A`的私有成员。**只有当一个类的定义已经被看到时它的成员函数才能被声明为另一个类的友元。**
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

友元类的所有成员函数都是另一个类的友元函数，都可以访问另一个类中的隐藏信息（包括私有成员和保护成员）。

当希望一个类可以存取另一个类的私有成员时，可以将该类声明为另一类的友元类。定义友元类的语句格式如下：
```cpp
friend class 类名;
```
其中，`friend`和`class`是关键字，类名必须是程序中的一个已定义过的类。
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
    friend class B;
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

### 其他

**1. 友元函数与成员函数的区别**

* 成员函数有`this`指针，而友元函数没有`this`指针
* 友元函数不能被继承

**2. 使用友元类时注意：**
* 友元关系不能被继承。
* 友元关系是单向的，不具有交换性。若类`B`是类`A`的友元，类`A`不一定是类`B`的友元，要看在类中是否有相应的声明。
* 友元关系不具有传递性。若类`B`是类`A`的友元，类`C`是`B`的友元，类`C`不一定是类`A`的友元，同样要看类中是否有相应的申明

**3. 两个类如何通过友元声明互相访问对方的非公有成员？**
```cpp
#include <iostream>
using namespace std;

class B;

class A{
private:
    int a;
public:
    int get_a(){
        return this->a;
    }
    friend class B;
    void set_b(B &b,int bb);
};

class B{
private:
    int b;
public:
    int get_b(){
        return this->b;
    }
    void set_a(A &a,int aa){
        a.a = aa;
    }
    friend void A::set_b(B &b, int bb);
};

void A::set_b(B &b, int bb){
    b.b = bb;
}

int main() {
    A a;B b;
    b.set_a(a,1);
    a.set_b(b,2);
    cout<<a.get_a()<<" "<<b.get_b()<<endl;
    return 0;
}
```
运行结果：
```
1 2
```


**参考链接**

[C++ 友元关系详解](http://blog.csdn.net/vincent040/article/details/8620715)</br>
[友元函数和友元类](http://www.cnblogs.com/staring-hxs/p/3432161.html)</br>
[关于C++中的友元函数的总结](http://www.cnblogs.com/BeyondAnyTime/archive/2012/06/04/2535305.html)</br>
[待补遗(6)[C++]两个类如何通过友元声明互相访问对方的非公有成员](http://blog.csdn.net/u011559205/article/details/41980205)</br>
[C++两个类使用同一个友元函数来进行相互调用](http://blog.csdn.net/pcliuguangtao/article/details/6433870)
