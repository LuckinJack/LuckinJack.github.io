# 默认方法



# Optional

*Optional* 类可以保存泛型类型为 *T* 的值，或者仅仅保存 *null* 。它可以与流或其它返回 *Optional* 的方法结合，代替传统的 *if-else* 进行 *null* 值判断，优雅的解决`NullPointException` 问题，并构建流畅的API。

## 定义

- `of()` 和 `ofNullable()` 方法创建包含值的 *Optional*
  - `of()`：明确对象不为*null* 时使用

  - `ofNullable()`：对象即可能是 *null* 也可能是非 *null*  时使用

- `empty()` 返回一个空 *Optional* 对象


- `orElse()` 和 `orElseGet()` 返回默认值

  - `orElse()`： 判断value不为 *null* 时返回前面的值，否则返回传递给它的参数值

  - `orElseGet()`：判断value不为 *null* 时返回前面的值，否则执行参数传入的函数式接口，并返回结果

- `orElseThrow()` 判断value为 *null* 时返回前面的值，否则抛出一个指定的异常

- `map()` 和 `flatMap()`方法进行值转换


  - `map()`：入参为 `Function<? super T, ? extends U>`
  - `flatMap()`：入参为`Function<? super T, Optional<U>>`

- `isPresent()`和`ifPresent()` 判断value是否为 *null*


  - `isPresent()`：判断value是否为 *null*，对应返回 true/false
  - `ifPresent()`：value值不为 *null* 时，做一些操作

- `filter()` 接受一个 *Predicate* 来对 *Optional* 中的值进行过滤，满足条件返回原来的 *Optional* ，否则返回 *Optional.empty*

## 写法示例

**写法1**

```java
// 旧写法
if(user != null){
	System.out.println(user);
}

// JDK8+的新写法
Optional.ofNullable(user).ifPresent(u -> {
	System.out.println(u);
});
```

**写法2**
```java
// 旧写法
public String getCity(User user)  throws Exception {
        if(user != null){
            if(user.getAddress() != null){
                Address address = user.getAddress();
                if(address.getCity() != null){
                    return address.getCity();
                }
            }
        }
        throw new Exception("user实例及其成员属性不能为null"); 
    }

// JDK8+的新写法
public String getCity(User user) throws Exception{
    return Optional.ofNullable(user)
                   .map(User::getAddress)
                   .map(Address::getCity)
                   .orElseThrow(() -> new Exception("user实例及其成员属性不能为null"));
}

```

**写法3**

```java
// 旧写法
public User getUser(User user) throws Exception{
    if(user!=null){
        String name = user.getName();
        if("luckinjack".equalsIgnoreCase(name)){
            return user;
        }
    }else{
        user = new User();
        user.setName("anonymous");
        return user;
    }
}

// JDK8+的新写法
public User getUser(User user) {
    return Optional.ofNullable(user)
                   .filter(u ->"luckinjack".equalsIgnoreCase(u.getName()))
                   .orElseGet(() -> {
                        User defaultUser = new User();
                        defaultUser.setName("anonymous");
                        return defaultUser;
                   });
}
```

**写法4**

```java
// 旧写法
List<OrgBase> orgList = orgDao.selectOrgList();
String temp = "";
if (CollectionUtils.isNotEmpty(orgList) && orgList.get(0) != null) {
    OrgBase orgBase = orgList.get(0);
    if (orgBase.getOrgName() != null) {
        temp = orgBase.getOrgName();
    }
}


// JDK8+的新写法
List<OrgBase> orgList = orgDao.selectOrgList();
String temp = Optional.ofNullable(orgList)
					.flatMap(list -> list.stream().findFirst())
                    .map(OrgBase::getOrgName)
                    .orElse("");
```



# Lambda

## 方法引用

若Lambda体中的内容有方法已经实现了，我们可以使用 **方法引用**，可以理解为方法引用是lambda表达式的另外一种表达形式。**方法引用** 主要有三种语法格式，都带有标志性的双冒号。需要注意的是，被引用的方法的参数和返回值必须和要实现的抽象方法的参数和返回值一致。

1. 对象::实例方法名
2. 类::静态方法名
3. 类::实例方法名


## 函数接口

`java.lang.FunctionalInterface`



