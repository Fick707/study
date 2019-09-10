# Kafka常用命令
[kafka常用命令收录](https://www.cnblogs.com/aquester/p/9891475.html)

## 管理
* 创建主题（4个分区，2个副本）
```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 2 --partitions 4 --topic test
```

* 删除主题
```
bin/kafka-topics.sh --delete --zookeeper 127.0.0.1:2181 --topic test0
```

* 平衡leader
```
bin/kafka-preferred-replica-election.sh --zookeeper 127.0.0.1:2181/chroot
```

## 查询
* 查询指定topic详情（分区，副本等），如果不指定topic,则显示所有topic的详情
```
bin/kafka-topics.sh --describe --zookeeper 127.0.0.1:2181 --topic test
```

* 查看topic
```
bin/kafka-topics.sh --zookeeper 127.0.0.1:2181 --list
```

* 查看消费者group列表，新版本信息存在broker里，用bootstrap-server参数；旧版本存放在zk上，用zookeeper参数；
```
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --list
bin/kafka-consumer-groups.sh --zookeeper 127.0.0.1:2181 --list
```

* 显示某个消费组的消费详情（仅支持offset存储在zookeeper上的）
```
bin/kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --zookeeper localhost:2181 --group test
```

* 显示某个消费组的消费详情（支持0.9版本+）
```
bin/kafka-consumer-groups.sh --new-consumer --bootstrap-server localhost:9092 --describe --group test-consumer-group
```

## 发送和消费
* 生产者
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

* 消费者
```
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test
```

* 新生产者（支持0.9版本+）
```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test --producer.config config/producer.properties
```

* 新消费者（支持0.9版本+）
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --new-consumer --from-beginning --consumer.config config/consumer.properties
```

* 高级点的用法
```
bin/kafka-simple-consumer-shell.sh --brist localhost:9092 --topic test --partition 0 --offset 1234  --max-messages 10
```

## 