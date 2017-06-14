### 虚函数与虚函数表

在类中加了virtual关键字的成员函数就是虚函数。其中，友元函数、构造函数、静态成员函数不能用virtual关键字修饰。

```cpp
#include <iostream>
using namespace std;

class A1{
public:
    int a;
    virtual void funA(){
        cout << "A1::funA()" << endl;
    }
};

class A2{
public:
    int a;
    void funA(){
        cout << "A2::funA()" << endl;
    }
};

int main() {
    A1 a1;
    A1 *pa1 = &a1;
    A1 &aa1 = a1;
    cout << "sizeof(a1)=" << sizeof(a1) << endl;
    a1.funA();
    pa1->funA();
    aa1.funA();

    A2 a2;
    A2 *pa2 = &a2;
    A2 &aa2 = a2;
    cout << "sizeof(a2)=" << sizeof(a2) << endl;
    a2.funA();
    pa2->funA();
    aa2.funA();
    return 0;
}
```
运行结果：
```
sizeof(a1)=8
A1::funA()
A1::funA()
A1::funA()
sizeof(a2)=4
A2::funA()
A2::funA()
A2::funA()
```
通过下面反汇编结果可以看到，实例a1与a2都是通过函数指针完成函数的调用，所以并不存在效率降低问题。但我们也看到包含虚函数的a1所占内存比a2多4字节，虚函数跟普通成员函数一样都保存在代码区，这里多出的4个字节存放的是一个指向类A1虚函数表的指针，在虚函数表中存放着A1中所定义的虚函数。当通过指针或引用调用普通成员函数时，还是直接通过函数指针调用，而此时调用虚函数则是先通过存放在实例的指向虚函数表的指针找到虚函数表，再从虚函数表的确定位置处来获取函数指针，这样就降低了效率。

![](C++之virtual关键字/1.png)

```cpp
...
25: 	a1.funA();
lea         ecx,[a1]
call        A1::funA (0BA150Fh)
26: 	pa1->funA();
mov         eax,dword ptr [pa1]
mov         edx,dword ptr [eax]
mov         esi,esp
mov         ecx,dword ptr [pa1]
mov         eax,dword ptr [edx]
call        eax
cmp         esi,esp
call        __RTC_CheckEsp (0BA1348h)
27: 	aa1.funA();
mov         eax,dword ptr [aa1]
mov         edx,dword ptr [eax]
mov         esi,esp
mov         ecx,dword ptr [aa1]
mov         eax,dword ptr [edx]
call        eax
cmp         esi,esp
call        __RTC_CheckEsp (0BA1348h)
...
33: 	a2.funA();
lea         ecx,[a2]
call        A2::funA (0BA153Ch)
34: 	pa2->funA();
mov         ecx,dword ptr [pa2]
call        A2::funA (0BA153Ch)
35: 	aa2.funA();
mov         ecx,dword ptr [aa2]
call        A2::funA (0BA153Ch)
...
```
由于虚函数指针存放在虚函数表中，我们可以通过存放在实例中的指向虚函数表的指针找到函数指针，这在一定程度上也破坏了类的封装性。当然，在下面代码中如果直接通过函数指针调用虚函数，函数体中this指针的使用会受到限制，可是通过上面的分析，我们知道也可以通过将实例a的地址转为类B的指针或将类B的引用实现对类A私有虚函数的直接调用。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    int a;
private:
    virtual void funA1(){
        cout << "A::funA1()" << endl;
    }
    virtual void funA2(int a){
        this->a = a;
        cout << "A::funA2()" << endl;
    }
};

class B{
public:
    virtual void funB1(){
        cout << "B::funB1()" << endl;
    }
    virtual void funB2(int a){
        cout << "B::funB2()" << endl;
    }
};

