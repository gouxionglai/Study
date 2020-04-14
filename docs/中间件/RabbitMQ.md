# RabbitMQ

消息中间件

市面上常用的消息中间件有：activeMQ, rabbitMQ, kafka, rocketMQ

kafka适合大数据，高吞吐量，效率极高，但是有一定的误差。

rabbitMQ中规中矩，对数据一致性、稳定性、可靠性要求很高的适用。对性能要求其次。

为什么rabbitMQ高性能的原因：Erlang语言开发，和原生socket一样的低延迟

![img](D:\gitubDATA\Study\docs\中间件\images\mq知识框架.png)

# 安装

1. 推荐docker方式安装，官网有教程
2. 安装docker管理插件：rabbitmq-plugins enable rabbitmq_management
3. 启动服务：systemctl start rabbitmq-server
4. 查看服务状态：systemctl status rabbitmq-server
5. 重启服务：systemctl restart rabbitmq-server
6. 停止服务：systemctl stoprabbitmq-server



# 核心概念

![img](D:\gitubDATA\Study\docs\中间件\images\rabbitMQ_模型.png)

- channel

  网络信道，类似session的概念，每建立一个连接，就代表一个channel.

- Message

  消息。有properties 和 body 组成

- virtual host

  虚拟主机。最上层的消息路由，用于逻辑隔离。

  一个virtual host里面有若干的Exchange + Queue 

  同一个virtual host里面不能有相同的Exchange 或者 Queue

- exchange

  交换机，接受消息，然后根据路由键发送消息到绑定的队列

- queue

  消息队列，存放消息。

- binding

  绑定exchange和queue

- routing key

  路由规则，虚拟机用来确定如何路由一个特定消息。



# 初级

## 消息类型

exchange类型

- direct exchange
  - 点对点通信
  - 所有发送到direct exchange的消息被转发到routing key 中指定的queue

- topic exhcange
  - 根据规则匹配分发消息，有点像direct，只不过是模糊匹配的。

- fanout exchange
  - 广播式（订阅）
  - 不管routing key是什么，只要绑定了就会推送。速度最快。

## 绑定

binding



## 消息

message = properties + body

### 其他属性

content_type 消息头

content_encoding 消息编码

priority 优先级



# 高级

## 概念与设置

### 可靠性投递

1. 确保消息**成功发出**
   1. 消息落库，对消息状态进行打标：每一条消息都做持久化存数据库，并且做标记。（性能不高）
   2. 消息延迟投递，做二次确认（重点是这个，减少了持久化操作来recheck），回调检查，如果二次确认失败则需要做补偿。
2. 确保MQ节点**成功接受**
3. 发送端收到MQ节点**确认应答**  
4. 完善的消息进行**补偿机制**

### 幂等

多次重复操作，只会成功一次。避免重复消费。

- 唯一ID+指纹码机制
  - 加版本号控制，每次操作先看版本号是否一致，一致才操作，不一致则不操作。 
  - update tableX set xx-1 ,version +1 where version = a;
  - 好处：简单
  - 坏处：需要写数据库，有写入瓶颈

- redis原子性
  - setEx(key,value);

### confirm,return

- confirm

  即consumer收到消息后反馈给producer的一种机制

  1. channel上开启确认模式：channel.confirmSelect();
  2. 在channel上添加监听：addConfirmListener，监听成功和失败的结果。然后根据结果进行处理（比如重新发送或者记录日志）

- return

  即那种投递不到的消息，需要return。对面有响应的是confirm，无响应的是return.



### 自定义消费者



消息ACK与重回队列

消费端进行消费的时候，由于业务异常可以进行日志记录，然后进行补偿；或者服务器宕机，手动ACK保障消费端消费成功。

ACK即确认消息无误；NACK代表确认消息但是消息有误。

```java
//参数1 固定的Envelope中的deliveryTag
//参数2 是否批量处理消息
channel.basicAck(envelope.getDeliveryTag(),false);
//最后一个参数代表是否重回队列。一般选择false
channel.basicNack(envelope.getDeliveryTag(),false,false);
```

重回队列：一般选择否。

### 消息限流

巨量的消息瞬间全部推送给消费端，但是无法同时处理大量的消息，所以需要限流。

措施：QOS, 服务质量保障，如果有一定数目的消息未被确认，则不进行消费新的消息。具体的数值可配置。

```java
void basicQos(int prefetchSize, int prefetchCount, boolean global) throws IOException;
//prefetchSize: 预读大小限制。默认0则不限制大小
//prefetchCount：一次性读取多少条消息。 N
//global：应用于channel上还是只应用于consumer上。
```



### TTL消息

time to live 剩余存活时间。

rabbitMQ支持定义消息过期时间，在消息发送的时候指定。

时间计算是从消息入列开始计算，只要超过了配置时间，则自动清除。（有点像redis的key  expire time）

### 死信队列

DLX： Dead-Letter-Exchange

指没有消费者进行消费的消息，会统一挪到一个队列，命名为死信队列。

产生死信的情况：

1. 消息被拒绝（basic.reject/ basic.nack ），并且requeue= false。
2. TTL过期。
3. 队列达到最大长度。

## 与spring整合

### Spring AMQP

### Spring boot2.0

### Spring cloud



## 集群



## 企业解决方案

### SET化架构

### 基础MQ消息组件设计思路