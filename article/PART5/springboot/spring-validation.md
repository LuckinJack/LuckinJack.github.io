# 校验注解

## @Validated、@Valid

## 使用@Validated与@Valid进行校验

# 常见的校验注解

- `@NotEmpty` 用在集合类上面。加了@NotEmpty的String类、Collection、Map、数组，是不能为null或者长度为0
- `@NotBlank` 只用于String，不能为null且trim()之后size>0
- `@NotNull` 不能为null，但可以为empty，没有Size的约束
- `@Null` 限制只能为null
- `@NotNull` 限制必须不为null
- `@AssertFalse` 限制必须为false
- `@AssertTrue` 限制必须为true
- `@DecimalMax(value)` 限制必须为一个不大于指定值的数字
- `@DecimalMin(value)` 限制必须为一个不小于指定值的数字
- `@Digits(integer,fraction)` 
限制必须为一个小数，且整数部分的位数不能超过integer，小数部分的位数不能超过fraction
- `@Future` 限制必须是一个将来的日期
- `@Max(value)` 限制必须为一个不大于指定值的数字
- `@Min(value)` 限制必须为一个不小于指定值的数字
- `@Past` 验证注解的元素值（日期类型）比当前时间早
- `@Pattern(value)` 限制必须符合指定的正则表达式
- `@Size(max,min)` 限制字符长度必须在min到max之间

# 分组校验

# 自定义注解校验枚举类型

```xml
<dependency>
  <groupId>javax.validation</groupId>
  <artifactId>validation-api</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

如果使用的是Springboot，不用手动添加如上依赖了，Springboot已经引入了这个依赖

# 实现优雅的注解校验

<div align=center>
<img src="/assets/Koji.webp"/>
</div>