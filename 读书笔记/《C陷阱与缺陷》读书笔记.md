如果一个整型常数的第一个字符是数字0,那么该常量将被视作八进制数。如0195相当于十进制数141

---

C语言中，else始终与同一对括号内最近的未匹配的if结合。

---

Switch语句中的case语句若在结尾处无break，程序将会继续执行下一条case语句

---

非数组的指针
```cpp
#include <stdlib.h>
char *r,*s = "hhh",*t = "sigalhu";
r = (char*)malloc(strlen(s) + strlen(t) + 1);
if(!r) {
	exit(1);
	}
strcpy(r, s);
strcat(r, t);
free(r);
```

---

extern修饰变量的声明。举例来说，**如果文件a.c需要引用b.c中变量int v，就可以在a.c中声明extern int v，然后就可以引用变量v。这里需要注意的是，被引用的变量v的链接属性必须是外链接（external）的，也就是说a.c要引用到v，不只是取决于在a.c中声明extern int v，还取决于变量v本身是能够被引用到的。** 这涉及到c语言的另外一个话题－－变量的作用域。能够被其他模块以extern修饰符引用到的变量通常是全局变量。还有很重要的一点是，extern int v可以放在a.c中的任何地方，比如你可以在a.c中的函数fun定义的开头处声明extern int v，然后就可以引用到变量v了，只不过这样只能在函数fun作用域中引用v罢了，这还是变量作用域的问题。对于这一点来说，很多人使用的时候都心存顾虑。好像extern声明只能用于文件作用域似的。

---

**static可以把变量和函数的作用域限制在一个源文件中，避免命名冲突。**
