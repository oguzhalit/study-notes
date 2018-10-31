# 13 Kafka
## 13.1 Kafka 的常用操作
### 13.1.1 启动 Kafka 集群

```shell
kafka-server-start.sh config/server.properties
```
### 13.1.2 创建 Kafka 主题

```shell
kafka-topics.sh --zookeeper hadoop1:2181 --topic calllog --create --replication-factor 1 --partitions 3
```
### 13.1.3 检查主题是否创建成功

```shell
kafka-topics.sh --zookeeper hadoop1:2181 --list
```
### 13.1.4 启动 Kafka 控制台消费者，等待 Flume 信息的输入

```shell
kafka-console-consumer.sh --zookeeper hadoop1:2181 --topic callog --from-beginning
```
