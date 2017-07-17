原文：[emplace_back() 和 push_back 的区别](http://blog.csdn.net/xiaolewennofollow/article/details/52559364)

在引入右值引用，转移构造函数，转移复制运算符之前，通常使用`push_back()`向容器中加入一个右值元素（临时对象）的时候，首先会调用构造函数构造这个临时对象，然后需要调用拷贝构造函数将这个临时对象放入容器中。原来的临时变量释放。这样造成的问题是临时变量申请的资源就浪费。

引入了右值引用，转移构造函数后，`push_back()`右值时就会调用构造函数和转移构造函数。

在这上面有进一步优化的空间就是使用`emplace_back()`，使用`emplace_back()`在容器尾部添加一个元素，这个元素原地构造，不需要触发拷贝构造和转移构造。而且调用形式更加简洁，直接根据参数初始化临时对象的成员。

测试代码：
```cpp
#include <vector>
#include <string>
#include <iostream>

struct President  {
    std::string name;
    std::string country;
    int year;

    President(std::string p_name, std::string p_country, int p_year)
        : name(std::move(p_name)), country(std::move(p_country)), year(p_year)  {
        std::cout << "I am being constructed.\n";
    }
    President(const President& other)
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year){
        std::cout << "I am being copy constructed.\n";
    }
    President(President&& other)
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)  {
        std::cout << "I am being moved.\n";
    }
    President& operator=(const President& other);
};

int main()  {
    std::vector<President> elections;
    std::cout << "emplace_back:\n";
    elections.emplace_back("Nelson Mandela", "South Africa", 1994); //没有类的创建

    std::vector<President> reElections;
    std::cout << "\npush_back:\n";
    reElections.push_back(President("Franklin Delano Roosevelt", "the USA", 1936));

    std::cout << "\nContents:\n";
    for (President const& president: elections) {
       std::cout << president.name << " was elected president of "
            << president.country << " in " << president.year << ".\n";
    }
    for (President const& president: reElections) {
        std::cout << president.name << " was re-elected president of "
            << president.country << " in " << president.year << ".\n";
    }

}
```
测试结果：
```
emplace_back:
I am being constructed.

push_back:
I am being constructed.
I am being moved.

Contents:
Nelson Mandela was elected president of South Africa in 1994.
Franklin Delano Roosevelt was re-elected president of the USA in 1936.
```
