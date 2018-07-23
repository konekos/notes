# RabbitMQ

JMS组件 慢
AMQP组件 协议，有两个开源项目，RabbitMQ、Kafka
erlang开发的AMQP

组成：
- Exchange 交换空间
- Queue 数据队列
- RoutingKey 如果所以队列routkey都一样，则是广播消息。不一样是点对点。

vhost 在RabbitMQ，有虚拟主机概念，虚拟主机不同队列与用户。

## 安装erlang环境
## RabbitMQ安装配置

Rabbit有自己Web客户端，做一个管理员


## Java操作MQ
在进行队列信息创建的时候会有如下可用选项：   · name：队列的名称，这个名称一定是唯一的；   · 消息保存类型：持久化保存（druid）、瞬时状态（Transient）；   · 允许自动删除。  本次建立一个“mldn.msg.queue”消息，该消息为持久消息，并且允许自动删除

## 虚拟主机
可以说是rabbitmq中的重大特点，实际上之前就已经存在有虚拟主机。会存在一个默认的虚拟主机，用户也可以建立属于自己的虚拟主机
1.配置虚拟主机最简单的做法是直接在控制台通过界面创建。 如果不想通过界面创建，可以命令行创建。rabbitmqctl
factory.setVirtualHost("hello"); // 设置虚拟主机  

使用虚拟主机的最大好处可以区分不同用户的操作空间。



## 发布订阅模式

- 之前的操作都有缺陷。exchange routingkey没有考虑到
- 之前只是简单指定了一个队列名称
- 在消息系统之中，除了有队列消息之外，还有主题消息。要实现该功能，就要使用exchange：一方面接收生产者的消息，另一方面负责向指定队列推送。

在 Exchange 之中有如下的几种类型：topic、director、fanout

- 广播模式（fanout）
生产者的消息被所有消费者处理
现在的形式是通过了一个 exchange 整合了要使用的队列以及消费者。现在表示为广播模式，也就是说一个主题信息可以同时 被多个消费者同时消费处理。

- 直连模式
 RoutingKey的使用，相当于每个人的电话号码。
 指定一个具有指定routingkey的消费者处理。

 RoutingKey和fanout的区别，RoutingKey是唯一标记，准确送到消费者手中。

 - 主题模式
 相当于广播，整个exchange的routingkey信息相当于实现了所有连接模式。

 类似于广播和直连的整合，所有消费者都可以接收主题信息，但要如果进行正确处理，则一定需要合适的routingkey
