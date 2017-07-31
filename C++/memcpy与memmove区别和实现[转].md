原文：[memcpy memmove区别和实现](http://blog.csdn.net/hanchaoman/article/details/7937224)

`memcpy`与`memmove`的目的都是将`N`个字节的源内存地址的内容拷贝到目标内存地址中。

但当源内存和目标内存存在重叠时，`memcpy`会出现错误，而`memmove`能正确地实施拷贝，但这也增加了一点点开销。
