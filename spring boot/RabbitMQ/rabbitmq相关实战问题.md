# 一，rabbitMQ消息持久化机制

为了保证消息的可靠性，需要对消息进行持久化。 
为了保证RabbitMQ在重启、奔溃等异常情况下数据没有丢失，除了对消息本身持久化为，还需要将消息传输经过的队列(queue)，交互机进行持久化(exchange)，持久化以上元素后，消息才算真正RabbitMQ重启不会丢失。

>详细参数：

>durable :是否持久化，如果true，则此种队列叫持久化队列（Durable queues）。此队列会被存储在磁盘上，当消息代理（broker）重启的时候，它依旧存在。没有被持久化的队列称作暂存队列（Transient queues）。 
 
>execulusive :表示此对应只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable 

>autoDelete: 
 当没有生成者/消费者使用此队列时，此队列会被自动删除。 
 (即当最后一个消费者退订后即被删除)
 
 eg:
 ```java
  /**
     * 设置持久化topic模式队列
     * durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
     * exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
     * autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
     * @return
     */
    @Bean
    public Queue durableTopicQueue(){
        return new Queue(TopicKeyInterface.TOPIC_DURABLE_QUEUE_NAME,true,true,false);
    }
    
     /**
         * 设置持久化交换机
         * durable:
         * autoDelete:
         * @return
         */
        @Bean
        public TopicExchange durableTopicExchange(){
            return new TopicExchange(TopicKeyInterface.TOPIC_DURABLE_QUEUE_NAME,true,false);
        }
```

# 二，rabbitMQ消息确认机制

问题 
> 生产者将消息发送出去后，消息到底有没有到达相应交换机，到达相应队列呢？rabbitmq默认是不知道的，所以我们在发送消息后，默认是不知道消息是否真正发送成功的，因此我们需要一个机制来确认消息是否真正发送成功

消息确认有两种方式：

* 1.AMQP协议的事务机制
* 2.rabbitMQ的confirm机制

## 1.rabbitMQ事务机制
RabbitMQ 支持消息发送的“事务”，且仅支持于“单个队列”的操作（生产者的消息发送、消费者的消息确认）。

```java

//开启事务
channel.txSelect();

try{

    //发送消息

    //提交事务
    channel.txCommit();
}catch(Exception e){
    //事务回滚,丢弃事务中的全部消息
    channel.txRollback();
}
//txCommit 和 txRollback 执行完成，将开启新的事务。
```
**总结：**

事务确实能够解决producer与broker之间消息确认的问题，只有消息成功被broker接受，事务提交才能成功，否则我们便可以在捕获异常进行事务回滚操作同时进行消息重发，但是使用事务机制的话会降低RabbitMQ的性能和消息吞吐量
(基于协议)

## 2.confirm机制

消息的确认，是指生产者投递消息后，如果Broker收到消息，则会给我们生产者一个应答。生产者进行接收应答，用来确定这条消息是否正常的发送到Broker，这种方式也是消息的可靠性投递的核心保障！

生产端发送消息到Broker，然后Broker接收到了消息后，进行回送响应，生产端有一个Confirm Listener，去监听应答，当然这个操作是异步进行的，生产端将消息发送出去就可以不用管了，让内部监听器去监听Broker给我们的响应。

原理图：

