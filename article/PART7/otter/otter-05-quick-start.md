# 手工安装模式

>  **[!NOTE]**
>  
>  1. 需要在Linux环境下进行安装部署
>  2. 环境配置：
>
>     - Linux
>     - MySQL（≥ 5.1）
>     - JDK （≥ 1.6）
>     - Zookeeper
>     - Otter Manager 
>     - Otter Node
>  3. 启动顺序

## 环境初始化
otter安装首先需要两台服务器，这里我们实现的是两台服务器之间的Mysql数据库双主同步，即双写同步。假设两个服务为A、B，下面进行安装


**1. 在AB上安装并配置JDK**


配置环境变量，在文件末尾加上
```sh
export JAVA_HOME=jdk文件路径
export JRE_HOME=jdk文件路径/jre 
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH

export PATH=$JAVA_HOME/bin:$PATH
```





**2. 在AB上安装并配置MySQL**

修改MySQL的`my.ini`文件

```sh
# 开启binlog
log-bin=mysql-bin
# 设置binlog格式为ROW
binlog-format=ROW
# 两个机房的serverid设置为不一样的值，不能和canal的slaveId重复
server_id=1
```
为Canal创建一个数据库用户，并分配相应权限
```sql
  CREATE USER canal IDENTIFIED BY 'canal';  
  GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
  FLUSH PRIVILEGES;
```
验证上述操作是否成功
```sh
 # 验证
 mysql> show variables like '%binlog_format%';
 +---------------+-------+
 | Variable_name | Value |
 +---------------+-------+
 | binlog_format | ROW   |
 +---------------+-------+
```
然后重启MySQL，使配置永久生效

**3. 安装Zookeeper**


##  Otter Manager 安装


## Otter Node 安装
node 需要aria2支持，在AB机房各安装一套。

otter node依赖于zookeeper进行分布式调度，需要安装一个zookeeper节点或者集群















# Docker安装模式






# 参考

 - [Otter wiki - Docker_QuickStart](https://github.com/alibaba/otter/wiki/Docker_QuickStart)
 - [Otter wiki - Manager_Quickstart](https://github.com/alibaba/otter/wiki/Manager_Quickstart)
 - [Otter wiki - Node_Quickstart](https://github.com/alibaba/otter/wiki/Node_Quickstart)
