# Spring Async

> 想必大家都知道同步调用与异步调用的区别。
> - 同步调用: 程序按预定顺序一步步执行，每一步必须等到上一步执行完后才能执行。
> - 异步调用: 无需等待上一步执行完，程序就可以接着执行后续的操作。
>
> 目前大部分Java应用都是通过同步的方式来实现交互处理的，但是某些场景下，如果一个主流程和一个本可以 “在后台执行” 的流程放在一起执行，此时使用同步调用则需要等待所有执行完毕，会导致响应迟缓，影响体验，此时我们应该使用异步调用。
>
> 非Spring项目实现异步调用的方式，通常是使用多线程，手动在主线程外创建独立线程去完成相应的异步调用逻辑。

Spring的异步线程组件`@Async`主要实现主业务流程与附加业务逻辑的线程异步解耦。例如：
- 生成订单后，发送短信，通过在发送短信方法上使用Spring的 `@Async`实现发送短信的异步化。
- 为了记录用户的日常行为，需要记录操作日志，通过在日志记录方法上使用Spring的 `@Async`实现日志记录的异步化。

一个方法被带有Spring的`@Async`注解标记时，表明该方法为 “异步方法” ，它会在单独的线程上运行。该方法的返回类型也会变为`CompletableFuture<T>`。

# 如何开启@Async


## 基于Java Config

```java
@Configuration
@EnableAsync
public class SpringAsyncConfig {
    ... 
}

```

## 基于SpringBoot

```java
@EnableAsync
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```
## 基于XML

```xml
<task:executor id="myexecutor" pool-size="5"  />
<task:annotation-driven executor="myexecutor"/>
```

# @Async使用方式


## @Async：无返回结果的调用

## @Async：有返回结果的调用


# Spring @Async实现原理


# 参考

- [《Apache SkyWalking实战》](https://book.douban.com/subject/35148417/)
