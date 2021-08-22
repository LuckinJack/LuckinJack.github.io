
在pom.xml文件中引入所需的依赖spring-boot-starter-actuator:
```xml
<!-- health check-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

访问http://localhost:port/actuator，可以看到如下输出
```json
{
    "_links": {
        "self": {
            "href": "http://192.168.47.1:10000/actuator",
            "templated": false
        },
        "health-path": {
            "href": "http://192.168.47.1:10000/actuator/health/{*path}",
            "templated": true
        },
        "health": {
            "href": "http://192.168.47.1:10000/actuator/health",
            "templated": false
        },
        "info": {
            "href": "http://192.168.47.1:10000/actuator/info",
            "templated": false
        }
    }
}
```

>
> 如果是在SpringCloud中使用，SpringBoot Actuator和Eureka Server是有区别的，后者并不记录Eureka Client的所有健康状况信息，仅仅维护了一个实例清单。
> 不过我们可以进入实例中，查看健康状况。
> - healthCheckUrl: 它的默认URL地址为/actuator/health
> - statusPageUrl:  它的默认URL地址为/actuator/info
>


```xml
#启用shutdown
management.endpoint.shutdown.enabled=true
#禁用密码验证
management.endpoint.shutdown.sensitive=false
#开启所有的端点
management.endpoints.web.exposure.include=*
```