### Comparator

以下面的实体为例子

```java
public class Test5 {

    public static void main(String[] args) {
        Employee e1 = new Employee(9922001L, "Ace", 25, 7000);
        Employee e2 = new Employee(5924001L, "Bob", 22, 5000);
        Employee e3 = new Employee(3924401L, "Calvin", 22, 6000);
        Employee e4 = new Employee(3924402L, "David", 22, 4000);
        Employee e5 = new Employee(3924403L, "Eve", 26, 15000);

        List<Employee> employees = new ArrayList<>();
        employees.add(e1);
        employees.add(e2);
        employees.add(e3);
        employees.add(e4);
        employees.add(e5);
    }


    @Data
    @AllArgsConstructor
    public static class Employee {
        private Long id;
        private String name;
        private Integer age;
        private Integer salary;
    }

}
```



❑  使用 *Comparator* 外部比较器 进行排序

```
employees.sort((o1, o2) -> o1.getName().compareTo(o2.getName()));

Collections.sort(employees, (o1, o2) -> o1.getName().compareTo(o2.getName()));
```

❑  使用  *Comparator* 内部比较器 进行排序

- `Comparator.comparing`

  ```java
  // 通过lambda 表达式完成比较
  employees.sort(Comparator.comparing(e -> e.getName()));
  
  // 用方法引用 Employee::getName 代替 lambda表达式
  employees.sort(Comparator.comparing(Employee::getName));
  
  // 自定义比较器
  employees.sort(Comparator.comparing(Employee::getName, (s1, s2) -> {
                      return s2.compareTo(s1);
                  }));
  ```

- `Comparator.reversed`

  ```java
  // 使用
  employees.sort(Comparator.comparing(Employee::getName).reversed());
  
  // 使用 Collections.reverseOrder()
  employees.sort(Comparator.comparing(Employee::getName, Comparator.reverseOrder()));
  ```

- `Comparator.nullsFirst`

  ```java
  // null元素排在集合的最前面
  employees.sort(Comparator.nullsFirst(Comparator.comparing(Employee::getName)));
  
  // null元素排在集合的最后面
  employees.sort(Comparator.nullsLast(Comparator.comparing(Employee::getName)));
  ```

- `Comparator.thenComparing`

  ```java
  // 在第一个键的基础之上再根据下一个键进行排序
  employees.sort(Comparator.comparing(Employee::getAge).thenComparing(Employee::getSalary));
  
  // thenComparing支持无限嵌套和自定义比较器    
  employees.sort(Comparator.comparing(Employee::getAge).thenComparing(Employee::getSalary, (s1, s2) -> {
              return s2.compareTo(s1);
          }));
  ```



## 流处理方法

### map



### max、min、count



### reduce

*reduce* 操作可以实现从一组值中生成一个值。在上述例子中用到的 *count*、*min* 和 *max* 方法，因为常用而被纳入标准库中。事实上，这些方法都是 *reduce* 操作



### filter、sorted



### collect

`java.util.stream.Collectors`



#### 转换为其他集合

- `Collectors.toList()`、`Collectors.toSet()`
  - 可以用指定的实现类 `ArrayList`、`LinkedList`、`CopyOnWriteArrayList`去收集

```java
// 将所有元素转换为 List，有顺序
List<String> list = Arrays.asList("Java", "Python", "C", "Go", "Java");  
List<String> listResult = list.stream().collect(Collectors.toList());
List<String> linkedListResult = list.stream().collect(Collectors.toCollection(LinkedList::new));
List<String> copyOnWriteArrayListResult = list.stream().collect(Collectors.toCollection(CopyOnWriteArrayList::new));
list.forEach(System.out::println);
listResult.forEach(System.out::println);
linkedListResult.forEach(System.out::println);
copyOnWriteArrayListResult.forEach(System.out::println);

// 将所有元素转换为 Set，删除重复项，无顺序
Stream<String> language = Stream.of("Java", "Python", "C", "Go", "Java");
Set<String> setResult = language.collect(Collectors.toSet());
setResult.forEach(System.out::println);
```

- `Collectors.toMap()`

  - 允许key为null，但不允许value的值为null
  - 可以用指定的Map实现类 `HashMap`、`LinkedHashMap` 等去收集

