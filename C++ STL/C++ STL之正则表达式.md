正则表达式是C\+\+11标准库中新加入的强大工具。正则表达式是一种用于字符串处理的微型语言，适用于一些与字符串相关的操作。

C\+\+11包含了对以下几种语法的支持：ECMAScript、basic、extended、awk、grep和egrep。C\+\+11中使用的默认语法是ECMAScript。

### 匹配

#### regex_match

`regex_match()`算法可以用于比较一个给定源字符串和一个正则表达式模式，如果模式匹配整个源字符串，则返回`true`，否则返回`false`。
```cpp
#include <iostream>
#include <regex>
using namespace std;

int main() {
    string str = "sigalhu233";

    regex r("[a-z0-9]+");
    cout<<"正则表达式：[a-z0-9]+"<<endl;
    if(regex_match(str,r))
        cout<<"字符串："<<str<<" 匹配成功！"<<endl;
    else
        cout<<"字符串："<<str<<" 匹配失败！"<<endl;

    cout<<endl<<"正则表达式：\\d+"<<endl;
    if(regex_match(str,regex("\\d+")))
        cout<<"字符串："<<str<<" 匹配成功！"<<endl;
    else
        cout<<"字符串："<<str<<" 匹配失败！"<<endl;

    cout<<endl<<"正则表达式：\\d+"<<endl;
    if(regex_match(str.begin()+7,str.end(),regex("\\d+")))
        cout<<"字符串："<<&str[7]<<" 匹配成功！"<<endl;
    else
        cout<<"字符串："<<&str[7]<<" 匹配失败！"<<endl;

    smatch sm;
    cout<<endl<<"正则表达式：([a-z]+)(\\d+)"<<endl;
    if(regex_match(str.cbegin()+5,str.cend(),sm,regex("([a-z]+)(\\d+)"))){
        cout<<"字符串："<<&str[5]<<" 匹配成功！"<<endl;
        cout<<"匹配字符串个数："<<sm.size()<<endl;
        cout<<"分别为：";
        for(auto aa:sm)
            cout<<aa.str()<<" ";
        cout<<endl;
    } else
        cout<<"字符串："<<&str[5]<<" 匹配失败！"<<endl;

    cmatch cm;
    cout<<endl<<"正则表达式：([a-z]+)(\\d+)"<<endl;
    if(regex_match(str.c_str(),cm,regex("([a-z]+)(\\d+)"))){
        cout<<"字符串："<<str<<" 匹配成功！"<<endl;
        cout<<"匹配字符串个数："<<cm.size()<<endl;
        cout<<"分别为：";
        for(auto aa:cm)
            cout<<aa.str()<<" ";
        cout<<endl;
    } else
        cout<<"字符串："<<str<<" 匹配失败！"<<endl;
    return 0;
}
```
运行结果：
```
正则表达式：[a-z0-9]+
字符串：sigalhu233 匹配成功！

正则表达式：\d+
字符串：sigalhu233 匹配失败！

正则表达式：\d+
字符串：233 匹配成功！

正则表达式：([a-z]+)(\d+)
字符串：hu233 匹配成功！
匹配字符串个数：3
分别为：hu233 hu 233

正则表达式：([a-z]+)(\d+)
字符串：sigalhu233 匹配成功！
匹配字符串个数：3
分别为：sigalhu233 sigalhu 233
```

### 查找

#### regex_search

`regex_search()`算法可以在输入字符串中提取匹配的子字符串。`smatch`对象`sm`将包含搜索结果。如果要获得第一个捕捉组的字符串表达形式，可在代码中编写`m[1]`或`m[1].str()`。通过查看`m[1].first`和`m[1].second`迭代器可以得到这个子字符串在源字符串中出现的准确位置。
```cpp
#include <iostream>
#include <regex>
using namespace std;

int main() {
    string str = "sigalhu233sigal233hu233";
    smatch sm;

    cout<<"正则表达式：([a-z]+)2"<<endl;
    for(auto it=str.cbegin();regex_search(it,str.cend(),sm,regex("([a-z]+)2"));it=sm.suffix().first){
        cout<<"字符串："<<&*it<<" 匹配成功！"<<endl;
        cout<<"匹配字符子串个数："<<sm.size()<<endl;
        cout<<"分别为：";
        for(auto aa:sm)
            cout<<aa.str()<<" ";
        cout<<endl;
        cout<<"字符串 "<<sm.str()<<" 前的字符串为："<<sm.prefix().str()<<endl;
        cout<<"字符串 "<<sm.str()<<" 后的字符串为："<<sm.suffix().str()<<endl;
        cout<<endl;
    }
    return 0;
}
```
运行结果：
```
正则表达式：([a-z]+)2
字符串：sigalhu233sigal233hu233 匹配成功！
匹配字符子串个数：2
分别为：sigalhu2 sigalhu
字符串 sigalhu2 前的字符串为：
字符串 sigalhu2 后的字符串为：33sigal233hu233

字符串：33sigal233hu233 匹配成功！
匹配字符子串个数：2
分别为：sigal2 sigal
字符串 sigal2 前的字符串为：33
字符串 sigal2 后的字符串为：33hu233

字符串：33hu233 匹配成功！
匹配字符子串个数：2
分别为：hu2 hu
字符串 hu2 前的字符串为：33
字符串 hu2 后的字符串为：33
```

#### regex_iterator

