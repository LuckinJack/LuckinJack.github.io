
# Java工具类

> 软件架构中,有两个软件重用的概念: **水平式重用** 与 **垂直式重用** 。
> 
> **水平式重用**：是指可以在不同应用领域中使用的软件元素，简单理解就是业务无关性，可以在任意业务场景中使用
>
> **垂直式重用**：是指在一类或具有较多公共性的应用领域之间进行软部件重用，简单理解就是可以在特定的业务领域或业务场景中使用
>
> Util类就属于上面所说的水平式重用。Util类更多是对JDK提供的类进行封装，或者是某一技术框架自己提供的对框架内部其他类的使用封装，但是这类一般都具有业务、领域的无关性，在任何业务、领域下均可使用。所以，既然是工具类，我们一定要保证其水平式重用这一特性。

# 命名方式

行业中工具类的命名普遍还是以Util或者Utils结尾，使开发人员能见名知意。例如 `FileUtil`、`DateUtils`、`CollectionUtils`。

# 内部结构

一般来讲，工具类要满足这些条件：工具类应该具有业务无关性，且定义的工具方法应全是static方法，直接用类名调用就可以了，而且工具类不应该被实例化。

> ※ 如果是使用了单例模式的工具类，那么应该使用static方法初始化某个单例

下面列举了两种写法，但更推荐用第二种写法：

**1. 类为abstract，类的成员方法为static静态方法**

源码示例：`org.springframework.util.CollectionUtils`

```java
public abstract class CollectionUtils {

    ···
	public static boolean isEmpty(@Nullable Collection<?> collection) {
		return (collection == null || collection.isEmpty());
	}

	public static boolean isEmpty(@Nullable Map<?, ?> map) {
		return (map == null || map.isEmpty());
	}
	
    ···
}
```
虽然用abstract来修饰工具类也能达到类似的效果，但abstract的作用会对外声明“这个类无法实例化，是用来被继承的”这一歧义，所以我们不推荐这种写法。

**2. 类为`final`，显式的将类构造方法私有化，类的成员方法为static静态方法**
    
源码示例 `org.apache.tools.ant.util.DateUtils`

```java
public final class DateUtils {

    ···
    
    private DateUtils() {
        throw new UnsupportedOperationException("This class should not be instantiated.");
    }

    public static String format(long date, String pattern) {
        return format(new Date(date), pattern);
    }

    public static String format(Date date, String pattern) {
        DateFormat df = createDateFormat(pattern);
        return df.format(date);
    }
    
    ···
    
}
```
上面的这个工具类，构造方法中的异常是后面追加上去的。因为DateUtils这个工具类依然能通过反射来生成实例对象 `Class.forName("DateUtils").newInstance()` , 在私有构造方法中追加异常则避免了这种情况，提升了工具类的严谨性。