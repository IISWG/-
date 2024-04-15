在 RocketMQ 中消息发送者、消息消费者统称为客户端，对应 RocketMQ 的 Client 模块。
大家在使用 RocketMQ 进行消息发送时，需要引入如下 Maven 依赖：
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.7.1</version>
</dependency>
```
# 消息发送 API 详解
RocketMQ 消息发送者的核心类层次结构如下图所示：
![[Pasted image 20240411145436.png]]
## **MQAdmin**
MQ 基本的管理接口，提供对 MQ 提供基础的管理能力，其方法说明如下。
```java
void createTopic(String key, String newTopic, int queueNum, int topicSysFlag)

```

创建 Topic，其参数含义如下：

- String key：根据 key 查找 Broker，即新主题创建在哪些 Broker 上
- String newTopic：主题名称
- int queueNum：主题队列个数
- int topicSysFlag：主题的系统参数

```java
long searchOffset(MessageQueue mq, long timestamp)
```
根据队列与时间戳，从消息消费队列中查找消息，返回消息的物理偏移量（在 commitlog 文件中的偏移量）。参数列表含义如下：

- MessageQueue mq：消息消费队列
- long timestamp：时间戳
```java
long maxOffset(final MessageQueue mq)
```

查询消息消费队列当前最大的逻辑偏移量，在 consumequeue 文件中的偏移量。

```java
long minOffset(final MessageQueue mq)
```

查询消息消费队列当前最小的逻辑偏移量。

```java
long earliestMsgStoreTime(MessageQueue mq)
```

返回消息消费队列中第一条消息的存储时间戳。

```Java
MessageExt viewMessage(String offsetMsgId)
```

根据消息的物理偏移量查找消息。

```Java
MessageExt viewMessage(String topic, String msgId)
```

根据主题与消息的全局唯一 ID 查找消息。

```Java
QueryResult queryMessage(String topic, String key, int maxNum, long begin,long end)
```

批量查询消息，其参数列表如下：

- String topic：主题名称
- String key：消息索引 Key
- int maxNum：本次查询最大返回消息条数
- long begin：开始时间戳
- long end：结束时间戳
## **MQProducer**
消息发送者接口。核心接口说明如下：

```Java
void start()
```

启动消息发送者，在进行消息发送之前必须先调用该方法。

```Java
void shutdown()
```

关闭消息发送者，如果不需要再使用该生产者，需要调用该方法释放资源。

```Java
List <MessageQueue> fetchPublishMessageQueues(String topic)
```

根据 Topic 查询所有的消息消费队列列表。

```Java
SendResult send(Message msg, long timeout)
```

同步发送，参数列表说明如下：

- Message msg：待发送的消息对象
- long timeout：超时时间，默认为 3s

```Java
void send(Message msg, SendCallback sendCallback, long timeout)
```

异步消息发送，参数列表说明如下：

- Message msg：待发送的消息
- SendCallback sendCallback：异步发送回调接口
- long timeout：发送超时时间，默认为 3s

```Java
void sendOneway(Message msg)
```

Oneway 消息发送模式，该模式的特点是不在乎消息的发送结果，无论成功或失败。

```Java
SendResult send(Message msg, MessageQueue mq)
```

指定消息队列进行消息发送，其重载方法分表代表同步、异步 Oneway 发送模式。

```Java
SendResult send(Message msg, MessageQueueSelector selector, Object arg,long timeout)
```

消息发送时使用自定义队列负载机制，由 MessageQueueSelector 实现，Object arg 为传入 selector 中的参数。MessageQueueSelector 声明如下，此处的 arg 为 select 方法的第三个参数。
![[Pasted image 20240411150615.png]]
同样该方法的重载方法支持异步、Oneway 方式。
发送事务消息，事务消息只有同步发送方式，其中 Object arg 为额外参数，用在事务消息回调相关接口。
![[Pasted image 20240411150813.png]]
**该 API 从 4.3.0 版本开始引入**。

```java
SendResult send(Collection<Message> msgs, MessageQueue mq, long timeout)
```

指定消息消费队列批量发送消息，批量发送只有同步发送模式。

```java
SendResult send(Collection<Message> msgs, long timeout)
```

批量消息发送，超时时间默认为 3s。

```java
Message request(Message msg, long timeout)
```

RocketMQ 在 4.6.0 版本中引入了 request-response 请求模型，就消息发送者发送到 Broker，需要等消费者处理完才返回，该 request 的重载方法与 send 方法一样，在此不再重复介绍。
## **ClientConfig**
客户端配置相关。这里先简单介绍几个核心参数，后续在实践部分还会重点介绍。

- String namesrvAddr：NameServer 地址
- String clientIP：客户端的 IP 地址
- String instanceName：客户端的实例名称
- String namespace：客户端所属的命名空间
**DefaultMQProducer**

消息发送者默认实现类。

**TransactionMQProducer**

事务消息发送者默认实现类。
# 消息发送 API 简单使用示例
示例代码如下：

```java
public class Producer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new 
            DefaultMQProducer("please_rename_unique_group_name");
        producer.setNamesrvAddr("127.0.0.1:9876");
        producer.start();
        //发送单条消息
        Message msg = new Message("TOPIC_TEST", "hello rocketmq".getBytes());
        SendResult sendResult = null;
        sendResult = producer.send(msg);
        //　输出结果
        System.out.printf("%s%n", sendResult);
        //　发送带 Key 的消息
        msg = new Message("TOPIC_TEST", null, "ODS2020072615490001", "{\"id\":1, \"orderNo\":\"ODS2020072615490001\",\"buyerId\":1,\"sellerId\":1  }".getBytes());
        sendResult = producer.send(msg);
        //　输出结果
        System.out.printf("%s%n", sendResult);
        //　批量发送
        List<Message> msgs = new ArrayList<>();
        msgs.add( new Message("TOPIC_TEST", null, "ODS2020072615490002", "{\"id\":2, \"orderNo\":\"ODS2020072615490002\",\"buyerId\":1,\"sellerId\":3  }".getBytes()) );
        msgs.add( new Message("TOPIC_TEST", null, "ODS2020072615490003", "{\"id\":4, \"orderNo\":\"ODS2020072615490003\",\"buyerId\":2,\"sellerId\":4  }".getBytes()) );
        sendResult = producer.send(msgs);
        System.out.printf("%s%n", sendResult);
        //　使用完毕后，关闭消息发送者
        producer.shutdown();
    }
}
```
# 消息发送 API 版本演变说明
## **Namespace 概念的引入**
在 RocketMQ 4.5.1 版本正式引入了 Namespace 概念，在 API 上体现在构建 DefaultMQProducer 上，如下图所示：
![[Pasted image 20240411153334.png]]
当然消息发送者与消息消费者的命名空间必须一样，才能彼此协作。

**一言以蔽之，Namespace 主要为消息发送者、消息消费者进行分组，底层的逻辑是改变 Topic 的名称。**

## **request-response 响应模型 API**

RocketMQ 在 4.6.0 版本中引入了 request-response 请求模型，就是消息发送者发送到 Broker，需要等消费者处理完才返回。其相关的 API 如下：
![[Pasted image 20240411153506.png]]
## **消息轨迹与 ACL**
RocketMQ 在 4.4.0 开始引入了消息轨迹与 ACL。

- **消息轨迹**：支持跟踪消息发送、消息消费的全过程，即能跟踪消息的发送 IP、存储服务器，什么时候被哪个消费者消费。
- **ACL**：访问权限控制，即可以 Topic 消息发送、订阅进行授权，只有授权用户才能发送消息到指定 Topic。

相关的 API 变更如下：
![[Pasted image 20240411153528.png]]