int main() {
    A a;
    typedef void(*Fun)();
    Fun fun = (Fun)*(unsigned int *)*(unsigned int*)&a;
    fun();
    cout << "a.a=" << a.a << endl;
    ((B *)&a)->funB2(1);
    cout << "a.a=" << a.a << endl;
    ((B &)a).funB2(2);
    cout << "a.a=" << a.a << endl;
    return 0;
}
```
运行结果：
```
A::funA1()
a.a=2686868
A::funA2()
a.a=1
A::funA2()
a.a=2
```
C\+\+语言为我们提供了一种语法结构，通过它可以指明，一个虚函数只是提供了一个可被子类改写的接口。但是，它本身并不能通过虚拟机制被调用。这就是纯虚函数。含有（或继续）一个或多个纯虚函数的类是抽象基类，抽象基类不能实例化，一般用于继承。
```cpp
#include <iostream>
using namespace std;

class A{
public:
    virtual void funA() = 0;  // 纯虚函数
};

class B :public A{
public:
    virtual void funA(){
        cout << "B::funA()" << endl;
    }
};

int main() {
//    A a;  // 错误
    A &&a = B();
    a.funA();
    return 0;
}
```

### 覆盖与多态

在基类的派生类中可以通过重写虚函数来实现对基类虚函数的覆盖，当基类的指针指向派生类的对象时，对point的print函数的调用实际上是调用了Derived的print函数而不是Base的print函数。这是面向对象中的多态性的体现
```cpp
#include <iostream>
using namespace std;

class A1{
public:
	int a;
	virtual void funA1(){
		cout << "A1::funA1()" << endl;
	}
};

class A2{
public:
	int a;
	void funA2(){
		cout << "A2::funA2()" << endl;
	}
};

class B :public A1{
public:
	int b;
	virtual void funA1(){
		cout << "B::funA1()" << endl;
	}
};

int main() {
	A1 a1; A2 a2; B b;
	cout << "sizeof(a1)=" << sizeof(a1) << endl;
	a1.funA1();
	cout << "sizeof(a2)=" << sizeof(a2) << endl;
	a2.funA2();
	cout << "sizeof(b)=" << sizeof(b) << endl;
	b.funA1();
	A1 &a = b;
	cout << "sizeof(a)=" << sizeof(a) << endl;
	a.funA1();
	cin.get();
	return 0;
}
```

```cpp
#include <iostream>
using namespace std;

class A{
public:
	virtual void funA1(){
		cout << "A::funA1()" << endl;
	}
};

class B:public A{
public:
	int a;
	virtual void funA1(){
		cout << "B::funA1()" << endl;
	}
};

int main() {
	B b;
	b.funA1();
	b.A::funA1();
	A &ab = b;
	ab.funA1();
	ab.A::funA1();
	A *pb = &b;
	pb->funA1();
	pb->A::funA1();
	cin.get();
	return 0;
}
```

```cpp
#include <iostream>
using namespace std;

class A{
public:
	virtual ~A(){
		cout << "A::~A()" << endl;
	}
};

class B:public A{
public:
	~B(){
		cout << "B::~B()" << endl;
	}
};

int main() {
	A *pb = new B();
	delete pb;
	cin.get();
	return 0;
}
```

### 虚基类

```cpp
#include <iostream>
using namespace std;

class A{
public:
	virtual void fun(){
		cout << "A::fun()" << endl;
	}
};

class B :virtual public A{
public:
	virtual void fun(){
		cout << "B::fun()" << endl;
	}
};

int main() {
	A &&a = B();
	a.fun();
	//B &b = static_cast<B&>(a);
	//b.fun();
	cin.get();
	return 0;
}
```

```cpp
#include <iostream>
using namespace std;

class A{
public:
	virtual ~A(){
		cout << "A::~A()" << endl;
	}
};

class B :virtual public A{
public:
	~B(){
		cout << "B::~B()" << endl;
	}
};

int main() {
		{
			A *pb = new B();
			delete pb;
		}
	cin.get();
	return 0;
}
```

**参考链接**

[C++ 的Virtual的用法](http://www.cnblogs.com/Yogurshine/archive/2013/01/10/2855654.html)</br>
[浅析c++中virtual关键字](http://blog.csdn.net/djh512/article/details/8973606)</br>
[c++ 虚继承与继承的差异](http://blog.csdn.net/dqjyong/article/details/8029527)</br>
[C++ 虚函数表解析](http://blog.csdn.net/haoel/article/details/1948051/)
