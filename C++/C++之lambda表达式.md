lambda表达式的语法归纳如下：

![](C++之lambda表达式/1.jpeg)

1. capture子句（在 C++ 规范中也称为 lambda 引导）
2. 参数列表（可选）
3. 可变规范（可选）
4. 异常规范（可选）
4. 尾随返回类型（可选）
5. lambda函数体

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

除了捕获变量，lambda还可接受输入参数。参数列表是可选的，它在大多数方面类似于函数的参数列表。
```cpp
int y = [] (int first, int second) {
    return first + second;
};
```

#### 可变规范

通常，lambda的函数调用运算符为`const-by-value`，但对`mutable`关键字的使用可将其取消。它不会生成可变的数据成员。利用可变规范，lambda表达式的主体可以修改通过值捕获的变量。
```cpp
#include <iostream>
using namespace std;

int main() {
   int m = 0;
   int n = 0;
   [&, n] (int a) mutable { m = ++n + a; }(4);
   cout << m << " " << n << endl;
}
```
运行结果：
```
5 0
```
由于变量`n`是通过值捕获的，因此在调用lambda表达式后，变量的值仍保持`0`不变。`mutable`规范允许在lambda中修改`n`。

#### 异常规范

可以使用`throw()`异常规范来指示lambda表达式不会引发任何异常。
```cpp
#include <iostream>
using namespace std;

int main() {
    []() {
        try {
            throw 5;
        } catch (int ex){
            cout<<ex<<endl;
        }
    }();

    try {
        []() throw(int) {
            throw 5;
        }();
    } catch (int ex){
        cout<<ex<<endl;
    }
}
```

#### 返回类型

如果lambda体仅包含一个返回语句或其表达式不返回值，则可以省略lambda表达式的返回类型部分。如果lambda体包含单个返回语句，编译器将从返回表达式的类型推导返回类型。否则，编译器会将返回类型推导为`void`。
```cpp
auto x1 = [](int i){ return i; };  // 正确
auto x2 = []{ return {1, 2}; };  // 错误
```

#### lambda函数体

lambda表达式的lambda体可包含普通方法或函数的主体可包含的任何内容。例如，可以直接访问保存在全局数据区的变量，而不需要通过capture子句。
```cpp
void fillVector(vector<int>& v) {
    static int nextValue = 1;
    generate(v.begin(), v.end(), [] { return nextValue++; });
}
```

**参考链接**

[C++ 中的 Lambda 表达式](https://msdn.microsoft.com/zh-cn/library/dd293608.aspx)</br>
[C++11 学习笔记 lambda表达式](http://blog.csdn.net/fjzpdkf/article/details/50249287)</br>
[C++11 lambda 表达式解析](http://www.cnblogs.com/haippy/archive/2013/05/31/3111560.html)
