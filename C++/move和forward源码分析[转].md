原文：[C++11尝鲜：std::move和std::forward源码分析](http://blog.csdn.net/zwvista/article/details/6848582)

`std::move`和`std::forward`是C\+\+0x中新增的标准库函数，分别用于实现移动语义和完美转发。

下面让我们分析一下这两个函数在gcc4.6中的具体实现。

### 预备知识

**引用折叠规则**
```cpp
X&  + &  => X&
X&& + &  => X&
X&  + && => X&
X&& + && => X&&
```

**函数模板参数推导规则（右值引用参数部分）**

当函数模板的模板参数为`T`而函数形参为`T&&`（右值引用）时适用本规则。

* 若实参为左值`U&`，则模板参数`T`应推导为引用类型`U&`。根据引用折叠规则，`U& + && => U&`，而`T&& ≡ U&`，故`T ≡ U&`。
* 若实参为右值`U&&`，则模板参数`T`应推导为非引用类型`U`。根据引用折叠规则，`U或U&& + && => U&&`，而`T&& ≡ U&&`，故`T ≡ U或U&&`，这里强制规定`T ≡ U`。

**std::remove_reference**

`std::remove_reference`为C\+\+0x标准库中的元函数，其功能为去除类型中的引用。
```cpp
std::remove_reference<U&>::type ≡ U
std::remove_reference<U&&>::type ≡ U
std::remove_reference<U>::type ≡ U
```

**static_cast**

以下语法形式将把表达式`t`转换为`T`类型的右值（准确的说是无名右值引用，是右值的一种）
```cpp
static_cast<T&&>(t)
```
无名的右值引用是右值，具名的右值引用是左值。

### std::move

**函数功能**

`std::move(t)`负责将表达式`t`转换为右值，使用这一转换意味着你不再关心`t`的内容，它可以通过被移动（窃取）来解决移动语意问题。

**源码与测试代码**

```cpp
001| template<typename _Tp>
002|   inline typename std::remove_reference<_Tp>::type&&
003|   move(_Tp&& __t)
004|   { return static_cast<typename std::remove_reference<_Tp>::type&&>(__t); }
```

```cpp
001| #include<iostream>
002| using namespace std;
003|
004| struct X {};
005| int main() {
006|     X a;
007|     X&& b = move(a);
008|     X&& c = move(X());
009| }
```
**代码说明**

1. 测试代码第7行用`X`类型的左值`a`来测试`move`函数，根据标准`X`类型的右值引用`b`只能绑定`X`类型的右值，所以`move(a)`的返回值必然是`X`类型的右值。
2. 测试代码第8行用`X`类型的右值`X()`来测试`move`函数，根据标准`X`类型的右值引用`c`只能绑定`X`类型的右值，所以`move(X())`的返回值必然是`X`类型的右值。
3. 首先我们来分析`move(a)`这种用左值参数来调用`move`函数的情况。
4. 模拟单步调用来到源码第3行，`_Tp&& ≡ X&, __t  ≡ a`。
5. 根据函数模板参数推导规则，`_Tp&& ≡ X&`可推出`_Tp ≡ X&`。
6. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type&& ≡ X&&`
7. 再次单步调用进入`move`函数实体所在的源码第4行。
8. `static_cast<typename std::remove_reference<_Tp>::type&&>(__t) ≡ static_cast<X&&>(a)`
9. 根据标准`static_cast<X&&>(a)`将把左值`a`转换为`X`类型的无名右值引用。
10. 然后我们再来分析`move(X())`这种用右值参数来调用`move`函数的情况。
11. 模拟单步调用来到源码第3行`_Tp&& ≡ X&&, __t  ≡ X()`。
12. 根据函数模板参数推导规则，`_Tp&& ≡ X&&`可推出`_Tp ≡ X`。
13. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type&& ≡ X&&`
14. 再次单步调用进入`move`函数实体所在的源码第4行。
15. `static_cast<typename std::remove_reference<_Tp>::type&&>(__t) ≡ static_cast<X&&>(X())`
16. 根据标准`static_cast<X&&>(X())`将把右值`X()`转换为`X`类型的无名右值引用。

由9和16可知源码中`std::move`函数的具体实现符合标准，因为无论用左值`a`还是右值`X()`做参数来调用`std::move`函数，该实现都将返回无名的右值引用（右值的一种），符合标准中该函数的定义。

### std::forward

**函数功能**

`std::forward<T>(u)`有两个参数：`T`与`u`。当`T`为左值引用类型时，`u`将被转换为`T`类型的左值，否则`u`将被转换为`T`类型右值。如此定义`std::forward`是为了在使用右值引用参数的函数模板中解决参数的完美转发问题。

**源码与测试代码**
