内省（`Introspector`）是`Java`语言对`JavaBean`类属性、事件的一种缺省处理方法。`JavaBean`是一种特殊的类，遵守`JavaBean`的规范，即提供默认构造方法，同时所有的属性都是私有的，且每个属性都有公开的读取和设定的方法（`getter`和`setter`），这些方法都遵守命名的规范。例如类`Person`中有属性`name`，可以通过`getName/setName`来得到其值或者设置新的值。

`Person`类：
```java
package com.sigalhu;

public class Person {
    private String name;
    private String sex;
    private int age;

    public Person() {
        this.name = "SigalHu";
        this.sex = "男";
        this.age = 26;
    }

    public Person(String name, String sex, int age) {
        this.name = name;
        this.sex = sex;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getSex() {
        return sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name=" + name +
                ", sex=" + sex +
                ", age=" + age +
                '}';
    }
}
```

首先通过`Introspector`类来获取某个对象的`BeanInfo`信息，然后通过`BeanInfo`来获取属性的描述器`PropertyDescriptor`，通过`PropertyDescriptor`就可以获取某个属性对应的`getter/setter`方法，因此就可以通过反射机制来调用这些方法。此外，还可以通过构造方法直接创建`PropertyDescriptor`对象来对某个对象的属性进行相应操作。

`PropertyDescriptor`类的主要方法有：
* `getPropertyType()`：获得属性的`Class`对象；
* `getReadMethod()`：获得用于读取属性值的方法；
* `getWriteMethod()`：获得用于写入属性值的方法；
* `hashCode()`：获取对象的哈希值；
* `setReadMethod(Method readMethod)`：设置用于读取属性值的方法；
* `setWriteMethod(Method writeMethod)`：设置用于写入属性值的方法。

`Main`类：
```java
package com.sigalhu;

import java.beans.BeanInfo;
import java.beans.Introspector;
import java.beans.PropertyDescriptor;

public class Main {
    static public void main(String[] args) throws Exception{
        Person person = new Person("a","男",25);

        //如果不想把父类的属性也列出来的话，getBeanInfo的第二个参数填写父类的信息
        BeanInfo beanInfo = Introspector.getBeanInfo(Person.class, Object.class);

        //读取所有属性值
        PropertyDescriptor[] props = beanInfo.getPropertyDescriptors();
        for(PropertyDescriptor prop:props){
            System.out.println(prop.getName() + ": " + prop.getReadMethod().invoke(person));
        }
        System.out.println();

        //写入指定属性值
        PropertyDescriptor propName = new PropertyDescriptor("name",Person.class);
        propName.getWriteMethod().invoke(person,"b");
        System.out.println(person);
        System.out.println();

        //设置用于写入属性值的方法
        PropertyDescriptor propSex = new PropertyDescriptor("sex",Person.class);
        propName.setWriteMethod(propSex.getWriteMethod());
        propName.getWriteMethod().invoke(person,"c");
        System.out.println(person);
    }
}
```
运行结果：
```java
age: 25
name: a
sex: 男

Person{name=b, sex=男, age=25}

Person{name=b, sex=c, age=25}
```

**参考链接**

[Java 内省机制](https://www.cnblogs.com/sunfie/p/4350941.html)<br>
[java的内省机制](http://geeksun.iteye.com/blog/539222)<br>
[Java内省机制小总结](http://blog.csdn.net/changqianger/article/details/45341651)<br>
[《JavaBean》-Java的内省机制](http://blog.csdn.net/zhoukun1008/article/details/52124242)<br>
[Java bean 是个什么概念？](https://www.zhihu.com/question/19773379)<br>
[Java内省机制](http://blog.csdn.net/hahalzb/article/details/5972421)

