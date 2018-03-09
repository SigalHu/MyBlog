#### 定义

`Java`反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为`Java`语言的反射机制。

`Java`反射机制容许程序在运行时加载、探知、使用编译期间完全未知的类。换言之，`Java`可以加载一个运行时才得知名称的类，获得其完整结构。

#### 功能

* 在运行时判断任意一个对象所属的类；
* 在运行时构造任意一个类的对象；
* 在运行时判断任意一个类所具有的成员变量和方法；
* 在运行时调用任意一个对象的方法；
* 生成动态代理。

反射机制允许程序在正在执行的过程中，利用`Reflection APIs`取得任何已知名称的类的内部信息，包括：`package`、`type parameters`、`superclass`、`implemented interfaces`、`inner classes`、`outer classes`、`fields`、`constructors`、`methods`、`modifiers`等，并可以在执行的过程中，动态生成对象、变更成员变量的值或调用成员方法。

#### `Class`类

`Class`类十分特殊。它和一般类一样继承自`Object`，其实体用以表达`Java`程序运行时的`classes`和`interfaces`，也用来表达`enum`、`array`、`primitive Java types`（`boolean`，`byte`，`char`，`short`，`int`，`long`，`float`，`double`）以及关键词`void`。当一个类被加载，或当类加载器（`class loader`）的`defineClass()`被`JVM`调用，`JVM`便自动产生一个`Class`对象。

`Class`没有公共构造方法。`Class`对象是在加载类时由`JVM`以及通过调用类加载器中的`defineClass()`方法自动构造的。一个类在`JVM`中只会有一个`Class`对象。

换句话说，当我们定义好一个类文件并编译成`.class`字节码后，编译器同时为我们创建了一个`Class`对象并将它保存`.class`文件中（某些书上直接把`.class`文件称之为`Class`对象）。同时在`JVM`内部有一个类加载机制，即在需要的时候（懒加载）将`.class`文件和对应的`Class`对象加载到内存中。

| 获取`Class`对象的方法 | 示例 |
|:--|:--|
|`getClass()`<br>注：每个`Class`都有此函数|`String str = "abc";`<br>`Class c1 = str.getClass();`|
|`Class.getSuperClass()`|`Button b = new Button();`<br>`Class c1 = b.getClass();`<br>`Class c2 = c1.getSuperClass();`|
|静态方法`Class.forName()`|`Class c1 = Class.forName("java.lang.String")`|
|`.class`属性|`Class c1 = String.class;`<br>`Class c2 = int.class;`|
|基本类型包装类的`TYPE`语法|`Class c1 = Boolean.TYPE`|

#### 示例

**1. 基础使用**

