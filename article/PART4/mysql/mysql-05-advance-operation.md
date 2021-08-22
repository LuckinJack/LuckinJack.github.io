# 主从复制

## 主从同步的原理

首先先了解 mysql 主从同步的原理

1. **master 提交完事务后，写入 binlog**
2. **slave 连接到 master，获取 binlog**
3. **master 创建 dump 线程，推送 binglog 到 slave**
4. **slave 启动一个 IO 线程读取同步过来的 master 的 binlog，记录到 relay log 中继日志中**
5. **slave 再开启一个 sql 线程读取 relay log 事件并在 slave 执行，完成同步**
6. **slave 记录自己的 binglog**

![](https://static001.geekbang.org/infoq/b0/b0831132b9b2fc1c7d1d5370cfb1ddb1.jpeg?x-oss-process=image/resize,p_80/auto-orient,1)

由于 mysql 默认的复制方式是异步的，主库把日志发送给从库后不关心从库是否已经处理，这样会产生一个问题就是假设主库挂了，从库处理失败了，这时候从库升为主库后，日志就丢失了。由此产生两个概念。
全同步复制
主库写入 binlog 后强制同步日志到从库，所有的从库都执行完成后才返回给客户端，但是很显然这个方式的话性能会受到严重影响。
半同步复制
和全同步不同的是，半同步复制的逻辑是这样，从库写入日志成功后返回 ACK 确认给主库，主库收到至少一个从库的确认就认为写操作完成。


[《我要进大厂》之mysql夺命连环13问](https://xie.infoq.cn/article/7d6bee26d18de35c65ca56ea7)