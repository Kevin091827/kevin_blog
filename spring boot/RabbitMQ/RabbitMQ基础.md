#### RabbitMQ

##### 1.简介：
RabbitMQ是一个实现了AMQP协议（Advanced Message Queue Protocol）的消息队列。
##### 2.作用：
   * **流量削峰**：
当我们在秒杀特价商品时，系统在一小段时间内突然收到大量的订单，如果将这么多订单请求写入数据库的话，数据库的负荷会非常之大，然而我们的数据库层也是最脆弱的，但是有了消息队列后，我们可以先将订单请求写入消息队列中，消息队列是位于我们程序和数据库之间的中间件，待订单请求入队后，在根据队列的先进先出特性，依次写入数据库，所以消息队列在本质上起到的作用就是削峰填谷，为业务保驾护航。

   * **降低系统耦合度**：
消息队列的主要特点是异步处理，主要目的是减少请求响应时间和解耦。所以主要的使用场景就是将比较耗时而且不需要即时（同步）返回结果的操作作为消息放入消息队列。同时由于使用了消息队列，只要保证消息格式不变，消息的发送方和接收方并不需要彼此联系，也不需要受对方的影响，即解耦合
   * **提高系统吞吐量，性能**


rabbitMQ的应用场景：


1.异步处理

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417204016307.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

2.应用解耦：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417204101623.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

3.流量削峰

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019041720411097.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 3.概念

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005509289.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

  * **生产者**

    创建消息，然后发送到代理服务器（RabbitMQ）的程序

  * **交换机**

    交换机是生产者和队列之间的中间件，消息经生产者发送到达交换机，交换机根据指定的路由规则分发给相应队列

  * **路由键**

    分发规则
  * **队列**

    存放消息的地方
  * **消费者**

    连接到代理服务器（RabbitMQ），并订阅到队列上接收消息

##### 4.工作模式：

   * **1.direct**

direct是rabbitMQ默认的工作模式，即Direct Exchange 是 RabbitMQ 默认的 Exchange，完全根据路由键来路由消息，只有完全匹配时，该消息才会路由到指定队列。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005733879.gif)

   * **2.fanout**

   该模式下会忽略路由键规则，直接将消息广播到该交换机绑定的所有队列，用于全体消息订阅服务

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005207284.gif)

   * **3.topic**

该模式和direct模式相似，但是该模式路由键是模糊匹配，分别支持*和#通配符，*表示匹配一个单词，#则表示匹配没有或者多个单词。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005706925.gif)

   * 4.header

Headers Exchange 会忽略 RoutingKey 而根据消息中的 Headers 和创建绑定关系时指定的 Arguments 来匹配决定路由到哪些 Queue。Headers Exchange 的性能比较差，而且 Direct Exchange 完全可以代替它，所以不建议使用。

##### 5.工作流程：

**单个队列：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005627185.gif)

**多个队列：**

多个队列情况下会比较消息的路由键，按照相应规则分发到相应队列中，

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190417005939307.gif)

routing key命名规则：用"."分割的字母或数字

匹配规则：

*：匹配单个字母或数字

#：匹配0~多个字母或数字