原文：[Java泛型详解](http://www.importnew.com/24029.html)

#### 引言

泛型是`Java`中一个非常重要的知识点，在`Java`集合类框架中泛型被广泛应用。本文我们将从零开始来看一下`Java`泛型的设计，将会涉及到通配符处理，以及让人苦恼的类型擦除。

#### 泛型类

我们首先定义一个简单的`Box`类：

```java
public class Box {
    private String object;
    public void set(String object) { this.object = object; }
    public String get() { return object; }
}
```

这是最常见的做法，这样做的一个坏处是`Box`里面现在只能装入`String`类型的元素，今后如果我们需要装入`Integer`等其他类型的元素，还必须要另外重写一个`Box`，代码得不到复用，使用泛型可以很好的解决这个问题。

```java
public class Box<T> {
    // T stands for "Type"
    private T t;
    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

这样我们的`Box`类便可以得到复用，我们可以将`T`替换成任何我们想要的类型：

```java
Box<Integer> integerBox = new Box<Integer>();
Box<Double> doubleBox = new Box<Double>();
Box<String> stringBox = new Box<String>();
```

#### 泛型方法

看完了泛型类，接下来我们来了解一下泛型方法。声明一个泛型方法很简单，只要在返回类型前面加上一个类似`<K, V>`的形式就行了：

```java
public class Util {
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {
        return p1.getKey().equals(p2.getKey()) &&
               p1.getValue().equals(p2.getValue());
    }
}
public class Pair<K, V> {
    private K key;
    private V value;
    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }
    public void setKey(K key) { this.key = key; }
    public void setValue(V value) { this.value = value; }
    public K getKey()   { return key; }
    public V getValue() { return value; }
}
```

我们可以像下面这样去调用泛型方法：

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.<Integer, String>compare(p1, p2);
```

或者在`Java1.7/1.8`利用`type inference`，让`Java`自动推导出相应的类型参数：

```java
Pair<Integer, String> p1 = new Pair<>(1, "apple");
Pair<Integer, String> p2 = new Pair<>(2, "pear");
boolean same = Util.compare(p1, p2);
```

#### 边界符

现在我们要实现这样一个功能，查找一个泛型数组中大于某个特定元素的个数，我们可以这样实现：

```java
public static <T> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e > elem)  // compiler error
            ++count;
    return count;
}
```