`Person`类：
```java
package com.sigalhu;

public class Person {
    public String name;
    protected String sex;
    private int age;

    public Person() {
        System.out.println("调用无参构造方法");
        this.name = "SigalHu";
        this.sex = "男";
        this.age = 26;
    }

    public Person(String name, String sex, int age) {
        System.out.println("调用public有参构造方法");
        this.name = name;
        this.sex = sex;
        this.age = age;
    }

    protected Person(String name, String sex){
        System.out.println("调用protected有参构造方法");
        this.name = name;
        this.sex = sex;
        this.age = 0;
    }

    private Person(String name){
        System.out.println("调用private有参构造方法");
        this.name = name;
        this.sex = "男";
        this.age = 0;
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

    private void privateMethod(){
        System.out.println("调用private方法");
    }

    protected void protectedMethod(){
        System.out.println("调用protected方法");
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
`Main`类：
```java
package com.sigalhu;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    static public void main(String[] args)
            throws ClassNotFoundException, IllegalAccessException,
            InstantiationException, NoSuchMethodException, InvocationTargetException,
            NoSuchFieldException{
        Class cl = null;

        System.out.println("*************** 获取目标类的Class实例 *************");
        //1.直接通过类名.class的方式得到
        cl = Person.class;
        System.out.println("通过类名: " + cl);

        //2.通过对象的getClass()方法获取,这个使用的少（一般是传的是Object，不知道是什么类型的时候才用）
        Object obj = new Person("a","男",1);
        cl = obj.getClass();
        System.out.println("通过getClass(): " + cl);

        //3.通过全类名获取，用的比较多，但可能抛出ClassNotFoundException异常
        cl = Class.forName("com.sigalhu.Person");
        System.out.println("通过全类名获取: " + cl);
        System.out.println();

        System.out.println("***************获取目标类的构造方法****************");
        //1.获取所有具有public属性的构造方法
        Constructor[] constructors = cl.getConstructors();
        System.out.println("getConstructors: ");
        for (Constructor constructor : constructors){
            System.out.println(constructor);
        }
        System.out.println();

        //2.获取所有的构造方法（不分public和非public属性）
        constructors = cl.getDeclaredConstructors();
        System.out.println("getDeclaredConstructors: ");
        for (Constructor constructor : constructors){
            System.out.println(constructor);
        }
        System.out.println();

        //3.根据构造方法的参数，返回一个具体的构造函数（不分public和非public属性）
        Constructor constructor = cl.getDeclaredConstructor(String.class);
        //若该构造方法是私有的，需要调用setAccessible(true)方法（目标类内除外）
        constructor.setAccessible(true);
        System.out.println("getDeclaredConstructor: ");
        System.out.println(constructor);
        System.out.println();

        //4.根据构造方法的参数，获取一个具体的具有public属性的构造方法
        constructor = cl.getConstructor(String.class,String.class,int.class);
        System.out.println("getConstructor: ");
        System.out.println(constructor);
        System.out.println();

        System.out.println("*********************创建目标对象*********************");
        //1.通过newInstance方法调用目标类的无参public构造方法创建目标对象
        obj = cl.newInstance();
        System.out.println(obj);

        //2.通过反射得到有参构造方法并创建目标对象
        constructor.newInstance("b","女",2);
        System.out.println(obj);
        System.out.println();

        System.out.println("***************获取目标类的成员方法***************");
        //1.获取所有具有public属性的成员方法（包括继承的方法）
        Method[] methods =cl.getMethods();
        System.out.println("getMethods: ");
        for (Method method : methods){
            System.out.println(method);
        }
        System.out.println();

        //2.获取所有在目标类中声明的成员方法（不包括继承的方法）
        methods = cl.getDeclaredMethods();
        System.out.println("getDeclaredMethods: ");
        for (Method method: methods){
            System.out.println(method);
        }
        System.out.println();

        //3.根据方法名和参数，返回一个具体的具有public属性的方法
        Method method = cl.getMethod("setName",String.class);
        System.out.println("getMethod: ");
        System.out.println(method);
        System.out.println();

        //4.根据方法名和参数，返回一个具体的方法（不分public和非public属性）
        method = cl.getDeclaredMethod("privateMethod");
        //若该方法是私有的，需要调用setAccessible(true)方法（目标类内除外）
        method.setAccessible(true);
        System.out.println("getDeclaredMethod: ");
        System.out.println(method);
        System.out.println();

        System.out.println("*********************执行方法*********************");
        method.invoke(obj);
        System.out.println();

        System.out.println("***************获取目标类的成员变量****************");
        //1.获取具有public属性的成员变量
        Field[] fields = cl.getFields();
        System.out.println("getFields: ");
        for (Field field: fields) {
            System.out.println(field);
        }
        System.out.println();

        //2.获取所有成员变量（不分public和非public属性）
        fields = cl.getDeclaredFields();
        System.out.println("getDeclaredFields: ");
        for (Field field: fields) {
            System.out.println(field);
        }
        System.out.println();

        //3.根据变量名，获取一个具体的具有public属性的成员变量
        Field field = cl.getField("name");
        System.out.println("getField: ");
        System.out.println(field);

        //4.根据变量名，获取一个成员变量（不分public和非public属性）
        field = cl.getDeclaredField("age");
        //若该字段是私有的，需要调用setAccessible(true)方法（目标类内除外）
        field.setAccessible(true);
        System.out.println("getDeclaredField: ");
        System.out.println(field);
        System.out.println();

        System.out.println("************获取目标对象相应成员变量的值************");
        Person person = (Person)obj;
        System.out.println(person);
        Object val = field.get(person);
        System.out.println(field.getName() + ": " + val);
        System.out.println();

        System.out.println("************设置目标对象相应成员变量的值*************");
        field.set(person, 100);
        System.out.println(person);
    }
}
```
运行结果：
```java
*************** 获取目标类的Class实例 *************
通过类名: class com.sigalhu.Person
调用public有参构造方法
通过getClass(): class com.sigalhu.Person
通过全类名获取: class com.sigalhu.Person

***************获取目标类的构造方法****************
getConstructors: 
public com.sigalhu.Person(java.lang.String,java.lang.String,int)
public com.sigalhu.Person()

