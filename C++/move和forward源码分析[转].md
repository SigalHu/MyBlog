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
```cpp
001| /// forward (as per N3143)
002| template<typename _Tp>
003|   inline _Tp&&
004|   forward(typename std::remove_reference<_Tp>::type& __t)
005|   { return static_cast<_Tp&&>(__t); }
006|
007| template<typename _Tp>
008|   inline _Tp&&
009|   forward(typename std::remove_reference<_Tp>::type&& __t)
010|   {
011|     static_assert(!std::is_lvalue_reference<_Tp>::value, "template argument"
012|     " substituting _Tp is an lvalue reference type");
013|     return static_cast<_Tp&&>(__t);
014|   }
```
```cpp
001| #include<iostream>
002| using namespace std;
003|
004| struct X {};
005| void inner(const X&) {cout << "inner(const X&)" << endl;}
006| void inner(X&&) {cout << "inner(X&&)" << endl;}
007| template<typename T>
008| void outer(T&& t) {inner(forward<T>(t));}
009|
010| int main()
011| {
012|     X a;
013|     outer(a);
014|     outer(X());
015|     inner(forward<X>(X()));
016| }
```
运行结果：
```
inner(const X&)
inner(X&&)
inner(X&&)
```

**代码说明**

1. 测试代码第13行用`X`类型的左值`a`来测试`forward`函数，程序输出表明`outer(a)`调用的是`inner(const X&)`版本，从而证明函数模板`outer`调用`forward`函数在将参数左值`a`转发给了`inner`函数时，成功地保留了参数`a`的左值属性。
2. 测试代码第14行用`X`类型的右值`X()`来测试`forward`函数，程序输出表明`outer(X())`调用的是`inner(X&&)`版本，从而证明函数模板`outer`调用`forward`函数在将参数右值`X()`转发给了`inner`函数时，成功地保留了参数`X()`的右值属性。
3. 首先我们来分析`outer(a)`这种调用`forward`函数转发左值参数的情况。
4. 模拟单步调用来到测试代码第8行，`T&& ≡ X&, t ≡ a`。
5. 根据函数模板参数推导规则，`T&& ≡ X&`可推出`T ≡ X&`。
6. `forward<T>(t) ≡ forward<X&>(t)`，其中`t`为指向`a`的左值引用。
7. 再次单步调用进入`forward`函数实体所在的源码第4行或第9行。
8. 先尝试匹配源码第4行的`forward`函数，`_Tp ≡ X&`。
9. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type& ≡ X&`
10. 形参`__t`与实参`t`类型相同，因此函数匹配成功。
11. 再尝试匹配源码第9行的`forward`函数，`_Tp ≡ X&`。
12. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type&& ≡ X&&`
13. 形参`__t`与实参`t`类型不同，因此函数匹配失败。
14. 由10与13可知7单步调用实际进入的是源码第4行的`forward`函数。
15. `static_cast<_Tp&&>(__t) ≡ static_cast<X&>(t) ≡ a`
16. `inner(forward<T>(t)) ≡ inner(static_cast<X&>(t)) ≡ inner(a) `
17. `outer(a) ≡ inner(forward<T>(t)) ≡ inner(a)`，再次单步调用将进入测试代码第5行的`inner(const X&)`版本，左值参数转发成功。
18. 然后我们来分析`outer(X())`这种调用`forward`函数转发右值参数的情况。
19. 模拟单步调用来到测试代码第8行，`T&& ≡ X&&, t ≡ X()`。
20. 根据函数模板参数推导规则，`T&& ≡ X&&`可推出`T ≡ X`。
21. `forward<T>(t) ≡ forward<X>(t)`，其中`t`为指向`X()`的右值引用。
22. 再次单步调用进入`forward`函数实体所在的源码第4行或第9行。
23. 先尝试匹配源码第4行的`forward`函数，`_Tp ≡ X`。
24. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type& ≡ X&`
25. 形参`__t`与实参`t`类型相同，因此函数匹配成功。
26. 再尝试匹配源码第9行的`forward`函数，`_Tp ≡ X`。
27. `typename std::remove_reference<_Tp>::type ≡ X`
`typename std::remove_reference<_Tp>::type&& ≡ X&&`
28. 形参`__t`与实参`t`类型不同，因此函数匹配失败。
29. 由25与28可知22单步调用**实际进入的仍然是源码第4行的forward函数**。
30. `static_cast<_Tp&&>(__t) ≡ static_cast<X&&>(t) ≡ X()`
31. `inner(forward<T>(t)) ≡ inner(static_cast<X&&>(t))  ≡ inner(X())`
32. `outer(X()) ≡ inner(forward<T>(t)) ≡ inner(X())`，再次单步调用将进入测试代码第6行的`inner(X&&)`版本，右值参数转发成功。

由17和32可知，源码中`std::forward`函数的具体实现符合标准，因为无论用左值`a`还是右值`X()`做参数来调用带有右值引用参数的函数模板`outer`，只要在`outer`函数内使用`std::forward`函数转发参数，就能保留参数的左右值属性，从而实现了函数模板参数的完美转发。
