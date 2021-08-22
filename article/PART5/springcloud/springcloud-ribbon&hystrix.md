





































> [!NOTE| lable=如何配置hystrix的熔断时间与ribbon超时重试时间]
>
> 我们先来看一组配置：
>
>
> ```yml
> hystrix:
> command:
> default:
> execution:
> # 配置命令的执行，是否会超时
>  timeout:
>    enabled: true
>  isolation:
>    thread:
>    # hystrix的熔断时间，应当大于ribbon的超时重试机制
>     timeoutInMilliseconds: 35000
> ribbon:
> # 建立连接后，请求处理的超时时间 
> ReadTimeout: 30000
> # 建立连接的超时时间
> ConnectTimeout: 1000
> # 当前实例的重试次数
> MaxAutoRetries: 0
> # 切换实例的重试次数
> MaxAutoRetriesNextServer: 1
> # 所有操作请求都进行重试
> OkToRetryOnAllOperations: false
> ```
>
> 使用上面的配置时，控制台会打印出如下的日志，日志中说，Hystrix的超时配置时间少于了Ribbon的读取与连接超时时间。这种情况下，ribbon的超时重试将不起作用，，因为重试前Hystrix已经熔断了。
>
>
> ```
> [gateway] | 2020-06-26 20:59:32.532 | WARN | o.s.c.n.z.f.r.s.AbstractRibbonCommand.getHystrixTimeoutorg.springframework.cloud.netflix.zuul.filters.route.support.AbstractRibbonCommand : The Hystrix timeout of 35000ms for the command behavioranalysis is set lower than the combination of the Ribbon read and connect timeout, 62000ms
> ```
> 结合上方日志，可以看到 Hystrix 的 timeout 时间 35000ms 就是我们配置的 *timeoutInMilliseconds*，但是这个 *Ribbon read and connect timeout* 的值是62000ms ，怎么算出来的？
>
> 阅读 `AbstractRibbonCommand.class` 的源码，结合日志的输出验证后可得知，Ribbon的读取与连接超时时间配置的计算公式是：
>
> **Ribbon read and connect timeout = **
>
> **(ReadTimeout + ConnectTimeout) × (1 + MaxAutoRetries) × (MaxAutoRetriesNextServer + 1)** 
>
> 带入按照上图日志打印的时间进行验证，结果正确：
>
> Ribbon read and connect timeout = (30000 + 1000) * (0 + 1) * (1 + 1) = 62000