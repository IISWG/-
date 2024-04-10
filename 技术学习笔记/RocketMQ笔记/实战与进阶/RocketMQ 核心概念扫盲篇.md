# RocketMQ 部署架构
![[Pasted image 20240319172645.png]]
在 RocketMQ 主要的组件如下。

**NameServer**

NameServer 集群，Topic 的路由注册中心，为客户端根据 Topic 提供路由服务，从而引导客户端向 Broker 发送消息。NameServer 之间的节点不通信。路由信息在 NameServer 集群中数据一致性采取的最终一致性。

**Broker**

消息存储服务器，分为两种角色：Master 与 Slave，上图中呈现的就是 2 主 2 从的部署架构，在 RocketMQ 中，主服务承担读写操作，从服务器作为一个备份，当主服务器存在压力时，从服务器可以承担读服务（消息消费）。所有 Broker，包含 Slave 服务器每隔 30s 会向 NameServer 发送心跳包，心跳包中会包含存在在 Broker 上所有的 Topic 的路由信息。

**Client**

消息客户端，包括 Producer（消息发送者）和 Consumer（消费消费者）。客户端在同一时间只会连接一台 NameServer，只有在连接出现异常时才会向尝试连接另外一台。客户端每隔 30s 向 NameServer 发起 Topic 的路由信息查询。

> 温馨提示：NameServer 是在内存中存储 Topic 的路由信息，持久化 Topic 路由信息的地方是在 Broker 中，即 `${ROCKETMQ_HOME}/store/config/topics.json`。

在 RocketMQ 4.5.0 版本后引入了多副本机制，即一个复制组（m-s）可以演变为基于 Raft 协议的复制组，复制组内部使用 Raft 协议保证 Broker 节点数据的强一致性，该部署架构在金融行业用的比较多。
## 消息订阅模型
在 RocketMQ 的消息消费模式采用的是发布与订阅模式。

- Topic：一类消息的集合，消息发送者将一类消息发送到一个主题中，例如订单模块将订单发送到 order_topic 中，而用户登录时，将登录事件发送到 user_login_topic 中。
- ConsumerGroup：消息消费组，一个消费单位的“群体”，消费组首先在启动时需要订阅需要消费的 Topic。一个 Topic 可以被多个消费组订阅，同样一个消费组也可以订阅多个主题。一个消费组拥有多个消费者。
例如我们在开发一个订单系统，其中有一个子系统：order-service-app，在该项目中会创建一个消费组 order_consumer 来订阅 order_topic，并且基于分布式部署，order-service-app 的部署情况如下：
![[Pasted image 20240320085648.png]]
即 order-service-app 部署了 3 台服务器，每一个 JVM 进程可以看做是消费组 order_consumer 消费组的其中一个消费者。
#### **消费模式**

那这三个消费者如何来分工来共同消费 order_topic 中的消息呢？

在 RocketMQ 中支持广播模式与集群模式。

- **广播模式**：一个消费组内的所有消费者每一个都会处理 Topic 中的每一条消息，通常用于**刷新内存缓存**。
- **集群模式**：一个消费组内的所有消费者共同消费一个 Topic 中的消息，即分工协作，一个消费者消费一部分数据，启动**负载均衡**。
集群模式是非常常见的，可以横向扩容，当前消费者无法快速及时处理消息时，可以增加消费者数量横向扩容，快速提高消费能力，及时处理积压的消息。
#### **消费队列负载算法与重平衡机制**