但是这样很明显是错误的，因为除了`short, int, double, long, float, byte, char`等原始类型，其他的类并不一定能使用操作符>，所以编译器报错，那怎么解决这个问题呢？答案是使用边界符。

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```

做一个类似于下面这样的声明，这样就等于告诉编译器类型参数`T`代表的都是实现了`Comparable`接口的类，这样等于告诉编译器它们都至少实现了`compareTo`方法。

```java
public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
    int count = 0;
    for (T e : anArray)
        if (e.compareTo(elem) > 0)
            ++count;
    return count;
}
```

#### 通配符

在了解通配符之前，我们首先必须要澄清一个概念，还是借用我们上面定义的`Box`类，假设我们添加一个这样的方法：

```java
public void boxTest(Box<Number> n) { /* ... */ }
```

那么现在`Box<Number> n`允许接受什么类型的参数？我们是否能够传入`Box<Integer>`或者`Box<Double>`呢？答案是否定的，虽然`Integer`和`Double`是`Number`的子类，但是在泛型中`Box<Integer>`或者`Box<Double>`与`Box<Number>`之间并没有任何的关系。这一点非常重要，接下来我们通过一个完整的例子来加深一下理解。

首先我们先定义几个简单的类，下面我们将用到它：

```java
class Fruit {}
class Apple extends Fruit {}
class Orange extends Fruit {}
```

下面这个例子中，我们创建了一个泛型类`Reader`，然后在`f1()`中当我们尝试`Fruit f = fruitReader.readExact(apples);`编译器会报错，因为`List<Fruit>`与`List<Apple>`之间并没有任何的关系。

```java
public class GenericReading {
    static List<Apple> apples = Arrays.asList(new Apple());
    static List<Fruit> fruit = Arrays.asList(new Fruit());
    static class Reader<T> {
        T readExact(List<T> list) {
            return list.get(0);
        }
    }
    static void f1() {
        Reader<Fruit> fruitReader = new Reader<Fruit>();
        // Errors: List<Fruit> cannot be applied to List<Apple>.
        // Fruit f = fruitReader.readExact(apples);
    }
    public static void main(String[] args) {
        f1();
    }
}
```

但是按照我们通常的思维习惯，`Apple`和`Fruit`之间肯定是存在联系，然而编译器却无法识别，那怎么在泛型代码中解决这个问题呢？我们可以通过使用通配符来解决这个问题：

```java
static class CovariantReader<T> {
    T readCovariant(List<? extends T> list) {
        return list.get(0);
    }
}
static void f2() {
    CovariantReader<Fruit> fruitReader = new CovariantReader<Fruit>();
    Fruit f = fruitReader.readCovariant(fruit);
    Fruit a = fruitReader.readCovariant(apples);
}
public static void main(String[] args) {
    f2();
}
```

这样就相当与告诉编译器，`fruitReader`的`readCovariant`方法接受的参数只要是满足`Fruit`的子类就行(包括`Fruit`自身)，这样子类和父类之间的关系也就关联上了。

#### `PECS`原则

上面我们看到了类似`<? extends T>`的用法，利用它我们可以从`list`里面`get`元素，那么我们可不可以往`list`里面`add`元素呢？我们来尝试一下：

```java
public class GenericsAndCovariance {
    public static void main(String[] args) {
        // Wildcards allow covariance:
        List<? extends Fruit> flist = new ArrayList<Apple>();
        // Compile Error: can't add any type of object:
        // flist.add(new Apple())
        // flist.add(new Orange())
        // flist.add(new Fruit())
        // flist.add(new Object())
        flist.add(null); // Legal but uninteresting
        // We Know that it returns at least Fruit:
        Fruit f = flist.get(0);
    }
}
```

答案是否定，`Java`编译器不允许我们这样做，为什么呢？对于这个问题我们不妨从编译器的角度去考虑。因为`List<? extends Fruit> flist`它自身可以有多种含义：

```java
List<? extends Fruit> flist = new ArrayList<Fruit>();
List<? extends Fruit> flist = new ArrayList<Apple>();
List<? extends Fruit> flist = new ArrayList<Orange>();
```

* 当我们尝试`add`一个`Apple`的时候，`flist`可能指向`new ArrayList<Orange>()`；
* 当我们尝试`add`一个`Orange`的时候，`flist`可能指向`new ArrayList<Apple>()`；
* 当我们尝试`add`一个`Fruit`的时候，这个`Fruit`可以是任何类型的`Fruit`，而`flist`可能只想某种特定类型的`Fruit`，编译器无法识别所以会报错。

所以对于实现了`<? extends T>`的集合类只能将它视为`Producer`向外提供（`get`）元素，而不能作为`Consumer`来对外获取（`add`）元素。

如果我们要`add`元素应该怎么做呢？可以使用`<? super T>`：

```java
public class GenericWriting {
    static List<Apple> apples = new ArrayList<Apple>();
    static List<Fruit> fruit = new ArrayList<Fruit>();
    static <T> void writeExact(List<T> list, T item) {
        list.add(item);
    }
    static void f1() {
        writeExact(apples, new Apple());
        writeExact(fruit, new Apple());
    }
    static <T> void writeWithWildcard(List<? super T> list, T item) {
        list.add(item)
    }
    static void f2() {
        writeWithWildcard(apples, new Apple());
        writeWithWildcard(fruit, new Apple());
    }
    public static void main(String[] args) {
        f1(); f2();
    }
}
```

这样我们可以往容器里面添加元素了，但是使用`super`的坏处是以后不能`get`容器里面的元素了，原因很简单，我们继续从编译器的角度考虑这个问题，对于`List<? super Apple> list`，它可以有下面几种含义：

```java
List<? super Apple> list = new ArrayList<Apple>();
List<? super Apple> list = new ArrayList<Fruit>();
List<? super Apple> list = new ArrayList<Object>();
```

当我们尝试通过`list`来`get`一个`Apple`的时候，可能会`get`得到一个`Fruit`，这个`Fruit`可以是`Orange`等其他类型的`Fruit`。

根据上面的例子，我们可以总结出一条规律，`Producer Extends, Consumer Super`：

* `Producer Extends` – 如果你需要一个只读`List`，用它来`produce T`，那么使用`? extends T`。
* `Consumer Super` – 如果你需要一个只写`List`，用它来`consume T`，那么使用`? super T`。
* 如果需要同时读取以及写入，那么我们就不能使用通配符了。

如何阅读过一些`Java`集合类的源码，可以发现通常我们会将两者结合起来一起用，比如像下面这样：

```java
public class Collections {
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i=0; i<src.size(); i++)
            dest.set(i, src.get(i));
    }
}
```

#### 类型擦除

`Java`泛型中最令人苦恼的地方或许就是类型擦除了，特别是对于有`C++`经验的程序员。类型擦除就是说`Java`泛型只能用于在编译期间的静态类型检查，然后编译器生成的代码会擦除相应的类型信息，这样到了运行期间实际上`JVM`根本就知道泛型所代表的具体类型。这样做的目的是因为`Java`泛型是`1.5`之后才被引入的，为了保持向下的兼容性，所以只能做类型擦除来兼容以前的非泛型代码。对于这一点，如果阅读`Java`集合框架的源码，可以发现有些类其实并不支持泛型。

说了这么多，那么泛型擦除到底是什么意思呢？我们先来看一下下面这个简单的例子：

```java
public class Node<T> {
    private T data;
    private Node<T> next;
    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }
    public T getData() { return data; }
    // ...
}
```

编译器做完相应的类型检查之后，实际上到了运行期间上面这段代码实际上将转换成：

```java
public class Node {
    private Object data;
    private Node next;
    public Node(Object data, Node next) {
        this.data = data;
        this.next = next;
    }
    public Object getData() { return data; }
    // ...
}
```

这意味着不管我们声明`Node<String>`还是`Node<Integer>`，到了运行期间，`JVM`统统视为`Node<Object>`。有没有什么办法可以解决这个问题呢？这就需要我们自己重新设置`bounds`了，将上面的代码修改成下面这样：

```java
public class Node<T extends Comparable<T>> {
    private T data;
    private Node<T> next;
    public Node(T data, Node<T> next) {
        this.data = data;
        this.next = next;
    }
    public T getData() { return data; }
    // ...
}
```

这样编译器就会将`T`出现的地方替换成`Comparable`而不再是默认的`Object`了：

```java
public class Node {
    private Comparable data;
    private Node next;
    public Node(Comparable data, Node next) {
        this.data = data;
        this.next = next;
    }
    public Comparable getData() { return data; }
    // ...
}
```

上面的概念或许还是比较好理解，但其实泛型擦除带来的问题远远不止这些，接下来我们系统地来看一下类型擦除所带来的一些问题，有些问题在`C++`的泛型中可能不会遇见，但是在`Java`中却需要格外小心。

#### 问题一

在`Java`中不允许创建泛型数组，类似下面这样的做法编译器会报错：

```java
List<Integer>[] arrayOfLists = new List<Integer>[2];  // compile-time error
```

为什么编译器不支持上面这样的做法呢？继续使用逆向思维，我们站在编译器的角度来考虑这个问题。我们先来看一下下面这个例子：

```java
Object[] strings = new String[2];
strings[0] = "hi";   // OK
strings[1] = 100;    // An ArrayStoreException is thrown.
```

对于上面这段代码还是很好理解，字符串数组不能存放整型元素，而且这样的错误往往要等到代码运行的时候才能发现，编译器是无法识别的。接下来我们再来看一下假设`Java`支持泛型数组的创建会出现什么后果：

```java
Object[] stringLists = new List<String>[];  // compiler error, but pretend it's allowed
stringLists[0] = new ArrayList<String>();   // OK
// An ArrayStoreException should be thrown, but the runtime can't detect it.
stringLists[1] = new ArrayList<Integer>();
```

假设我们支持泛型数组的创建，由于运行时期类型信息已经被擦除，`JVM`实际上根本就不知道`new ArrayList<String>()`和`new ArrayList<Integer>()`的区别。类似这样的错误假如出现才实际的应用场景中，将非常难以察觉。

如果你对上面这一点还抱有怀疑的话，可以尝试运行下面这段代码：

```java
public class ErasedTypeEquivalence {
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2); // true
    }
}
```

#### 问题二

继续复用我们上面的`Node`的类，对于泛型代码，`Java`编译器实际上还会偷偷帮我们实现一个`Bridge method`。

```java
public class Node<T> {
    public T data;
    public Node(T data) { this.data = data; }
    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}
