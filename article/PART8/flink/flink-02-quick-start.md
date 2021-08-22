# 运行环境

- JDK1.8 以上 
- Scala（可选）
- Maven 3.0.4 以上
- Hadoop Yarn 2.4 以上


# 上手官方模版
Flink提供了分别基于Java和Scala实现的模板。创建模板项目的方式有两种，一种方式是通过Maven archetype命令进行创建，另一种方式是通过Flink提供的Quickstart Shell脚本进行创建。

- Maven archetype

```ssh
mvn archetype:generate -DarchetypeGroupId=org.apache.flink -DarchetypeArtifactId=flink-quickstart-java -DarchetypeCatalog=https://repository.apache.org/content/repositories/snapshots/ -DarchetypeVersion=1.7.0
```

- Quickstart Shell脚本

```ssh
curl https://flink.apache.org/q/quickstart.sh | bash -s 1.7.0
```

# 第三方学习模版
几个优秀的学习案例：
```
git clone https://github.com/streaming-with-flink/examples-java
git clone https://github.com/streaming-with-flink/examples-scala
```

# 参考

1. [flink基础——安装和demo](https://www.jianshu.com/p/6763408a37fa)