```java
// 不允许key值重复，否则会报错 `IllegalStateException: Duplicate key`
userList.stream().collect(Collectors.toMap(User::getKey, User::getValue))

// 允许key值重复，如果key值重复，就用后面的值覆盖（即取最新的值）
userList.stream().collect(Collectors.toMap(User::getKey, User::getValue, (key1,key2) -> key2))
```

> **[!TIP]**
> map的value值为null的时，会导致NullPointerException异常
>
> 解决方法：
> 1. 设值时加判断，如果是null，改为一个特定值
> ```java
> Map<String, Integer> collect = userStream.collect(Collectors.toMap(User::getKey, user -> user.value == null ? 0 : user.value));
> ```
>
> 2. 使用`collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)`来构建，允许value的值为null
> ```java
> Map<String, Integer> map = userStream.collect(HashMap::new, (n, v) -> n.put(v.getKey(), v.getValue()), HashMap::putAll);
> ```
>
> 3. 使用`Optional<T>`对值进行包装
> ```java
> Map<String, Optional<Integer>> map = userStream.collect(Collectors.toMap(User::getKey, user -> Optional.ofNullable(user.getValue())));
> Integer age = map.get("a").orElse(0);
> ```
>

#### 转换为其他值





#### 数据分块

- `Collectors.partitioningBy`

```java
List<String> list = Arrays.asList("Java", "Python", "C", "Go", "Java");
// 使用Predicate对象的判断结果（true 或 false），分类成一个key为 true 和 false 的Map
Map<Boolean, List<String>> result = list.stream().collect(Collectors.partitioningBy(s -> s.length() > 2));
System.out.println(result);
```

#### 数据分组

- `Collectors.groupingBy`

```java
public class Test {
    public static void main(String[] args) {
		Language language1 = new Language("front", "Vue");
        Language language2 = new Language("front", "React");
        Language language3 = new Language("back", "Java");
        Language language4 = new Language("back", "Python");
		
		// 1. 按照属性分组，用原有属性收集
		Map<String, List<Language>> groupedMap1 = languageList.stream().collect(Collectors.groupingBy(Language::getType));
		
		// 2. 计数等统计类操作
		Map<String, Long> groupedMap2 = languageList.stream().collect(Collectors.groupingBy(Language::getType, Collectors.counting()));
		
		// 3. 按照不同条件分组
		Map<String, List<Language>> groupedMap3 = languageList.stream().collect(Collectors.groupingBy(item ->{
        if ("back".equals(item.getType())) {
            return "back-end";
        } else {
            return "front-end";
            }
        }));
		
		// 4. 把收集器的结果转换为另一种类型
		Map<String, Set<String>> groupedMap4 = languageList.stream().collect(Collectors.groupingBy(Language::getType, Collectors.mapping(Language::getName, Collectors.toSet())));
		
		// {back=[Test.Language(type=back, name=Java), Test.Language(type=back, name=Python), front=[Test.Language(type=front, name=Vue), Test.Language(type=front, name=React)]}
		System.out.println(groupedMap1);
		
		// {back=2, front=2}
		System.out.println(groupedMap2);
		
		// {back-end=[Test.Language(type=back, name=Java), Test.Language(type=back, name=Python), front-end=[Test.Language(type=front, name=Vue), Test.Language(type=front, name=React)]}
		System.out.println(groupedMap3);
		
		// {back=[Java, Python], front=[Vue, React]}
		System.out.println(groupedMap4);
		
	@Data
    public static class Language {
        private String type;
        private String name;

        public Language(String type, String name) {
            this.type = type;
            this.name = name;
        }
    }
}
```

#### 字符串处理

- `Collectors.joining`

```java
List<String> list = Arrays.asList("Java", "Python", "C", "Go","Java");
//直接将输出结果拼接
System.out.println(list.stream().collect(Collectors.joining()));
//每个输出结果之间加拼接符号“|”
System.out.println(list.stream().collect(Collectors.joining(" , ")));
//输出结果开始头为Start--，结尾为--End，中间用拼接符号“||”
System.out.println(list.stream().collect(Collectors.joining(" | ", " [ ", " ] ")));
```

#### 自定义收集器

