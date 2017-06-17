### 强制转换运算符

有几种特定于C\+\+语言的转换运算符。这些运算符用于删除旧式C语言转换中的一些多义性和危险继承。这些运算符是：

* **dynamic_cast：**用于多态类型的转换
* **static_cast：**用于非多态类型的转换
* **const_cast：**用于删除const、volatile和__unaligned特性
* **reinterpret_cast：**用于位的简单重新解释
* **safe_cast：**用于生成可验证的MSIL
