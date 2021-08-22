## 什么是注解

注解（Annotation）是Java提供的设置程序中元素的关联信息和元数据（MetaData）的方法，它是一个接口，程序可以通过反射获取指定程序中元素的注解对象，然后通过该注解对象获取注解中的元数据信息

## 标准元注解：@Target、@Retention、@Documented、@Inherited
元注解（Meta-Annotation）负责注解其他注解。在Java中定义了4个标准的元注解类型@Target、@Retention、@Documented、@Inherited，用于定义不同类型的注解。

- @Target:说明了注解所修饰的对象范围，它用于明确其修饰的目标。

注解可被用于packages、types（类、接口、枚举、注解类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（循环变量、catch参数等），详细如下：

名称 | 修饰目标
---|---
ElementType.TYPE | 用于描述类、接口注解、枚举类型
ElementType.FIELD | 用于描述域
ElementType.METHOD | 用于描述方法
ElementType.PARAMETER|用于描述参数
ElementType.CONSTRUCTOR|用于描述构造器
ElementType.LOCAL_VARIABLE|用于描述局部变量
ElementType.ANNOTATION_TYPE|用于描述一个注解
ElementType.PACKAGE|用于描述包
ElementType.TYPE_PARAMETER|用于描述普通变量（JDK8）
ElementType.TYPE_USE|用于描述类型使用，能标注任何类型（JDK8）
ElementType.MODULE| 用于描述模块（JDK9）

- `@Retention`: 定义了该注解被保留的级别，为注解声明生命周期，它表示注解存在阶段是保留在源码（编译期），字节码（类加载）或者运行期（JVM中运行），有以下3种类型：

    - @Retention(RetentionPolicy.SOURCE)：注解仅存在于源码中，在class字节码文件中不存在。
    - @Retention(RetentionPolicy.CLASS)：默认的保留策略，注解会在源码、class字节码文件中存在，但运行时无法获得。
    - @Retention(RetentionPolicy.RUNTIME)：在运行时有效，注解会在源码、class字节码文件中存在，在运行时可以通过反射获取到。




- `@Documented`: 表明这个注解应该被javadoc工具记录，因此可以被javadoc类的工具文档化

- `@Inherited`: 被@Inherited注解的父类，其子类也会继承父类的注解

## Java常用的注解

- `@SuppressWarnings`：描述所标注内容产生的警告，编译器会对这些警告保持静默


- `@Deprecated`：描述所标注内容，是过时的，不再被建议使用

- `@Override`：描述当前方法是一个重写的方法，在编译阶段对方法进行检查

- `@FunctionalInterface`：Java 8 开始支持@FunctionalInterface，标识一个匿名函数或函数式接口

- `@Repeatable`：被这个元注解修饰的注解可以同时作用一个对象多次，但是每次作用注解又可以代表不同的含义

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Repeatable {
    /**
     * Indicates the <em>containing annotation type</em> for the
     * repeatable annotation type.
     * @return the containing annotation type
     */
    Class<? extends Annotation> value();
}


```

## 通过反射来操作注解的属性
> **[!NOTE]**
> 
> 这里所讨论的获取并动态修改注解属性值，建立在 @Retention(RetentionPolicy.RUNTIM) 的基础上，这种情况下才能在运行时(runtime)通过反射机制进行操作


```java
public class TestAnnotaionChange {

    public static void main(String... args) throws NoSuchFieldException, IllegalAccessException {
        MyAnnotation annotation = AnnotatedChangeTestPojo.class.getDeclaredField("name").getAnnotation(MyAnnotation.class);
        System.out.println(annotation.value());  // 输出：张三
        changeAnnotationFieldValue(AnnotatedChangeTestPojo.class, "name", "value", "LuckingJack");
        System.out.println(annotation.value()); // 输出：LuckingJack
    }

    /**
     * 变更指定的实体类中的成员变量上，MyAnnotation注解的属性值
     * <p>使用了JDK动态代理</p>
     *
     * @param annotatedPojo               打上{@link MyAnnotation}注解的实体类
     * @param annotatedPojoFieldName      打上{@link MyAnnotation}注解的实体类的属性名
     * @param declareAnnotationFieldName  {@link MyAnnotation}注解的属性名
     * @param declareAnnotationFieldValue {@link MyAnnotation}注解的属性值变更为
     * @author LuckinJack
     * @date 2021/3/2 14:31
     */
    public static void changeAnnotationFieldValue(@NotNull Class<?> annotatedPojo,
                                                  @NotNull String annotatedPojoFieldName,
                                                  String declareAnnotationFieldName,
                                                  Object declareAnnotationFieldValue) throws NoSuchFieldException, IllegalAccessException {
        // 获取类AnnotatedChangeTestPojo的成员变量上标记的注解MyAnnotation的实例
        MyAnnotation annotation = annotatedPojo.getDeclaredField(annotatedPojoFieldName).getAnnotation(MyAnnotation.class);
        // 获取 annotation 这个代理实例所持有的 InvocationHandler
        InvocationHandler invocationHandler = Proxy.getInvocationHandler(annotation);
        //获取私有 memberValues 属性
        Field field = invocationHandler.getClass().getDeclaredField("memberValues");
        // 因为"memberValues"这个字段是 private final 修饰，所以要打开权限
        field.setAccessible(true);
        // 获取实例的属性map
        Map<String, Object> memberValues = (Map<String, Object>) field.get(invocationHandler);
        // 修改属性值
        memberValues.put(declareAnnotationFieldName, declareAnnotationFieldValue);
    }

    @Data
    public static class AnnotatedChangeTestPojo {

        @MyAnnotation(value = "张三")
        private String name;
    }

    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    public @interface MyAnnotation {

        String value() default "UNKNOWN";
    }
}


```
