# kafka 原理总结

## kafka 历史
Kafka是最初由Linkedin公司开发，是一个分布式、支持分区的（partition）、多副本的（replica），基于zookeeper协调的分布式消息系统，它的最大的特性就是可以实时的处理大量数据以满足各种需求场景：比如基于hadoop的批处理系统、低延迟的实时系统、storm/Spark流式处理引擎，web/nginx日志、访问日志，消息服务等等，用scala语言编写，Linkedin于2010年贡献给了Apache基金会并成为顶级开源 项目。

# 前言

## Kafka的特性:

* 高吞吐量、低延迟：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。
* 可扩展性：kafka集群支持热扩展
* 持久性、可靠性：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
* 容错性：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
* 高并发：支持数千个客户端同时读写

##  Kafka的使用场景：

* 日志收集：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、Hbase、Solr等。
* 消息系统：解耦和生产者和消费者、缓存消息等。
* 用户活动跟踪：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
* 运营指标：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
* 流式处理：比如spark streaming和storm
* 事件源

## Kakfa的设计思想

### Kakfa Broker Leader的选举

* Kakfa Broker集群受Zookeeper管理;
* 所有的Kafka Broker节点一起去Zookeeper上注册一个临时节点，因为只有一个Kafka Broker会注册成功，其他的都会失败;
* 在Zookeeper上注册临时节点成功的Kafka Broker会成为Kafka Broker Controller，其他的叫Kafka Broker follower。（这个过程叫Controller在ZooKeeper注册Watch）；
* 这个Controller会监听其他的Kafka Broker的所有信息；
	* 如果有一个broker宕机了，kafka broker controller会读取该宕机broker上所有的partition在zookeeper上的状态，并选取ISR列表中的一个replica作为partition leader
	* 如果ISR列表中的replica全挂，选一个幸存的replica作为leader; 
	* 如果该partition的所有的replica都宕机了，则将新的leader设置为-1，等待恢复;
	* 等待ISR中的任一个Replica“活”过来，并且选它作为Leader,或选择第一个“活”过来的Replica（不一定是ISR中的）作为Leader;
	* broker宕机的事情，kafka controller也会通知zookeeper，zookeeper就会通知其他的kafka broker;
* 如果kafka broker controller宕机了，所有的kafka broker又会一起去Zookeeper上注册临时节点，会又有一个成为Kafka Broker Controller；

### Consumer Group

* 各个consumer（consumer 线程）可以组成一个分组（Consumer group ）；
* partition中的每个message只能被分组中的一个consumer消费；
* 如果一个message想被多个consumer消费的话，那么这些consumer必须在不同的分组；
* Kafka不支持一个partition中的message由同一个分组下的两个或两个以上的consumer来处理；
* 新启动的consumer默认从partition队列最头端最新的地方开始阻塞的读message。
* 某个分组下面的所有consumer 一定会消费全部的partition；即便这个分组下只有一个consumer；
* 最优的设计就是:分组下的consumer 的数量等于partition数量，这样效率是最高的；
* 我们在设定分组的时候，只需要指明有几个consumer即可，无需指定对应的消费partition序号，consumer会自动进行rebalance；

### Consumer Rebalance的触发条件

1. Consumer增加或删除;
2. Broker的增加或者减少;

### Consumer

* Consumer处理partition里面的message的时候是o（1）顺序读取的,所以必须维护着上一次读到哪里的offsite信息;
* high level API,offset存于Zookeeper中，low level API的offset由自己维护;
* 一般来说都是使用high level api的;
* Consumer的交付保证，默认是读完message先commmit再处理；autocommit默认是true。

### Topic & Partition

* Topic相当于传统消息系统MQ中的一个队列queue；多个partition时，每个partiton相当于是一个子queue
* kafka会把收到的message进行load balance，均匀的分布在这个topic下的不同的partition上（ hash(message) % [broker数量]  ）；
* 在物理结构上，每个partition对应一个物理的目录，目录命名是[topicname]_[partition]_[序号]；
* 一个topic可以有无数多的partition；
* 在kafka配置文件中可随时通过num.partitions参数来配置topic的默认partition数量；
* Topic创建之后通过Kafka提供的工具也可以修改partiton数量；
* Kafka Topic Partition 如何决定：
	1. 一个Topic的Partition数量大于等于Broker的数量，可以提高吞吐率;
	2. 同一个Partition的Replica尽量分散到不同的机器，高可用;
	3. 越多的分区需要打开更多的文件句柄,如果超过操作系统限制，则会出大问题；
	4. 更多的分区会导致生产端对消费端的延迟；
	5. 越多的partition意味着需要更多的内存。批量提交和消费需要占用内存；
	6. 越多的partition会导致更长时间的恢复期;
