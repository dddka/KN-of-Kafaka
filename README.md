# KN-of-Kafaka
# kafaka为啥快
1、顺序读写：
磁盘读写的快慢取决于你怎么使用它，也就是顺序读写或者随机读写。在顺序读写的情况下，可以减少磁盘寻址，磁盘寻址是一个“机械动作”，是最耗时的。
2、零拷贝：
每一个Partition其实都是一个文件 ，收到消息后Kafka会把数据插入到文件末尾（虚框部分）这种方法有一个缺陷，没有办法删除数据 ，所以Kafka是不会删除数据的，它会把所有的数据都保留下来，每个消费者（Consumer）对每个Topic都有一个offset用来表示读取到了第几条数据(相当于是一个书签，标注你每次读的位置)
3.批量压缩
在部分情况下，系统的瓶颈不是CPU或磁盘，而是网络IO。进行数据压缩会消耗少量的CPU资源,不过对于kafka而言,网络IO更应该需要考虑。如果每个消息都压缩，但是压缩率相对很低，所以Kafka使用了批量压缩，即将多个消息一起压缩而不是单个消息压缩，缓解网络传输过程中的压力。
# 通信模式
1、点对点模式(queue)
消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并且消费消息。一条消息被消费以后，queue中就没有了，不存在重复消费。
优点：是消费者拉取消息的频率可以由自己控制。
缺点：消息队列是否有消息需要消费，在消费者端无法感知，所以在消费者端需要额外的线程去监控。
2、发布/订阅模式(topic)
消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。发布到topic的消息会被所有订阅者消费（类似于关注了微信公众号的人都能收到推送的文章）。
优点：消费者被动接收推送，所以无需感知消息队列是否有待消费的消息
缺点：消费者性能不一样，处理消息的能力也会不一样，但消息队列却无法感知消费者消费的速度！
假设三个消费者处理速度分别是8M/s、5M/s、2M/s，如果队列推送的速度为5M/s，则consumer3无法承受！推送的速度成了发布订阅模模式的一个问题！
# 应用场景
 一个典型的 Kafka 集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group(多个消费者实例可以组成一个消费者组，但是每个分区最多只能被消费者组中的一个实例消费)
# 高容错，高可用：副本机制
复制功能是 Kafka 架构的核心功能，在 Kafka 文档里面 Kafka 把自己描述为 一个分布式的、可分区的、可复制的提交日志服务。复制之所以这么关键，是因为消息的持久存储非常重要，这能够保证在主节点宕机后依旧能够保证 Kafka 高可用。副本机制也可以称为备份机制(Replication)，通常指分布式系统在多台网络交互的机器上保存有相同的数据备份/拷贝。
Kafka 使用主题来组织数据，每个主题又被分为若干个分区，分区会部署在一到多个 broker 上，每个分区都会有多个副本，所以副本也会被保存在 broker 上，每个 broker 可能会保存成千上万个副本。
