# Flink Window





# Flink的时间语义

在 Flink 的流式处理中，会涉及到时间的不同概念

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/assets/flink-time-semantics.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Event-Time、Ingestion-Time 与 Processing-Time</div>
	<br>
	<br>
</center>

- **Event Time**（事件时间）：事件创建的时间。它通常由事件中的时间戳描述，例如采集的日志数据中，每一条日志都会记录自己的生成时间，Flink 通过时间戳分配器访问事件时间戳。
- **Ingestion Time**（进入时间）：数据进入 Flink 的时间。
- **Processing Time**（处理时间）：是每一个执行基于时间操作的算子的本地系统时间，以本地机器时间来衡量。



> **[!TIP]**
> Q：对于业务来说，要统计 1min 内的故障日志个数，哪个时间是最有意义的？
>
> A：应该是 Event Time，因为我们要根据日志的生成时间进行统计



# Watermark