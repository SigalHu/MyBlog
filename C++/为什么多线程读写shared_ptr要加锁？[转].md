原文：[为什么多线程读写 shared_ptr 要加锁？](http://blog.csdn.net/solstice/article/details/8547547)

`shared_ptr`的引用计数本身是安全且无锁的，但对象的读写则不是，因为`shared_ptr`有两个数据成员，读写操作不能原子化。`shared_ptr`的线程安全级别和内建类型、标准库容器、`std::string`一样，即：

1. 一个`shared_ptr`对象实体可被多个线程同时读取
2. 两个`shared_ptr`对象实体可以被两个线程同时写入
3. 如果要从多个线程读写同一个`shared_ptr`对象，那么需要加锁

请注意，以上是`shared_ptr`对象本身的线程安全级别，不是它管理的对象的线程安全级别。

### `shared_ptr`的数据结构

`shared_ptr`是引用计数型（reference counting）智能指针，几乎所有的实现都采用在堆（heap）上放个计数值（count）的办法（除此之外理论上还有用循环链表的办法，不过没有实例）。具体来说，`shared_ptr<Foo>`包含两个成员，一个是指向`Foo`的指针`ptr`，另一个是`ref_count`指针（其类型不一定是原始指针，有可能是`class`类型，但不影响这里的讨论），指向堆上的`ref_count`对象。`ref_count`对象有多个成员，具体的数据结构如图所示，其中`deleter`和`allocator`是可选的。

![](为什么多线程读写shared_ptr要加锁？[转]/1.png)

为了简化并突出重点，后文只画出`use_count`的值：

![](为什么多线程读写shared_ptr要加锁？[转]/2.png)

以上是`shared_ptr<Foo> x(new Foo);`对应的内存数据结构。

如果再执行`shared_ptr<Foo> y = x;`那么对应的数据结构如下：

![](为什么多线程读写shared_ptr要加锁？[转]/3.png)

但是`y=x`涉及两个成员的复制，这两步拷贝不会同时（原子）发生。

* 步骤1：复制`ptr`指针：

![](为什么多线程读写shared_ptr要加锁？[转]/4.png)

* 步骤2：复制`ref_count`指针，导致引用计数加1：

![](为什么多线程读写shared_ptr要加锁？[转]/5.png)

步骤1和步骤2的先后顺序跟实现相关（因此步骤2里没有画出`y.ptr`的指向），我见过的都是先1后2。

既然`y=x`有两个步骤，如果没有`mutex`保护，那么在多线程里就有race condition。