![](https://upload-images.jianshu.io/upload_images/14795543-a1fa4d9c45bc445a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/620/format/webp)

```java
//开启confirm模式
channel.confirmSelect();

//发送消息
 channel.basicPublish(exchangeName, routingKey, null, msg.getBytes());

//channel.addConfirmListener(new ConfirmListener() {
            //消息失败处理
            @Override
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                //deliveryTag；唯一消息标签(消息id)
                //multiple：是否批量
                System.err.println("-------no ack!-----------");
            }
            //消息成功处理
            @Override
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                System.err.println("-------ack!-----------");
            }
        });
```

## 3.springboot中的手动ACK机制

在springboot中使用rabbitmq设置ack主要分为以下三步：

* 设置ack确认模式
* 生产者消息发送确认
* ack设置

**1.设置ack确认模式**
```java
######消息确认机制配置
#消息确认机制 --- 消息发送回调(默认false）
spring.rabbitmq.publisher-confirms=true
#消息确认机制 --- 消息发送失败返回回调（默认false）
spring.rabbitmq.publisher-returns=true
#消息确认机制 --- 是否开启手动ack确认模式
spring.rabbitmq.listener.direct.acknowledge-mode=manual
#消息确认机制 --- 是否开启手动ack确认模式
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

**2.生产者消息ack发送确认**

判断消息是否从生产者发送到交换机上

```java
@Slf4j
public class RabbitConfirmCallback implements RabbitTemplate.ConfirmCallback{

    /**
     * 发送到交换机上失败回调
     * 消息发送回调(判断是否发送到相应的交换机上)
     * @param correlationData   消息唯一标识
     * @param ack               消息确认结果
     * @param cause             失败原因
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            log.info("消息发送到exchange成功");
        } else {
            log.info("消息发送到exchange失败");
        }
    }
}
```

判断消息是否路由到指定队列
```java
@Slf4j
public class RabbitReturnCallback implements RabbitTemplate.ReturnCallback {

    /**
     * 发送到队列失败后回调
     * 消息可以发送到相应交换机，但是没有相应路由键和队列绑定
     * @param message   返回消息
     * @param i         返回状态码
     * @param s         回复文本
     * @param s1        交换机
     * @param s2        路由键
     */
    @Override
    public void returnedMessage(Message message, int i, String s, String s1, String s2) {
        log.info("消息发送失败");
    }
}

```


**3.ack消息接收**

确认消息是否被正确消费

```java
@Slf4j
public class AckUtils {

    public static void ack(Channel channel, Message message,Map<String,Object> map){
        if (map.get("error")!= null){
            log.info("错误的消息");
            try {
                //否认消息,拒接该消息重回队列
                channel.basicNack((Long)map.get(AmqpHeaders.DELIVERY_TAG),false,false);
                return;
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        //手动ACK
        //默认情况下如果一个消息被消费者所正确接收则会被从队列中移除
        //如果一个队列没被任何消费者订阅，那么这个队列中的消息会被 Cache（缓存），
        //当有消费者订阅时则会立即发送，当消息被消费者正确接收时，就会被从队列中移除
        try {
            //手动ack应答
            //告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了
            // 否则消息服务器以为这条消息没处理掉 后续还会在发，true确认所有消费者获得的消息
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
            log.info("消息消费成功：id：{}",message.getMessageProperties().getDeliveryTag());
        } catch (IOException e) {
            e.printStackTrace();
            log.info("消息消费失败：id：{}",message.getMessageProperties().getDeliveryTag());
            //丢弃这条消息
            try {
                //最后一个参数是：是否重回队列
                channel.basicNack(message.getMessageProperties().getDeliveryTag(), false,false);
                //拒绝消息
                //channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
                //消息被丢失
                //channel.basicReject(message.getMessageProperties().getDeliveryTag(), false);
                //消息被重新发送
                //channel.basicReject(message.getMessageProperties().getDeliveryTag(), true);
                //多条消息被重新发送
                //channel.basicNack(message.getMessageProperties().getDeliveryTag(), true, true);
            } catch (IOException e1) {
                e1.printStackTrace();
            }
        }
    }
}

```

>忘记ack确认会导致什么问题?

忘记通过basicAck返回确认信息是常见的错误。这个错误非常严重，将导致消费者客户端退出或者关闭后，消息会被退回RabbitMQ服务器，这会使RabbitMQ服务器内存爆满，而且RabbitMQ也不会主动删除这些被退回的消息。只要程序还在运行，没确认的消息就一直是 Unacked 状态，无法被 RabbitMQ 重新投递。更厉害的是，RabbitMQ 消息消费并没有超时机制，也就是说，程序不重启，消息就永远是 Unacked 状态。处理运维事件时不要忘了这些 Unacked 状态的消息。当程序关闭时（实际只要 消费者 关闭就行），消息会恢复为 Ready 状态。


##  三，Exchange 类型的 topic 与 header 区别？

**共同点：**

- 两者都是基于特定规则（路由键）将消息路由到特定的队列

**区别：**

- topic是基于和队列绑定的路由键采取模糊比对的方式进行路由

- header是基于消息的 header 和绑定的 header 中，K-V 的匹配，并且支持 all 和 any（由绑定的 header 的 x-match 属性确认）





