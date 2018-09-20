# Kafka基础（安装与配置）
## 1. 下载代码
### 1.1 安装JVM环境
### 1.2 从下面地址下载Kafka
```
https://kafka.apache.org/downloads
```
### 1.3 解压压缩包并进入Kafka目录
```
c
cd kafka_2.11-0.10.2.1
```
## 2. 启动服务器
### 2.1 启动ZooKeeper
- Kafka使用ZooKeeper，所以您需要先启动一个ZooKeeper服务器。
- 如果您还没有。您可以使用随Kafka一起打包的便捷脚本来获取一个快速但是比较粗糙的单节点ZooKeeper实例。

```
bin/zookeeper-server-start.sh config/zookeeper.properties
或
bin/zookeeper-server-start.sh config/zookeeper.properties & (&符号表示：ctrl+C退出后仍在后台运行)
```

Kafka配置文件中主要包含3个配置：

```
# the directory where the snapshot is stored.
dataDir=/tmp/zookeeper
# the port at which the clients will connect
clientPort=2181
# disable the per-ip limit on the number of connections since this is a non-production config
maxClientCnxns=0
```

启动成功后，可以检查相应端口是否已被使用：

```
netstat -apn | grep 2181 
杀掉进程使用：kill -9 1403 （1403代表占用端口的进程号码）
```

### 2.2 修改Kafka配置
打开config/server.properties文件，在很靠前的位置有listeners和 advertised.listeners两处配置的注释，去掉这两个注释，并且根据当前服务器的IP修改如下：

```
# The address the socket server listens on. It will get the value returned from 
# java.net.InetAddress.getCanonicalHostName() if not configured.
#   FORMAT:
#     listeners = listener_name://host_name:port
#   EXAMPLE:
#     listeners = PLAINTEXT://your.host.name:9092
listeners=PLAINTEXT://:9092

# Hostname and port the broker will advertise to producers and consumers. If not set, 
# it uses the value for "listeners" if configured.  Otherwise, it will use the value
# returned from java.net.InetAddress.getCanonicalHostName().
advertised.listeners=PLAINTEXT://服务器IP:9092
```
如果是进行localhost本地测试，可按如下配置；（下面的测试，就是基于本地服务进行）
```
listeners=PLAINTEXT://127.0.0.1:9092
advertised.listeners=PLAINTEXT://127.0.0.1:9092
```

### 2.3 启动Kafka
```
bin/kafka-server-start.sh config/server.properties
```
## 3. 创建Topic
### 通过命令行创建Topic:
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
最后的test代表topic的名字
### 查看目前已经创建了的Topic:
```
bin/kafka-topics.sh --list --zookeeper localhost:2181
```
## 4. 启动生产者
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```
启动后，可以输入内容，然后回车。
## 5. 启动消费者
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```
可以将生产者所产生的消息，输出到控制台。
## 6. 测试多代理集群
以上使用的是单个代理。对Kafka来说，单个代理只是一个大小为一的集群，除了启动更多的代理实例外，没有什么变化。为了深入了解它，让我们把集群扩展到三个节点（仍然在本地机器上）。
### 6.1 为每个代理创建一个配置文件
```
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```
编辑这些新文件，设置如下属性：
```
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dir=/tmp/kafka-logs-1
```
```
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dir=/tmp/kafka-logs-2
```
'broker.id'属性是集群中每个节点的名称，这一名称是唯一且永久的。我们必须重写端口和日志目录，因为我们在同一台机器上运行这些，我们不希望所有的代理尝试在同一个端口注册，或者覆盖彼此的数据。
### 6.2 启动两个新的节点：
```
bin/kafka-server-start.sh config/server-1.properties &
...
```
```
bin/kafka-server-start.sh config/server-2.properties &
...
```
### 6.3 创建一个副本(replication)为3的新topic：
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```
### 6.4 运行"describe topics"命令来查看集群中的代理(broker)在做什么
```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
```
第一行给出了所有分区的摘要，下面的每行都给出了一个分区(Partition)的信息。因为我们只有一个分区，所以只有一行。
- “leader”是负责给定分区所有读写操作的节点。每个节点都是随机选择的部分分区的领导者。
- “replicas”是复制分区日志的节点列表，不管这些节点是leader还是仅仅活着。
- “isr”是一组“同步”replicas，是replicas列表的子集，它活着并被指到leader。
### 6.5 向新topic发送消息：
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
```
### 6.6 消费新topic的消息：
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```
### 6.7 测试一下容错性：
杀掉当前的leader进程(Broker 1)
```
ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
kill -9 7564
```
### 6.8 使用"describe topics"命令来查看当前状态
```
bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
```
此时，此前的消息仍可继续消费。
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
```
## 7. Refer 
- [**官方QuickStart文档**](https://kafka.apache.org/quickstart)
- [**Kafka中文站**](http://kafka.apachecn.org) ：介绍、快速开始、案例、文档
- [Apache Kafka 入门 - 基本配置和运行](https://blog.csdn.net/isea533/article/details/73611035) : 包含基础的配置内容。全部内容共5部分，[参见](https://blog.csdn.net/isea533/article/category/6986649)
- [kafka快速开始教程](https://www.jianshu.com/p/efc8b9dbd3bd) ： 包含代理集群、Kafka Connect、Kafka Streams等高级内容