public class MyNode extends Node<Integer> {
    public MyNode(Integer data) { super(data); }
    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

看完上面的分析之后，你可能会认为在类型擦除后，编译器会将`Node`和`MyNode`变成下面这样：

```java
public class Node {
    public Object data;
    public Node(Object data) { this.data = data; }
    public void setData(Object data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}
public class MyNode extends Node {
    public MyNode(Integer data) { super(data); }
    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

实际上不是这样的，我们先来看一下下面这段代码，这段代码运行的时候会抛出`ClassCastException`异常，提示`String`无法转换成`Integer`：

```java
MyNode mn = new MyNode(5);
Node n = mn; // A raw type - compiler throws an unchecked warning
n.setData("Hello"); // Causes a ClassCastException to be thrown.
// Integer x = mn.data;
```

如果按照我们上面生成的代码，运行到第3行的时候不应该报错(注意我注释掉了第4行)，因为`MyNode`中不存在`setData(String data)`方法，所以只能调用父类`Node`的`setData(Object data)`方法，既然这样上面的第3行代码不应该报错，因为`String`当然可以转换成`Object`了，那`ClassCastException`到底是怎么抛出的？

实际上`Java`编译器对上面代码自动还做了一个处理：

```java
class MyNode extends Node {
    // Bridge method generated by the compiler
    public void setData(Object data) {
        setData((Integer) data);
    }
    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
    // ...
}
```

这也就是为什么上面会报错的原因了，`setData((Integer) data);`的时候`String`无法转换成`Integer`。所以上面第2行编译器提示`unchecked warning`的时候，我们不能选择忽略，不然要等到运行期间才能发现异常。如果我们一开始加上`Node<Integer> n = mn`就好了，这样编译器就可以提前帮我们发现错误。

#### 问题三

正如我们上面提到的，`Java`泛型很大程度上只能提供静态类型检查，然后类型的信息就会被擦除，所以像下面这样利用类型参数创建实例的做法编译器不会通过：

```java
public static <E> void append(List<E> list) {
    E elem = new E();  // compile-time error
    list.add(elem);
}
```

但是如果某些场景我们想要需要利用类型参数创建实例，我们应该怎么做呢？可以利用反射解决这个问题：

```java
public static <E> void append(List<E> list, Class<E> cls) throws Exception {
    E elem = cls.newInstance();   // OK
    list.add(elem);
}
```

我们可以像下面这样调用：

```java
List<String> ls = new ArrayList<>();
append(ls, String.class);
```

实际上对于上面这个问题，还可以采用`Factory`和`Template`两种设计模式解决，感兴趣的朋友不妨去看一下`Thinking in Java`中第15章中关于`Creating instance of types`（英文版第664页）的讲解，这里我们就不深入了。

#### 问题四

我们无法对泛型代码直接使用`instanceof`关键字，因为`Java`编译器在生成代码的时候会擦除所有相关泛型的类型信息，正如我们上面验证过的JVM在运行时期无法识别出`ArrayList<Integer>`和`ArrayList<String>`的之间的区别：

```java
public static <E> void rtti(List<E> list) {
    if (list instanceof ArrayList<Integer>) {  // compile-time error
        // ...
    }
}
=> { ArrayList<Integer>, ArrayList<String>, LinkedList<Character>, ... }
```

和上面一样，我们可以使用通配符重新设置`bounds`来解决这个问题：

```java
public static void rtti(List<?> list) {
    if (list instanceof ArrayList<?>) {  // OK; instanceof requires a reifiable type
        // ...
    }
}
```

#### 工厂模式

接下来我们利用泛型来简单的实现一下工厂模式，首先我们先声明一个接口`Factory`：

```java
package typeinfo.factory;
public interface Factory<T> {
    T create();
}
```

接下来我们来创建几个实体类`FuelFilter`和`AirFilter`以及`FanBelt`和`GeneratorBelt`。

```java
class Filter extends Part {}

class FuelFilter extends Filter {
    public static class Factory implements typeinfo.factory.Factory<FuelFilter> {
        public FuelFilter create() {
            return new FuelFilter();
        }
    }
}

class AirFilter extends Filter {
    public static class Factory implements typeinfo.factory.Factory<AirFilter> {
        public AirFilter create() {
            return new AirFilter();
        }
    }
}
```

```java
class Belt extends Part {}

class FanBelt extends Belt {
    public static class Factory implements typeinfo.factory.Factory<FanBelt> {
        public FanBelt create() {
            return new FanBelt();
        }
    }
}

class GeneratorBelt extends Belt {
    public static class Factory implements typeinfo.factory.Factory<GeneratorBelt> {
        public GeneratorBelt create() {
            return new GeneratorBelt();
        }
    }
}
```

`Part`类的实现如下，注意我们上面的实体类都是`Part`类的间接子类。在`Part`类我们注册了我们上面的声明的实体类。所以以后我们如果要创建相关的实体类的话，只需要在调用`Part`类的相关方法了。这么做的一个好处就是如果的业务中出现了`CabinAirFilter`或者`PowerSteeringBelt`的话，我们不需要修改太多的代码，只需要在`Part`类中将它们注册即可。

```java
class Part {
    static List<Factory<? extends Part>> partFactories =
        new ArrayList<Factory<? extends Part>>();
    static {
        partFactories.add(new FuelFilter.Factory());
        partFactories.add(new AirFilter.Factory());
        partFactories.add(new FanBelt.Factory());
        partFactories.add(new PowerSteeringBelt.Factory());
    }
    private static Random rand = new Random(47);
    public static Part createRandom() {
        int n = rand.nextInt(partFactories.size());
        return partFactories.get(n).create();
    }
    public String toString() {
        return getClass().getSimpleName();
    }
}
```

最后我们来测试一下：

```java
public class RegisteredFactories {
    public static void main(String[] args) {
        for (int i = 0; i < 10; i++) {
            System.out.println(Part.createRandom());
        }
    }
}
```