* 当新增partition时，原分区中的数据不会重新分配；新来的消息才会重参与所有分区的负载均衡；

### Partition Replica 分区副本
* 每个partition可以在其他的kafka broker节点上存副本；
* 存replica副本的方式是按照kafka broker的顺序存。例如有5个kafka broker节点，某个topic有3个partition，每个partition存2个副本，那么partition1存broker1,broker2，partition2存broker2,broker3。。。以此类推；
* replica副本数目不能大于kafka broker节点的数目，否则报错；
* 这里的replica数其实就是partition的副本总数;
* 主分区是partition leader，其他的就follower;
	1. 怎样传送消息：producer先把message发送到partition leader，再由leader发送给其他partition follower;
	2. 在向Producer发送ACK前需要保证有多少个Replica已经收到该消息：根据ack配的个数而定;
	3. 怎样处理某个Replica不工作的情况：
		* 如果这个partition replica不在ack列表中，就是partition leader向partition follower发送message没有响应而已，这个不会影响整个系统；
		* 如果这个partition replica在ack列表中，producer发送的message的时候会等待这个partition replca写message成功，直到time out，然后返回失败。此时kafka会自动的把这个partition replica从ack列表中移除，以后的producer发送message的时候就不会有这个ack列表下的这个partition replica了。 
	4. 怎样处理Failed Replica恢复回来的情况：
		* 如果这个partition replica之前不在ack列表中，那么启动后重新受Zookeeper管理即可，之后producer发送message的时候，partition leader会继续发送message到这个partition follower上;
		* 如果这个partition replica之前在ack列表中，需要把这个partition replica再手动加到ack列表中。（ack列表是手动添加的，出现某个部工作的partition replica的时候自动从ack列表中移除的）

#### Partition leader与follower

* partition也有leader和follower之分；
* leader是主partition，producer写kafka的时候先写partition leader，再由partition leader push给其他的partition follower；
* partition leader与follower的信息受Zookeeper控制；
* 一旦partition leader所在的broker节点宕机，zookeeper会从其他的broker的partition follower上选择follower变为parition leader。

#### Topic分配partition和partition replica的算法：

1. 将Broker（size=n）和待分配的Partition排序;
2. 将第i个Partition分配到第（i%n）个Broker上;
3. 将第i个Partition的第j个Replica分配到第（(i + j) % n）个Broker上.

### 消息投递可靠性

* Kafka提供了三种模式:
	1. 第一种是啥都不管，发送出去就当作成功，这种情况当然不能保证消息成功投递到broker；
	2. 第二种是Master-Slave模型，只有当Master和所有Slave都接收到消息时，才算投递成功，这种模型提供了最高的投递可靠性，但是损伤了性能；
	3. 第三种模型，即只要Master确认收到消息就算投递成功；
* 实际使用时，根据应用特性选择，绝大多数情况下都会中和可靠性和性能选择第三种模型;
* 消息在broker上的可靠性:
	* 因为消息会持久化到磁盘上，所以如果正常stop一个broker，其上的数据不会丢失；
	* 但是如果不正常stop，可能会使存在页面缓存来不及写入磁盘的消息丢失，这可以通过配置flush页面缓存的周期、阈值缓解，但是同样会频繁的写磁盘会影响性能。
* 消息消费的可靠性:
	* Kafka提供的是“At least once”模型，因为消息的读取进度由offset提供，offset可以由消费者自己维护也可以维护在zookeeper里;
	* 当消息消费后consumer挂掉，offset没有即时写回，就有可能发生重复读的情况，这种情况同样可以通过调整commit offset周期、阈值缓解，甚至消费者自己把消费和commit offset做成一个事务解决，但是如果你的应用不在乎重复消费，那就干脆不要解决，以换取最大的性能。

### Partition ack

* 当ack=1，表示producer写partition leader成功后，broker就返回成功;
* 当ack=2，表示producer写partition leader和其他一个follower成功的时候，broker就返回成功;
* 当ack=-1[parition的数量]的时候，表示只有producer全部写成功的时候，才算成功;
* 这里需要注意的是，如果ack=1的时候，一旦有个broker宕机导致partition的follower和leader切换，会导致丢数据;

