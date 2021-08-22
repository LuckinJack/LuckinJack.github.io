## 什么是泛型
"泛型"的字面意思就是广泛的类型。
类、接口和方法代码可以应用于非常广泛的类型，代码与它们能够操作的数据类型不再绑定在一起，同一套代码可以用于多种数据类型。


## 泛型标记与泛型限定

### 泛型标记与通配符

`<?>`、`<T>`、`<E>`、`<INPUT>`、`<OUTPUT>`

## 泛型类

T表示类型参数，泛型就是类型参数化，处理的数据类型不是固定的，而是可以作为参数传入

## 泛型方法

除了泛型类，方法也可以是泛型的，而且，一个方法是不是泛型的，与它所在的类是不是泛型没有什么关系。

## 泛型接口


### 泛型限定

1．对泛型上限的限定：`<? extends T>`

在Java中使用通配符 *?* 和 *extends* 关键字指定泛型的上限，具体用法为`<? extends T>`，它表示该通配符所代表的类型是T类的子类或者接口T的子接口。

2．对泛型下限的限定：`<? super T>`

在Java中使用通配符 *?* 和 *super* 关键字指定泛型的下限，具体用法为`<? super T>`，它表示该通配符所代表的类型是T类型的父类或者父接口。

> **[!NOTE]**
>
> - `<? super T>` 用于灵活写入或比较，使得对象可以写入父类型的容器，使得父类型的比较方法可以应用于子类对象，它不能被类型参数形式替代
>
> - `<? > 和 <? extends T>` 用于灵活读取，使得方法可以读取 `T` 或 `T` 的任意子类型的容器对象，它们可以用类型参数的形式替代，但通配符形式更为简洁



## 理解Java泛型原理

我们先从下面这个例字入手，感受下泛型：
```java
public class Test1 {

    public static void main(String[] args) {
        PairObject pairObject = new PairObject("name", "LuckinJack");
        String pairObjectKey = (String) pairObject.getFirst();
        String pairObjectValue = (String) pairObject.getSecond();

        PairGenerics pairGenerics = new PairGenerics("name", "LuckinJack");
        String pairGenericsKey = (String) pairGenerics.getFirst();
        String pairGenericsValue = (String) pairGenerics.getSecond();
    }
}

public class PairObject {
    Object first;
    Object second;

    public PairObject(Object first, Object second) {
        this.first = first;
        this.second = second;
    }

    public Object getFirst() {
        return first;
    }

    public Object getSecond() {
        return second;
    }
}

public class PairGenerics<K, V> {
    K first;
    V second;

    public PairGenerics(K first, V second) {
        this.first = first;
        this.second = second;
    }

    public K getFirst() {
        return first;
    }

    public V getSecond() {
        return second;
    }
}
```

可以注意到，`PairObject`和`PairGenerics`的使用方式，居然是一样的，为什么呢？

### Java泛型的特点

其实对于泛型类，Java编译器会将泛型代码转换为普通的非泛型代码，就像上面展示的那样，泛型代码中，JVM将类型参数`T`擦除，替换为了`Object`，并插入必要的强制类型转换。

> **[!NOTE]**
>
> 1. 在不使用泛型的情况下，我们可以通过引用`Object`类型来实现参数的任意化，因为在Java中`Object`类是所有类的父类，但在具体使用时需要进行强制类型转换。
>
> 2. 强制类型转换要求开发者必须明确知道实际参数的引用类型，不然可能引起前置类型转换错误，程序在编译期无法识别这种错误，只有在程序运行期间抛异常时我们才能发现原来类型转换错了。
>
> 3. **使用泛型的好处是在编译期就能够检查类型是否安全、正确，并自动且隐式的进行强制性类型转换**。这样不仅可以复用代码，降低耦合，还提高了代码的安全性和重用性，通过下面的代码就能体会到泛型的这种特点：
> ```java
> public class Test2 {
> 
>     public static void main(String[] args) {
> 		// 不小心把类型搞错了，运行时程序才会抛出类型转换异常ClassCastException
>         PairObject pairObject = new PairObject("name", "LuckinJack");
>         Integer pairObjectKey = (Integer) pairObject.getFirst();
>         String pairObjectValue = (String) pairObject.getSecond();
> 
> 		// 不小心把类型搞错了，IDE和编译器都会有错误提示，更安全
> 		// 而且还省去了烦琐的强制类型转换
>         PairGenerics pairGenerics = new PairGenerics("name", "LuckinJack");
>         Integer pairGenericsKey =  pairGenerics.getFirst();
>         String pairGenericsValue = pairGenerics.getSecond();
>     }
> }
> ```
> 


### Java泛型的实现机制：类型擦除

**Java泛型其实不是真正的泛型，它是通过擦除实现的**。

> **[!WARNING]**
>
> **1. 什么是Java的泛型擦除？**
>
>  Java的泛型主要用于编译阶段，因为在编码阶段采用泛型时加上的类型参数，会被编译器在编译后去掉，仅保留其原始类型，这个过程就被称为**类型擦除**。
>
>   - 如果指定了类型参数的上界，则那么原始类型就用第一个上界的类型变量类替换(比如 `class Pair<T extends Comparable>` 的原始类型就是`Comparable`)；
>   - 如果未指定类型参数的上界，原始类型就是`Object`。
>
> **2. 泛型类并没有自己独有的 Class 类对象，运行时无法获得泛型的真实类型信息**
>
> 比如类 `public interface List<E> extends Collection<E>`
>
> 编码时定义的无论是 `List<Integer>` 还是 `List<String>` ，在经过 JVM 编译之后编译后都会统一为 `List` ，并不会因为传入参数的不同而存在 `List<String>.class` 或是 `List<Integer>.class`，泛型附加的类型信息对 JVM 来说是不可见的。
>
> ```java
> public class Test3 {
> 
>     public static void main(String[] args) {
>         ArrayList<String> list1 = new ArrayList<>();
>         list1.add("abc");
>         ArrayList<Integer> list2 = new ArrayList<>();
>         list2.add(123);
> 
>         System.out.println(list1.getClass() == list2.getClass());   // 输出 true
>         System.out.println(ArrayList.class == list1.getClass());    // 输出 true
>         System.out.println(ArrayList.class == list2.getClass());    // 输出 true
>     }
> 
> }
> ```
> **3. 静态变量是被泛型类的所有实例所共享的。**
>
> 对于声明为 `MyClass<T>` 的类，访问其中的静态变量的方法仍然是 MyClass.myStaticVar。不管是通过 new MyClass<String> 还是 new MyClass<Integer> 创建的对象，都是共享一个静态变量。

Java的泛型为什么要设计得这么局限呢？泛型是 Java 5 以后才支持的，Java 5 以前的设计不支持真正的泛型，算是一个历史遗留问题，类型擦除是为了兼容性而不得已的一个选择。



# 参考

- [Java的泛型](https://mp.weixin.qq.com/s/lsh3bSGrNUHhemS4rY1-xQ)