自定义收集器的接口是`Collector<T, A, R>`，我们先看下源码。

```java
    /**
     *  1. Collector<T, A, R> 中的 T A R 分别是什么意思？
     *  T：第一个参数是待收集的元素类型
     *  A：第二个参数是累加器的类型
     *  R：第三个参数是最终结果类型
     *
     *  2.Collector<T, A, R> 的构造方法用到的成员变量 分别是什么意思？
     *  Supplier<A> supplier()：                     容器的提供者
     *  BiConsumer<A, T> accumulator()：             累加操作
     *  BinaryOperator<A> combiner()：               并发的情况将每个线程的中间容器A合并
     *  Function<A, R> finisher()：                  终止操作
     *  Set<Characteristics> characteristics()：     收集器特性（值为IDENTITY_FINISH时，会将A强转为R）											
     */
    public static<T, A, R> Collector<T, A, R> of(Supplier<A> supplier,
                                                 BiConsumer<A, T> accumulator,
                                                 BinaryOperator<A> combiner,
                                                 Function<A, R> finisher,
                                                 Characteristics... characteristics) {
        Objects.requireNonNull(supplier);
        Objects.requireNonNull(accumulator);
        Objects.requireNonNull(combiner);
        Objects.requireNonNull(finisher);
        Objects.requireNonNull(characteristics);
        Set<Characteristics> cs = Collectors.CH_NOID;
        if (characteristics.length > 0) {
            cs = EnumSet.noneOf(Characteristics.class);
            Collections.addAll(cs, characteristics);
            cs = Collections.unmodifiableSet(cs);
        }
        return new Collectors.CollectorImpl<>(supplier, accumulator, combiner, finisher, cs);
    }

  enum Characteristics {
        /**
         * 如果一个收集器被标记为 CONCURRENT 特性，那么 accumulator 方法可以被多线程并发的的调用，
         * 并且只使用一个容器A.如果收集器被标记为 CONCURRENT ，但是要操作的集合是有序的，那么
         * 最终得到的结果不能保证原来的顺序
         */
        CONCURRENT,

        /**
         * 适用于无序的集合
         */
        UNORDERED,

        /**
         * 如果收集器特性被设置 IDENTITY_FINISH ，那么会强制将中间容器A类型转换为结果类型R
         */
        IDENTITY_FINISH
    }
```

要实现自定义收集器，有两种方法

1. 实现接口`java.util.stream.Collector<T, A, R>`，重写并实现五个成员方法：`supplier()`、`accumulator()`、`combiner()`、`finisher()`、`characteristics()`。

```java
public class LanguageCollector implements Collector<Language, List<Language>, List<Language>> {
    
    @Override
    public Supplier<List<Language>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<Set<Language>, Language> accumulator() {
        return (set, language) -> {
            Predicate<Language> languagePredicate = s -> s.getType().equals(language.getType());
            boolean b = set.stream().noneMatch(languagePredicate);
            if (b) {
                language.setType("UNKNOW");
                set.add(language);
            }
            set.stream().filter(languagePredicate).forEach(s -> s.setName(language.getType() + s.getName()));
        };
    }

    @Override
    public BinaryOperator<List<Language>> combiner() {
        return null;
    }

    @Override
    public Function<List<Language>, List<Language>> finisher() {
        return Function.identity();
    }

    @Override
    public Set<Collector.Characteristics> characteristics() {
        return EnumSet.of(Characteristics.IDENTITY_FINISH);
    }
    
}
```

2. 直接调用 JDK 提供的方法`Collector.of()`，简洁明了。

```java
// 1. 对应传入五个成员方法的实现
Collector<Employee, List<Employee>, List<Employee>> employeeCollector = Collector.of(ArrayList::new, (left, right) -> {
            // 计算逻辑
        }, (left, right) -> null, Function.identity(), Collector.Characteristics.IDENTITY_FINISH);


// 2. 当 A 和 R 是同一种类型时，方法可以进行简化(少传两个参数)
Collector<Employee, List<Employee>, List<Employee>> employeeCollector2 = Collector.of(ArrayList::new, (left, right) -> {
            // 计算逻辑
        }, (left, right) -> null);
```









## 新版Date-Time API (JSR 310)