为了逐一迭代正则查找的所有匹配成果，我们也可以使用`regex_iterator`。一般情况下，需要为某个特定的容器指定一个尾迭代器，但是对于`regex_iterator`，只有一个`end`值。只需要通过默认的构造函数声明一个`regex_iterator`类型，就可以获得这个尾迭代器：这个尾迭代器会被隐式地初始化为`end`值。
```cpp
#include <iostream>
#include <regex>
using namespace std;

int main() {
    string str = "sigalhu233sigal233hu233";
    regex reg("([a-z]+)2");

    cout<<"正则表达式：([a-z]+)2"<<endl;
    for(sregex_iterator it(str.begin(),str.end(),reg),end;it != end;it++){
        cout<<"字符串："<<&*it->prefix().first<<" 匹配成功！"<<endl;
        cout<<"匹配字符子串个数："<<it->size()<<endl;
        cout<<"分别为：";
        for(auto aa:*it)
            cout<<aa.str()<<" ";
        cout<<endl;
        cout<<"字符串 "<<it->str()<<" 前的字符串为："<<it->prefix().str()<<endl;
        cout<<"字符串 "<<it->str()<<" 后的字符串为："<<it->suffix().str()<<endl;
        cout<<endl;
    }
    return 0;
}
```
运行结果：
```
正则表达式：([a-z]+)2
字符串：sigalhu233sigal233hu233 匹配成功！
匹配字符子串个数：2
分别为：sigalhu2 sigalhu
字符串 sigalhu2 前的字符串为：
字符串 sigalhu2 后的字符串为：33sigal233hu233

字符串：33sigal233hu233 匹配成功！
匹配字符子串个数：2
分别为：sigal2 sigal
字符串 sigal2 前的字符串为：33
字符串 sigal2 后的字符串为：33hu233

字符串：33hu233 匹配成功！
匹配字符子串个数：2
分别为：hu2 hu
字符串 hu2 前的字符串为：33
字符串 hu2 后的字符串为：33
```

#### regex_token_iterator

`regex_iterator`有助于迭代“匹配合格”的子序列。然而有时候你会想处理那些子序列之间的内容，特别是当你打算将string拆分为一个个语汇单元`token`或以某个东西分割`string`，分隔符甚至可能被指定为一个正则表达式。`regex_token_iterator`就提供了这样的功能。

为了将它初始化，需要传给它字符序列的起点和终点，以及一个正则表达式。此外还可以指明一列整数值，用来表示语汇化过程中的元素：
* -1：表示你对每一个“匹配之正则表达式之间”或“语汇切分器之间”的子序列感兴趣
* 0：表示你对每一个匹配之正则表达式或语汇切分器感兴趣
* 任何其他数字$n$：表示你对正则表达式中的第$n$个匹配次表达式感兴趣

```cpp
#include <iostream>
#include <regex>
using namespace std;

int main() {
    string str = "11sigalhu233sigal244hu255";
    regex reg("([a-z]+)2");

    cout<<"正则表达式：([a-z]+)2"<<endl;
    cout<<"字符串为："<<str<<endl;
    for(sregex_token_iterator it(str.begin(),str.end(),reg),end;it != end;it++){
        cout<<"匹配到的字符串为："<<it->str()<<endl;
    }
    cout<<endl;

    for(sregex_token_iterator it(str.begin(),str.end(),reg,1),end;it != end;it++){
        cout<<"匹配到的字符串为："<<it->str()<<endl;
    }
    cout<<endl;

    for(sregex_token_iterator it(str.begin(),str.end(),reg,-1),end;it != end;it++){
        cout<<"匹配到的字符串为："<<it->str()<<endl;
    }
    cout<<endl;
    return 0;
}
```
运行结果：
```
正则表达式：([a-z]+)2
字符串为：11sigalhu233sigal244hu255
匹配到的字符串为：sigalhu2
匹配到的字符串为：sigal2
匹配到的字符串为：hu2

匹配到的字符串为：sigalhu
匹配到的字符串为：sigal
匹配到的字符串为：hu

匹配到的字符串为：11
匹配到的字符串为：33
匹配到的字符串为：44
匹配到的字符串为：55
```

### 替换

#### regex_replace

`regex_replace()`算法要求输入一个正则表达式，以及一个用于替换匹配子字符串的格式化字符串。这个格式化字符串可以通过转义序列引用匹配子字符串中的部分内容。

| 转义序列 | 替换为 |
| -- | -- |
| $n | 匹配第`n`个捕捉组的字符串。例如`$l`表示第一个捕捉组，`$2`表示第二个，依此类推 |
| $& | 匹配整个正则表达式的字符串，等同于`$0` |
| $` | 在源字符串中，在匹配正则表达式的子字符串左侧的部分 |
| $' | 在源字符串中，在匹配正则表达式的子字符串右侧的部分 |
| $$ | 美元符号 |

```cpp
#include <iostream>
#include <regex>
using namespace std;

int main() {
    string str = "11sigalhu22sigalhu33",str1;

    str1 = regex_replace(str,regex("s(igal)h(u)"),"SS$1HH$2");
    cout<<str1<<endl;
    str1 += regex_replace(str,regex("s(igal)h(u)"),"SS$1HH$2");
    cout<<str1<<endl;

    str1 = "123";
    regex_replace(str1.begin(),str.cbegin(),str.cend(),regex("s(igal)h(u)"),"SS$1HH$2");
    cout<<str1<<endl;
    regex_replace(back_inserter(str1),str.cbegin(),str.cend(),regex("s(igal)h(u)"),"SS$1HH$2");
    cout<<str1<<endl;
    return 0;
}
```
运行结果：
```
11SSigalHHu22SSigalHHu33
11SSigalHHu22SSigalHHu3311SSigalHHu22SSigalHHu33
11S
11S11SSigalHHu22SSigalHHu33
```
