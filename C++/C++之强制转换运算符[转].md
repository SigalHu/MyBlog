原文：[强制转换运算符](https://msdn.microsoft.com/zh-cn/library/5f6c9f8h.aspx)

### 强制转换运算符

有几种特定于 C++ 语言的转换运算符。 这些运算符用于删除旧式 C 语言转换中的一些多义性和危险继承。 这些运算符是：

* **dynamic_cast：**用于多态类型的转换
* **static_cast：**用于非多态类型的转换
* **const_cast：**用于删除const、volatile和__unaligned特性
* **reinterpret_cast：**用于位的简单重新解释

在万不得已时使用 const_cast 和 reinterpret_cast，因为这些运算符与旧的样式转换带来的危险相同。 但是，若要完全替换旧的样式转换，仍必须使用它们。

### dynamic_cast 运算符

将操作数 expression 转换成类型为type-id 的对象。

**语法**

```cpp
dynamic_cast<type-id>(expression)
```

**备注**

type-id 必须是一个指针或引用到以前已定义的类类型的引用或“指向 void 的指针”。 如果 type-id 是指针，则 expression 的类型必须是指针，如果 type-id 是引用，则为左值。

在托管代码中的 dynamic_cast 的行为中有两个重大更改。

* 为指针的 dynamic_cast 对指向装箱的枚举的基础类型的指针将在运行时失败，则返回 0 而不是已转换的指针。
* dynamic_cast 将不再引发一个异常，当 type-id 是指向值类型的内部指针，则转换在运行时失败。该转换将返回 0 指示运行值而不是引发。

如果 type-id 是指向 expression 的明确的可访问的直接或间接基类的指针，则结果是指向 type-id 类型的唯一子对象的指针。 例如：
```cpp
// dynamic_cast_1.cpp
// compile with: /c
class B { };
class C : public B { };
class D : public C { };

void f(D* pd) {
	C* pc = dynamic_cast<C*>(pd);   // ok: C is a direct base class
	// pc points to C subobject of pd
	B* pb = dynamic_cast<B*>(pd);   // ok: B is an indirect base class
	// pb points to B subobject of pd
}
```
此转换类型称为“向上转换”，因为它将在类层次结构上的指针，从派生的类移到该类派生的类。向上转换是一种隐式转换。

如果 type-id 为 void*，则做运行时进行检查确定 expression 的实际类型。 结果是指向 by expression 的完整的对象的指针。 例如：
```cpp
// dynamic_cast_2.cpp
// compile with: /c /GR
class A { virtual void f(); };
class B { virtual void f(); };

void f() {
	A* pa = new A;
	B* pb = new B;
	void* pv = dynamic_cast<void*>(pa);
	// pv now points to an object of type A

	pv = dynamic_cast<void*>(pb);
	// pv now points to an object of type B
}
```
如果 type-id 不是 void*，则做运行时进行检查以确定是否由 expression 指向的对象可以转换为由 type-id 指向的类型。

如果 expression 类型是 type-id 类型的基类，则做运行时检查来看是否 expression 确实指向 type-id 类型的完整对象。如果为 true，则结果是指向 type-id 类型的完整对象的指针。例如：
```cpp
// dynamic_cast_3.cpp
// compile with: /c /GR
class B { virtual void f(); };
class D : public B { virtual void f(); };

void f() {
	B* pb = new D;   // unclear but ok
	B* pb2 = new B;

	D* pd = dynamic_cast<D*>(pb);   // ok: pb actually points to a D
	D* pd2 = dynamic_cast<D*>(pb2);   // pb2 points to a B not a D
}
```
此转换类型称为“向下转换”，因为它将在类层次结构下的指针，从给定的类移到该类派生的类。

对于多重继承，引入多义性的可能性。考虑下图中显示的类层次结构。

对于 CLR 类型，如果转换可以隐式执行，则 dynamic_cast 结果为 no-op，如果转换失败，则 MSIL isinst 指令将执行动态检查并返回 nullptr。

以下示例使用 dynamic_cast 以确定一个类是否为特殊类型的实例：
```cpp
// dynamic_cast_clr.cpp
// compile with: /clr
using namespace System;

void PrintObjectType(Object^o) {
	if (dynamic_cast<String^>(o))
		Console::WriteLine("Object is a String");
	else if (dynamic_cast<int^>(o))
		Console::WriteLine("Object is an int");
}

int main() {
	Object^o1 = "hello";
	Object^o2 = 10;

	PrintObjectType(o1);
	PrintObjectType(o2);
}
```
显示多继承的类层次结构：

![](C++之强制转换运算符[转]/1.png)

指向类型 D 对象的指针可以安全地强制转换为 B 或 C。但是，如果 D 强制转换为指向 A 对象的指针，会导致 A 的哪个实例？这将导致不明确的强制转换错误。若要避免此问题，可以执行两个明确的转换。例如：
```cpp
// dynamic_cast_4.cpp
// compile with: /c /GR
class A { virtual void f(); };
class B { virtual void f(); };
class D : public B { virtual void f(); };

void f() {
	D* pd = new D;
	B* pb = dynamic_cast<B*>(pd);   // first cast to B
	A* pa2 = dynamic_cast<A*>(pb);   // ok: unambiguous
}
```
当使用虚拟基类时，其他多义性问题会被引入。考虑下图中显示的类层次结构。

显示虚拟基类的类层次结构：

![](C++之强制转换运算符[转]/2.png)

在此层次结构中，A 是虚拟基类。给定一个 E 类实例和一个指向 A 子对象的指针，指向 B 指针的 dynamic_cast 将失败于多义性。 必须先将强制转换回完整 E 对象，然后以明确的方式反向沿层次结构，到达正确的 B 对象。

显示重复基类的类层次结构：

![](C++之强制转换运算符[转]/3.png)

给定一个 E 类型的对象和一个指向 D 子对象的指针，从 D 子对象定位到最左侧的 A 子对象，可进行三个转换。可以从 D 指针到 E 指针执行 dynamic_cast 转换，然后从 E 到 B 执行转换（dynamic_cast 或隐式转换），最后从 B 到 A 执行隐式转换。例如：
```cpp
// dynamic_cast_5.cpp
// compile with: /c /GR
class A { virtual void f(); };
class B : public A { virtual void f(); };
class C : public A { };
class D { virtual void f(); };
class E : public B, public C, public D { virtual void f(); };

void f(D* pd) {
	E* pe = dynamic_cast<E*>(pd);
	B* pb = pe;   // upcast, implicit conversion
	A* pa = pb;   // upcast, implicit conversion
}
```
dynamic_cast 运算符还可以使用执行 “相互转换”。使用同一个类层次结构可能进行指针转换，例如：从 B 子对象转换到 D子对象（只要整个对象是类转换型E。

考虑相互转换，实际上从指针转换到 D 到指针到最左侧的 A 子对象只要两个步骤。可以从 D 到 B 执行相互转换，然后从 B 到 A 执行隐式转换。例如：
```cpp
// dynamic_cast_6.cpp
// compile with: /c /GR
class A { virtual void f(); };
class B : public A { virtual void f(); };
class C : public A { };
class D { virtual void f(); };
class E : public B, public C, public D { virtual void f(); };

void f(D* pd) {
	B* pb = dynamic_cast<B*>(pd);   // cross cast
	A* pa = pb;   // upcast, implicit conversion
}
```
通过 dynamic_cast 将 null 指针值转换到目标类型的 null 指针值。

当您使用 `dynamic_cast<type-id>(expression)` 时，如果 expression 无法安全地转换成类型 type-id，则运行时检查会引起变换失败。 例如：
```cpp
// dynamic_cast_7.cpp
// compile with: /c /GR
class A { virtual void f(); };
class B { virtual void f(); };

void f() {
	A* pa = new A;
	B* pb = dynamic_cast<B*>(pa);   // fails at runtime, not safe;
	// B not derived from A
}
```
指针类型的非限定转换的值是 null 指针。 引用类型的非限定转换会引发 bad_cast 异常。 如果 expression 不指向也不引用有效的对象，则 __non_rtti_object 异常引发。

### static_cast 运算符

仅根据表达式中存在的类型，将 expression 转换为 type-id，类型。

**语法**

```cpp
static_cast<type-id>(expression)
```

**备注**

在标准 C++ 中，不进行运行时类型检查来帮助确保转换的安全。在 C++/CX 中，将执行编译时和运行时检查。

static_cast 运算符可用于将指向基类的指针转换为指向派生类的指针等操作。此类转换并非始终安全。

通常使用 static_cast 转换数值数据类型，例如将枚举型转换为整型或将整型转换为浮点型，而且你能确定参与转换的数据类型。static_cast 转换安全性不如 dynamic_cast 转换，因为 static_cast 不执行运行时类型检查，而 dynamic_cast 执行该检查。对不明确的指针的 dynamic_cast 将失败，而 static_cast 的返回结果看似没有问题，这是危险的。尽管 dynamic_cast 转换更加安全，但是 dynamic_cast 只适用于指针或引用，而且运行时类型检查也是一项开销。

在下面的示例中，因为 D 可能有不在 B 内的字段和方法，所以行 `D* pd2 = static_cast<D*>(pb);` 不安全。 但是，因为 D 始终包含所有 B，所以行 `B* pb2 = static_cast<B*>(pd);` 是安全的转换。
```cpp
// static_cast_Operator.cpp
// compile with: /LD
class B {};
class D : public B {};

void f(B* pb, D* pd) {
	D* pd2 = static_cast<D*>(pb);   // Not safe, D can have fields
	// and methods that are not in B.
	B* pb2 = static_cast<B*>(pd);   // Safe conversion, D always
	// contains all of B.
}
```
与 dynamic_cast 不同，pb 的 static_cast 转换不执行运行时检查。由 pb 指向的对象可能不是 D 类型的对象，在这种情况下使用 *pd2 会是灾难性的。例如，调用 D 类（而非 B 类）的成员函数可能会导致访问冲突。

dynamic_cast 和 static_cast 运算符可以在整个类层次结构中移动指针。然而，static_cast 完全依赖于转换语句提供的信息，因此可能不安全。例如：
```cpp
// static_cast_Operator_2.cpp
// compile with: /LD /GR
class B {
public:
	virtual void Test(){}
};
class D : public B {};

void f(B* pb) {
	D* pd1 = dynamic_cast<D*>(pb);
	D* pd2 = static_cast<D*>(pb);
}
```
如果 pb 确实指向 D 类型的对象，则 pd1 和 pd2 将获取相同的值。如果 `pb == 0`，它们也将获取相同的值。

如果 pb 指向 B 类型的对象，而非指向完整的 D 类，则 dynamic_cast 足以判断返回零。但是，static_cast 依赖于程序员的断言，即 pb 指向 D 类型的对象，因而只是返回指向那个假定的 D 对象的指针。

因此，static_cast 可以反向执行隐式转换，而在这种情况下结果是不确定的。这需要程序员来验证 static_cast 转换的结果是否安全。

该行为也适用于类以外的类型。例如，static_cast 可用于将 int 转换为 char。但是，得到的 char 可能没有足够的位来保存整个 int 值。同样，这需要程序员来验证 static_cast 转换的结果是否安全。

static_cast 运算符还可用于执行任何隐式转换，包括标准转换和用户定义的转换。例如：
```cpp
// static_cast_Operator_3.cpp
// compile with: /LD /GR
typedef unsigned char BYTE;

void f() {
	char ch;
	int i = 65;
	float f = 2.5;
	double dbl;

	ch = static_cast<char>(i);   // int to char
	dbl = static_cast<double>(f);   // float to double
	i = static_cast<BYTE>(ch);
}
```
static_cast 运算符可以将整数值显式转换为枚举类型。如果整型值不在枚举值的范围内，生成的枚举值是不确定的。

static_cast 运算符将 null 指针值转换为目标类型的 null 指针值。

任何表达式都可以通过 static_cast 运算符显式转换为 void 类型。目标 void 类型可以选择性地包含 const、volatile 或 __unaligned 特性。

static_cast 运算符无法转换掉 const、volatile 或 __unaligned 特性。

由于在一个重定位垃圾回收器顶部执行无检查转换的危险，你只应在确信代码将正常运行的时候，在性能关键代码中使用 static_cast。如果必须在发布模式下使用 static_cast，请在调试版本中用 safe_cast 替换它以确保成功。

### const_cast 运算符

从类中移除 const、volatile 和 __unaligned 特性。

**语法**

```cpp
const_cast<type-id>(expression)
```

**备注**

指向任何对象类型的指针或指向数据成员的指针可显式转换为完全相同的类型（const、volatile 和 __unaligned 限定符除外）。 对于指针和引用，结果将引用原始对象。对于指向数据成员的指针，结果将引用与指向数据成员的原始（未强制转换）的指针相同的成员。 根据引用对象的类型，通过生成的指针、引用或指向数据成员的指针的写入操作可能产生未定义的行为。

您不能使用 const_cast 运算符直接重写常量变量的常量状态。

const_cast 运算符将 null 指针值转换为目标类型的 null 指针值。
```cpp
#include <iostream>
using namespace std;

class CCTest {
public:
	void setNumber(int);
	void printNumber() const;
private:
	int number;
};

void CCTest::setNumber(int num) { number = num; }

void CCTest::printNumber() const {
	cout << "\nBefore: " << number;
	const_cast< CCTest * >(this)->number--;
	cout << "\nAfter: " << number;
}

int main() {
	CCTest X;
	X.setNumber(8);
	X.printNumber();
}
```
在包含 const_cast 的行中，this 指针的数据类型为 const CCTest *。 const_cast 运算符会将 this 指针的数据类型更改为 CCTest *，以允许修改成员 number。 强制转换仅对其所在的语句中的其余部分持续。

### reinterpret_cast 运算符

允许将任何指针转换为任何其他指针类型。也允许将任何整数类型转换为任何指针类型以及反向转换。

**语法**

```cpp
reinterpret_cast<type-id>(expression)
```

**备注**

滥用 reinterpret_cast 运算符可能很容易带来风险。 除非所需转换本身是低级别的，否则应使用其他强制转换运算符之一。

reinterpret_cast 运算符可用于 char\* 到 int\* 或 One_class\* 到 Unrelated_class\* 之类的转换，这本身并不安全。

reinterpret_cast 的结果不能安全地用于除强制转换回其原始类型以外的任何用途。在最好的情况下，其他用途也是不可移植的。

reinterpret_cast 运算符不能丢掉 const、volatile 或 __unaligned 特性。

reinterpret_cast 运算符将 null 指针值转换为目标类型的 null 指针值。

reinterpret_cast 的一个实际用途是在哈希函数中，即，通过让两个不同的值几乎不以相同的索引结尾的方式将值映射到索引。
```cpp
#include <iostream>
using namespace std;

// Returns a hash code based on an address
unsigned short Hash(void *p) {
	unsigned int val = reinterpret_cast<unsigned int>(p);
	return (unsigned short)(val ^ (val >> 16));
}

int main() {
	int a[20];
	for (int i = 0; i < 20; i++)
		cout << Hash(a + i) << endl;
}
```
reinterpret_cast 允许将指针视为整数类型。结果随后将按位移位并与自身进行“异或”运算以生成唯一的索引（具有唯一性的概率非常高）。该索引随后被标准 C 样式强制转换截断为函数的返回类型。