getDeclaredConstructors: 
private com.sigalhu.Person(java.lang.String)
protected com.sigalhu.Person(java.lang.String,java.lang.String)
public com.sigalhu.Person(java.lang.String,java.lang.String,int)
public com.sigalhu.Person()

getDeclaredConstructor: 
private com.sigalhu.Person(java.lang.String)

getConstructor: 
public com.sigalhu.Person(java.lang.String,java.lang.String,int)

*********************创建目标对象*********************
调用无参构造方法
Person{name=SigalHu, sex=男, age=26}
调用public有参构造方法
Person{name=SigalHu, sex=男, age=26}

***************获取目标类的成员方法***************
getMethods: 
public java.lang.String com.sigalhu.Person.toString()
public java.lang.String com.sigalhu.Person.getName()
public void com.sigalhu.Person.setName(java.lang.String)
public java.lang.String com.sigalhu.Person.getSex()
public void com.sigalhu.Person.setSex(java.lang.String)
public int com.sigalhu.Person.getAge()
public void com.sigalhu.Person.setAge(int)
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()

getDeclaredMethods: 
public java.lang.String com.sigalhu.Person.toString()
public java.lang.String com.sigalhu.Person.getName()
public void com.sigalhu.Person.setName(java.lang.String)
private void com.sigalhu.Person.privateMethod()
public java.lang.String com.sigalhu.Person.getSex()
public void com.sigalhu.Person.setSex(java.lang.String)
public int com.sigalhu.Person.getAge()
public void com.sigalhu.Person.setAge(int)
protected void com.sigalhu.Person.protectedMethod()

getMethod: 
public void com.sigalhu.Person.setName(java.lang.String)

getDeclaredMethod: 
private void com.sigalhu.Person.privateMethod()

*********************执行方法*********************
调用private方法

***************获取目标类的成员变量****************
getFields: 
public java.lang.String com.sigalhu.Person.name

getDeclaredFields: 
public java.lang.String com.sigalhu.Person.name
protected java.lang.String com.sigalhu.Person.sex
private int com.sigalhu.Person.age

getField: 
public java.lang.String com.sigalhu.Person.name
getDeclaredField: 
private int com.sigalhu.Person.age

************获取目标对象相应成员变量的值************
Person{name=SigalHu, sex=男, age=26}
age: 26

************设置目标对象相应成员变量的值*************
Person{name=SigalHu, sex=男, age=100}
```

**2. 通过反射越过泛型检查**

泛型用在编译期，编译过后泛型擦除，所以可以通过反射越过泛型检查。

```java
package com.sigalhu;

import java.lang.reflect.Method;
import java.util.ArrayList;

/* 
 * 通过反射越过泛型检查 
 *  
 * 例如：有一个String泛型的集合，怎样能向这个集合中添加一个Integer类型的值？ 
 */
public class Demo {
    public static void main(String[] args) throws Exception{
        ArrayList<String> strList = new ArrayList<>();
        strList.add("aaa");
        strList.add("bbb");

        //  strList.add(100);
        //获取ArrayList的Class对象，反向的调用add()方法，添加数据
        Class listClass = strList.getClass(); //得到 strList 对象的字节码 对象
        //获取add()方法
        Method m = listClass.getMethod("add", Object.class);
        //调用add()方法
        m.invoke(strList, 100);

        //遍历集合
        for(Object obj : strList){
            System.out.println(obj);
        }
    }
}
```
运行结果：
```
aaa
bbb
100
```

#### 注意

* 性能第一。反射包括了一些动态类型，`JVM`无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。

* 安全限制。使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如`Applet`，那么这就是个问题了。

* 内部暴露。由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用－－代码有功能上的错误，降低可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。

**参考链接**

[Java反射机制详解](https://www.cnblogs.com/bojuetech/p/5896551.html)<br>
[深入理解java反射机制](http://blog.csdn.net/u012585964/article/details/52011138)<br>
[Java反射机制及IoC原理](https://www.cnblogs.com/Eason-S/p/5851078.html)<br>
[反射setAccessible()方法](http://blog.csdn.net/kjfcpua/article/details/8496911)<br>
[Java基础之—反射](http://blog.csdn.net/sinat_38259539/article/details/71799078)<br>
[粗浅看java反射机制](http://www.importnew.com/20339.html)<br>
[Java反射机制及IoC原理](https://www.cnblogs.com/Eason-S/p/5851078.html)<br>
[Class对象和Java反射机制](http://www.importnew.com/21235.html)