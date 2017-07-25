lambda表达式的语法归纳如下：
```cpp
[ caputrue ] ( params ) opt -> ret { body; };
```
1. `capture`是捕获列表
2. `params`是参数表(选填)
3. `opt`是函数选项；可以填`mutable`，`exception`，`attribute`（选填）
    * `mutable`说明lambda表达式体内的代码可以修改被捕获的变量，并且可以访问被捕获的对象的non-const方法
    * `exception`说明lambda表达式是否抛出异常以及何种异常
    * `attribute`用来声明属性
4. `ret`是返回值类型(选填)
5. `body`是函数体

#### capture子句

capture子句指定要捕获的变量以及是通过值还是引用进行捕获。有与号`&`前缀的变量通过引用访问，没有该前缀的变量通过值访问。空capture子句`[]`指示 lambda 表达式的主体不访问封闭范围中的变量。

可以使用默认捕获模式来指示如何捕获lambda中引用的任何外部变量：`[&]`表示通过引用捕获引用的所有变量，而`[=]`表示通过值捕获它们。另外，可以使用默认捕获模式，然后为特定变量显式指定相反的模式。
```cpp
[a,&b]   // a变量以值的方式呗捕获，b以引用的方式被捕获
[this]   // 以值的方式捕获this指针
[&]      // 以引用的方式捕获所有的外部自动变量
[=]      // 以值的方式捕获所有的外部自动变量
[]       // 不捕获外部的任何变量
[=,&a] // 按值捕获外部作用域中所有变量，并按引用捕获a变量。
```

#### 参数列表

**参考链接**

[C++ 中的 Lambda 表达式](https://msdn.microsoft.com/zh-cn/library/dd293608.aspx)</br>
[C++11 学习笔记 lambda表达式](http://blog.csdn.net/fjzpdkf/article/details/50249287)</br>
[C++11 lambda 表达式解析](http://www.cnblogs.com/haippy/archive/2013/05/31/3111560.html)
