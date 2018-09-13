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
## 6. Refer 
- [官方QuickStart文档](https://kafka.apache.org/quickstart)
- [Apache Kafka 入门 - 基本配置和运行](https://blog.csdn.net/isea533/article/details/73611035) : 包含基础的配置内容
- [kafka快速开始教程](https://www.jianshu.com/p/efc8b9dbd3bd) ： 包含代理集群、Kafka Connect、Kafka Streams等高级内容
