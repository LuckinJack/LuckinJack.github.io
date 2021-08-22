## 什么是反射

动态语言指程序在运行时可以改变其结构的语言，比如新的属性或方法的添加、删除等结构上的变化。JavaScript、Ruby、Python等都属于动态语言；C、C++不属于动态语言。从反射的角度来说，Java属于半动态语言。

反射机制，指在程序运行过程中，对任意一个类都能获取其所有属性和方法，并且对任意一个对象都能调用其任意一个方法。这种动态获取类和对象的信息，以及动态调用对象的方法的功能被称为Java语言的反射机制。

程序在编译期间无法预知该对象和类的真实信息，只能通过运行时信息来发现该对象和类的真实信息，而其真实信息（对象的属性和方法）通常通过反射机制来获取，这便是Java语言中反射机制的核心功能。

## 通过反射获取类的Class对象、方法、属性

Java的反射API主要用于在运行过程中动态生成类、接口或对象等信息，其常用API如下。
- *Class* 类：用于获取类的属性、方法等信息。
- *Field* 类：表示类的成员变量，用于获取和设置类中的属性值。
- *Method* 类：表示类的方法，用于获取方法的描述信息或者执行某个方法。
- *Constructor* 类：表示类的构造方法。

```java
//1: 获取Person类的Class对象
Class clazz = Class.forName("java.reflect.Person");
/2: 获取Person类的所有方法的信息
Method[] method = clazz.getDeclaredMethods(); 
for (Method m: method) {
	System.out.println(m.toString()));
}
//3: 获取Person类的所有成员的属性信息 
Field[] field = clazz.getDeclaredFields(); 
for(Field f: field){
	System.out.println(f.toString());
)
//4: 获取Person类的所有构造方法的信息 
Constructor[] constructor = clazz.getDeclaredConstructors();
for(Constructor c: constructor){
	System.out.println(c.toString());
}
```

## 通过反射创建类对象

创建对象的两种方式如下：
1. 使用Class对象的newInstance方法创建该Class对象对应类的实例，这种方法要求该Class对象对应的类有默认的空构造器。
2. 先使用Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance方法创建Class对象对应类的实例，通过这种方法可以选定构造方法创建实例。

```java
// 方法1：
//step1∶ 获取Person类的Class对象
Class clazz = Class.forName("java.reflect.Person");
//step2∶ 使用newInstane方法创建对象 
Person p1 = (Person) clazz.newInstance());

// 方法2：
//step1∶ 获取构造方法并创建对象
Constructor c = clazz.getDeclaredConstructor(String.class, String.class, int.class);
//step2∶ 根据构造方法创建对象并设置属性
Person p2 = (Person) c.newInstance("LuckinJack"，"男"，25);
```

## 通过反射动态调用类方法

我们通过 `method.invoke()` 方法可以实现动态调用，以动态传入参数及将方法参数化。具体过程为：获取对象的 *Method* ，并调用 *Method* 的 invoke 方法，实现方式如下:

```java
//step1∶ 获取 Person 类（java.reflect.Person）的Class对象
Class clz = Class.forName("java.reflect.Person")
//step2∶ 获取Class对象中的setName方法
Method method = clz.getMethod("setName", String.class)
//step3∶ 获取Constructor对象
Constructor constructor = clz.getConstructor());
//step4∶ 根据Constructor定义对象
Object object = constructor.newInstance();
//step5∶ 调用method的invoke方法，这里的method表示setName方法
//因此，相当于动态调用object对象的setName方法并传入参数 “LuckinJack”
method.invoke(object, "LuckinJack");
```
