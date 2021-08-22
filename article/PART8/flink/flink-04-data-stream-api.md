# Flink 流处理API



...





## Environment

一个Flink作业必须依赖于一个执行环境。

### getExecutionEnvironment

`getExecutionEnvironment` 是最常用的一种创建执行环境的方式，它会根据查询运行的方式决定返回什么样的运行环境创建一个执行环境，表示当前执行程序的上下文：

- 如果程序是独立调用的，此方法返回本地执行环境

- 如果从命令行客户端调用程序以提交到集群，此方法返回该集群的执行环境

  

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
```

```java
StreamExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
```

> **[!NOTE]**
>
> 1. 如果没有设置并行度，会以 `flink-conf.yml` 中的配置为准，默认是 1。
>
> 2. 流处理和批处理的执行环境不同：
>
> - 流处理的执行环境名为org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
>
> - 批处理的执行环境名为org.apache.flink.api.java.ExecutionEnvironment



### createLocalEnvironment

`createLocalEnvironment`返回本地执行环境，需要在调用时指定默认的并行度

```java
LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(1);
```

### createRemoteEnvironment

返回集群执行环境，将 Jar 提交到远程服务器。需要在调用时指定 *JobManager* 的 IP 和端口号，并指定要在集群中运行的 Jar 包。

```java
StreamExecutionEnvironment env = 
StreamExecutionEnvironment.createRemoteEnvironment("jobmanage-hostname", 6123,"YOUR_PATH//FlinkJob.jar"));
```



## Source



### 从集合读取数据



### 从文件读取数据

```java
DataStream<String> dataStream = env.readTextFile("YOUR_FILE_PATH");
```



### 从kafka消息队列读取数据



```java
// kafka 配置项
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "consumer-group");
properties.setProperty("key.deserializer", 
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer", 
"org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("auto.offset.reset", "latest");

// 从 kafka 读取数据
DataStream<String> dataStream = env.addSource( new 
FlinkKafkaConsumer011<String>("sensor", new SimpleStringSchema(), properties));
```



### 自定义Source

我们还可以自定义 source数据来源，实现 SourceFunction 接口即可

```java
public static void main(String[] args) {
        LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment(1);
        DataStream<SensorReading> dataStream = env.addSource(new MySensor());
}

public static class MySensor implements SourceFunction<SensorReading> {
        private boolean running = true;
		@Override
        public void run(SourceContext<SensorReading> ctx) throws Exception {
            Random random = new Random();
            HashMap<String, Double> sensorTempMap = new HashMap<>();
            for (int i = 0; i < 10; i++) {
                sensorTempMap.put("sensor_" + (i + 1), 60 + random.nextGaussian() * 20);
            }
            while (running) {
                for (String sensorId : sensorTempMap.keySet()) {
                    Double newTemp = sensorTempMap.get(sensorId) + random.nextGaussian();
                    sensorTempMap.put(sensorId, newTemp);
                    ctx.collect(new SensorReading(sensorId, System.currentTimeMillis(),
                            newTemp));
                }
                Thread.sleep(1000L);
            }
        }

        public void cancel() {
            this.running = false;
        }
}
```



## TransForm

转换算子



### map

map中传入的是一个接口 *MapFunction<T,O>*，我们可以继承并重写 *MapFunction* 或 *RichMapFunction* 来自定义map()。


```java
// 从文件读取数据
DataStream<String> inputStream = env.readTextFile("D:\\Projects\\testData.txt");

//map，输出每一行的字符串长度
DataStream<Integer> mapStream = inputStream.map(new MapFunction<String, Integer>() {
    @Override
    public Integer map(String value) throws Exception {
                return value.length();
  }
});
```

```java
@Public
@FunctionalInterface
public interface MapFunction<T, O> extends Function, Serializable {

	/**
	 * The mapping method. Takes an element from the input data set and transforms
	 * it into exactly one element.
	 *
	 * @param value The input value.
	 * @return The transformed value
	 *
	 * @throws Exception This method may throw exceptions. Throwing an exception will cause the operation
	 *                   to fail and may trigger recovery.
	 */
	O map(T value) throws Exception;
}
```






### flatMap



### filter



### keyBy



### Rolling Aggregation（滚动聚合算子）

-  sum()
-  min()
-  max()
-  minBy()
-  maxBy()



### Reduce



### Split 和 Select

#### split



#### select



## Sink

Flink的对外输出操作都用 Sink 完成。