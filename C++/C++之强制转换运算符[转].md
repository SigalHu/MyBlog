原文：[强制转换运算符](https://msdn.microsoft.com/zh-cn/library/5f6c9f8h.aspx)

### 强制转换运算符

有几种特定于 C++ 语言的转换运算符。 这些运算符用于删除旧式 C 语言转换中的一些多义性和危险继承。 这些运算符是：

* **dynamic_cast：**用于多态类型的转换
* **static_cast：**用于非多态类型的转换
* **const_cast：**用于删除const、volatile和__unaligned特性
* **reinterpret_cast：**用于位的简单重新解释
* **safe_cast：**用于生成可验证的MSIL

在万不得已时使用 const_cast 和 reinterpret_cast，因为这些运算符与旧的样式转换带来的危险相同。 但是，若要完全替换旧的样式转换，仍必须使用它们。

### dynamic_cast 运算符

将操作数 expression 转换成类型为type-id 的对象。

**语法**

```cpp
dynamic_cast < type-id > ( expression )
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
class A {virtual void f();};
class B {virtual void f();};

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
