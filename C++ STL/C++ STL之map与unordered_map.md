### map

`map`是STL的一个关联容器，它提供一对一（第一个为`key`，每个`key`只能在`map`中出现一次，第二个为`value`）的数据处理能力。`map`内部自建一颗红黑树（一种非严格意义上的平衡二叉树），所以在`map`内部所有的数据都是有序的，且`map`的查询、插入、删除操作的时间复杂度都是`O(logN)`。在使用时，`map`的`key`需要定义`operator<`。

### unordered_map

`unordered_map`和`map`类似，都是存储的`key-value`的值，可以通过`key`快速索引到`value`。不同的是`unordered_map`不会根据`key`的大小进行排序，存储时是根据`key`的`hash`值判断元素是否相同，即`unordered_map`内部元素是无序的。`unordered_map`的`key`需要定义`hash_value`函数并且重载`operator==`。

`unordered_map`的底层是一个防冗余的哈希表（采用除留余数法）。哈希表最大的优点，就是把数据的存储和查找消耗的时间大大降低，时间复杂度为`O(1)`；而代价仅仅是消耗比较多的内存。

#### 基本原理

使用一个下标范围比较大的数组来存储元素。可以设计一个函数（哈希函数，也叫做散列函数），使得每个元素的`key`都与一个函数值（即数组下标，`hash`值）相对应，于是用这个数组单元来存储这个元素；也可以简单的理解为，按照`key`为每一个元素“分类”，然后将这个元素存储在相应“类”所对应的地方，称为桶。

但是，不能够保证每个元素的`key`与函数值是一一对应的，因此极有可能出现对于不同的元素，却计算出了相同的函数值，这样就产生了“冲突”，换句话说，就是把不同的元素分在了相同的“类”之中。 总的来说，“直接定址”与“解决冲突”是哈希表的两大特点。

一般可采用拉链法解决冲突：

![](C++%20STL之map与unordered_map/1.png)

**参考链接**

[map/unordered_map原理和使用整理](http://blog.csdn.net/blues1021/article/details/45054159)</br>
[解析unordeded_map和unordeded_set的底层实现](http://blog.csdn.net/turn__back/article/details/56005723)</br>
[Linux下map hash_map和unordered_map效率比较](http://blog.csdn.net/whizchen/article/details/9286557)</br>
[C++中map、hash_map、unordered_map、unordered_set通俗辨析](http://blog.csdn.net/u013195320/article/details/23046305)</br>
[C++11 新特性： unordered_map 与 map 的对比](http://www.cnblogs.com/NeilZhang/p/5724996.html)