### 其他

* message状态：在Kafka中，消息的状态被保存在consumer中，broker不会关心哪个消息被消费了被谁消费了，只记录一个offset值；
* message持久化：Kafka中会把消息持久化到本地文件系统中，并且保持o(1)极高的效率。我们众所周知IO读取是非常耗资源的性能也是最慢的，这就是为了数据库的瓶颈经常在IO上，需要换SSD硬盘的原因。但是Kafka作为吞吐量极高的MQ，却可以非常高效的message持久化到文件。这是因为Kafka是顺序写入o（1）的时间复杂度，速度非常快。也是高吞吐量的原因。由于message的写入持久化是顺序写入的，因此message在被消费的时候也是按顺序被消费的，保证partition的message是顺序消费的。一般的机器,单机每秒100k条数据。
* message有效期：Kafka会长久保留其中的消息，以便consumer可以多次消费，当然其中很多细节是可配置的。
* Produer : Producer向Topic发送message，不需要指定partition，直接发送就好了。kafka通过partition ack来控制是否发送成功并把信息返回给producer，producer可以有任意多的thread，这些kafka服务器端是不care的。Producer端的delivery guarantee默认是At least once的。也可以设置Producer异步发送实现At most once。Producer可以用主键幂等性实现Exactly once
* Kafka高吞吐量： 
	* Kafka的高吞吐量体现在读写上，分布式并发的读和写都非常快；
	* 写的性能体现在以o(1)的时间复杂度进行顺序写入；
	* 读的性能体现在以o(1)的时间复杂度进行顺序读取；
	* 对topic进行partition分区，consume group中的consume线程可以以很高能性能进行顺序读。
* Kafka delivery guarantee(message传送保证)：
	* At most once消息可能会丢，绝对不会重复传输；
	* At least once 消息绝对不会丢，但是可能会重复传输；
	* Exactly once每条信息肯定会被传输一次且仅传输一次，这是用户想要的。
* 批量发送：Kafka支持以消息集合为单位进行批量发送，以提高push效率。
* push-and-pull : Kafka中的Producer和consumer采用的是push-and-pull模式，即Producer只管向broker push消息，consumer只管从broker pull消息，两者对消息的生产和消费是异步的。
* Kafka集群中broker之间的关系：不是主从关系，各个broker在集群中地位一样，我们可以随意的增加或删除任何一个broker节点。
* 负载均衡方面： Kafka提供了一个 metadata API来管理broker之间的负载;
* 同步异步：Producer采用异步push方式，极大提高Kafka系统的吞吐率（可以通过参数控制是采用同步还是异步方式）。
* 离线数据装载：Kafka由于对可拓展的数据持久化的支持，它也非常适合向Hadoop或者数据仓库中进行数据装载。
* 实时数据与离线数据：kafka既支持离线数据也支持实时数据，因为kafka的message持久化到文件，并可以设置有效期，因此可以把kafka作为一个高效的存储来使用，可以作为离线数据供后面的分析。当然作为分布式实时消息系统，大多数情况下还是用于实时的数据处理的，但是当cosumer消费能力下降的时候可以通过message的持久化在淤积数据在kafka。
* 插件支持：现在不少活跃的社区已经开发出不少插件来拓展Kafka的功能，如用来配合Storm、Hadoop、flume相关的插件。
* 解耦:  相当于一个MQ，使得Producer和Consumer之间异步的操作，系统之间解耦
* 冗余:  replica有多个副本，保证一个broker node宕机后不会影响整个服务
* 扩展性:  broker节点可以水平扩展，partition也可以水平增加，partition replica也可以水平增加
* 峰值:  在访问量剧增的情况下，kafka水平扩展, 应用仍然需要继续发挥作用
* 可恢复性:  系统的一部分组件失效时，由于有partition的replica副本，不会影响到整个系统。
* 顺序保证性：由于kafka的producer的写message与consumer去读message都是顺序的读写，保证了高效的性能。
* 缓冲：由于producer那面可能业务很简单，而后端consumer业务会很复杂并有数据库的操作，因此肯定是producer会比consumer处理速度快，如果没有kafka，producer直接调用consumer，那么就会造成整个系统的处理速度慢，加一层kafka作为MQ，可以起到缓冲的作用。
* 异步通信：作为MQ，Producer与Consumer异步通信


https://blog.csdn.net/lingbo229/article/details/80761778