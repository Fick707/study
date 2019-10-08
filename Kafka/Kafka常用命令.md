# Kafka常用命令
[kafka常用命令收录](https://www.cnblogs.com/aquester/p/9891475.html)

## 管理
* 创建主题（4个分区，2个副本）
```
bin/kafka-topics.sh --create --zookeeper 10.91.0.57:2181 --replication-factor 2 --partitions 4 --topic test
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

* 查看消费者group列表，新版本信息存在broker里，用bootstrap-server参数；
```
bin/kafka-consumer-groups.sh --bootstrap-server 10.91.0.57:9092 --list
```

* 显示某个消费组的消费详情（支持0.9版本+）
```
bin/kafka-consumer-groups.sh --bootstrap-server 10.91.0.57:9092 --describe --group group_name | grep topic_name
```

## 发送和消费
* 生产者
```
bin/kafka-console-producer.sh --broker-list 10.91.0.57:9092 --topic test
```

* 消费者
```
bin/kafka-console-consumer.sh --bootstrap-server 10.91.0.57:9092 --topic CW_OCEAN_SN_FACEDB
```

* 新生产者（支持0.9版本+）
```
bin/kafka-console-producer.sh --broker-list 10.91.0.57:9092 --topic test --producer.config config/producer.properties
```

* 新消费者（支持0.9版本+）
```
bin/kafka-console-consumer.sh --bootstrap-server 10.91.0.57:9092 --topic test --new-consumer --from-beginning --consumer.config config/consumer.properties
```

* 高级点的用法
```
bin/kafka-simple-consumer-shell.sh --brist 10.91.0.57:9092 --topic test --partition 0 --offset 1234  --max-messages 10
```

## 