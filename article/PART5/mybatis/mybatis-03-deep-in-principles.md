

mybatis为何可以只写接口而不写实现类?



通过mybatis源码分析可知：



mybatis通过JDK的动态代理方式，在启动加载配置文件时，根据配置mapper的xml去生成Dao的实现。session.getMapper()使用了代理，当调用一次此方法，都会产生一个代理class的instance,看看这个代理class的实现。











[超全MyBatis动态代理详解！（绝对干货）](https://blog.csdn.net/qq_42046105/article/details/112914184)