> **[!Warning]**
> **旧版Date-Time API的问题**：
>
> **1.** 非线程安全
> 
> java.util.Date 是线程不安全的，所有的日期类都可变，同时时间日期格式化工具SimpleDateFormat也是线程不安全的，这是Java日期类最大的问题之一。
> 
> **2.** 设计很差
> 
> Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类在java.text包中定义。
> java.util.Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
> java.util.Date中，获取的年份居然减去了1900，而且获取的月份是从0开始
> 
> **3.** 无法处理时区
> 
> 由于日期类并不提供国际化，因此Java引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在类似的所有问题。比如Calendar依旧是可变的，而且该类在语义上也完全不符合日期/时间的含义。

java8在java.time包下提供了很多新的API。

**新增内容：**

- Local(本地)：简化了日期时间的处理，没有时区的问题

- Zoned(时区)：通过制定的时区处理日期时间

- Instant：时间戳

- LocalDate：存储不包含时间的日期，比如2020-01-11

- LocalTime：存储不包含日期的时间，比如11:07:03.580

- LocalDateTime：日期和时间的组合，相当于 LocalDate + LocalDateTime

- ZonedDateTime：处理带时区的日期和时间

- Duration/Period：时间段/日期段，前者支持基于LocalDateTime初始化，后者支持基于LocalDate初始化
  - 两者的操作方式基本一致，父接口都是TemporalAmount。
  - 不同之处是：
    - Duration的单位是秒级和纳秒，使用于时间精度要求比较高的情况；
    - Period的单位是year, month 和day

```java
// 1.1 用Instant创建Duration
Instant start1 = Instant.parse("2021-05-11T17:14:30.00Z");
Instant end1 = Instant.parse("2021-06-01T18:00:30.12Z");
Duration duration1 = Duration.between(start1, end1);
System.out.println(duration1.getSeconds());     // 输出 1817160
System.out.println(duration1.getNano());        // 输出 120000000
System.out.println(duration1.getUnits());       // 输出 [Seconds, Nanos]

// 1.2 用LocalTime创建Duration
LocalTime start2 = LocalTime.of(9, 0, 25, 1314);
LocalTime end2 = LocalTime.of(18, 30, 27, 1516);
System.out.println(Duration.between(start2, end2).getSeconds());    // 输出 34202
System.out.println(Duration.between(start2, end2).getNano());       // 输出 202
System.out.println(Duration.between(start2, end2).getUnits());      // 输出 [Seconds, Nanos]

// 2 用LocalDate创建Period
LocalDate start3 = LocalDate.of(2020, 5, 30);
LocalDate end3 = LocalDate.of(2021, 6, 1);

System.out.println(Period.between(start3, end3).getYears());    // 输出 1
System.out.println(Period.between(start3, end3).getMonths());       // 输出 0
System.out.println(Period.between(start3, end3).getDays());      // 输出 2
System.out.println(Period.between(start3, end3).getUnits());      // 输出 [Years, Months, Days]
```

- `ChronoUnit`：基于Duration实现，用来表示时间单位，也提供了一些非常有用的between方法来计算两个时间的差值

```java
LocalDate startDate = LocalDate.of(2020, 5, 30);
LocalDate endDate = LocalDate.of(2021, 5, 1);
long years = ChronoUnit.YEARS.between(startDate, endDate);
long months = ChronoUnit.MONTHS.between(startDate, endDate);
long weeks = ChronoUnit.WEEKS.between(startDate, endDate);
long days = ChronoUnit.DAYS.between(startDate, endDate);
......
System.out.println(years); // 输出 0
System.out.println(months); // 输出 11
System.out.println(weeks); // 输出 48
System.out.println(days); // 输出 336
......
```

 - 线程安全的日期格式化类：`DateTimeFormatter`

日期和时间的格式化可通过`LocalDateTime`的format方法进行格式化，我们可以使用`DateTimeFormatter`预置的格式，也可以通过`DateTimeFormatter.ofPattern`方法来指定格式

```java
LocalDateTime dateTime = LocalDateTime.now();
String str = dateTime.format(DateTimeFormatter.ISO_DATE); // 输出2021-04-09
System.out.println(str);
str = dateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")); // 输出2021-04-09 16:30:17
System.out.println(str);
```
