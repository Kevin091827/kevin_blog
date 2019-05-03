### 一，消息接收确认
##### 1.ACK机制：消息确认机制
**1.作用：** 

* 确认消息是否被消费者消费，消息通过ACK机制确认是否被正确接收，每个消息都要被确认。
* 默认情况下，一个消息被消费者正确消费就会从队列中移除

**2.ACK确认模式**

* **AcknowledgeMode.NONE ：不确认**

	**1**. 默认所有消息消费成功，会不断的向消费者推送消息
	**2**. 因为rabbitMq认为所有消息都被消费成功，所以队列中不在存有消息，消息存在丢失的危险
* **AcknowledgeMode.AUTO：自动确认**

	**1**. 由spring-rabbit依据消息处理逻辑是否抛出异常自动发送ack（无异常）或nack（异常）到server端。				      存在丢失消息的可能，如果消费端消费逻辑抛出异常，也就是消费端没有处理成功这条消息，那么就相当于丢失了消息
如果消息已经被处理，但后续代码抛出异常，使用 Spring 进行管理的话消费端业务逻辑会进行回滚，这也同样造成了实际意义的消息丢失

	**2**. 使用自动确认模式时，需要考虑的另一件事是消费者过载

* **AcknowledgeMode.MANUAL：手动确认**

	**1**. 手动确认则当消费者调用 ack、nack、reject 几种方法进行确认，手动确认可以在业务失败后进行一些操作，如果消息未被 ACK 则会发送到下一个消费者

	**2**. 手动确认模式可以使用 prefetch，限制通道上未完成的（“正在进行中的”）发送的数量

**3. 忘记ACK确认**

忘记通过basicAck返回确认信息是常见的错误。这个错误非常严重，将导致消费者客户端退出或者关闭后，消息会被退回RabbitMQ服务器，这会使RabbitMQ服务器内存爆满，而且RabbitMQ也不会主动删除这些被退回的消息。只要程序还在运行，没确认的消息就一直是 Unacked 状态，无法被 RabbitMQ 重新投递。更厉害的是，RabbitMQ 消息消费并没有超时机制，也就是说，程序不重启，消息就永远是 Unacked 状态。处理运维事件时不要忘了这些 Unacked 状态的消息。当程序关闭时（实际只要 消费者 关闭就行），消息会恢复为 Ready 状态。

##### 2.局部消息确认

**开启手动ack确认**

```application
#消息确认机制 --- 是否开启手ack动确认模式
spring.rabbitmq.listener.direct.acknowledge-mode=manual
#消息确认机制 --- 是否开启手ack动确认模式
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

**配置消费者**

```java
@RabbitHandler
    @RabbitListener(queues = DirectKeyInterface.DIRECT_QUEUE_NAME)
    public void receiveObjectDel(Channel channel, String json, Message message,@Headers Map<String,Object> map){

        log.info("接收到的id："+json);
        int id = Integer.parseInt(json);
        registerDao.deleteUser(id);

        //<P>代码为在消费者中开启消息接收确认的手动ack</p>
        //<H>配置完成</H>
        //<P>可以开启全局配置</p>
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
            log.info("消息消费失败：id：{}",message.getMessageProperties().getDeliveryTag());
        }
    }
```
##### 3.全局消息确认

```java
    /**
     * 消费者全局消息手动ACK确认(还没配置完成)
     * @return
     */
    @Bean
    public SimpleMessageListenerContainer messageListenerContainer(){

        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory());
        //监听的队列（是一个String类型的可变参数,将监听的队列配置上来，可减少在消费者中代码量）
        container.setQueueNames(DirectKeyInterface.DIRECT_QUEUE_NAME);
        //手动确认
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
        container.setMessageListener(channelAwareMessageListener());
    
        //消息处理
        container.setMessageListener((ChannelAwareMessageListener) (message, channel) -> {
            log.info("====接收到消息=====");
            log.info(new String(message.getBody()));
            //它会根据方法的执行情况来决定是否确认还是拒绝（是否重新入queue）
                //1.抛出NullPointerException异常则重新入队列
                    //throw new NullPointerException("消息消费失败");
                //2.当抛出的异常是AmqpRejectAndDontRequeueException异常的时候，则消息会被拒绝，且requeue=false
                    //throw new AmqpRejectAndDontRequeueException("消息消费失败");
                //3.当抛出ImmediateAcknowledgeAmqpException异常，则消费者会被确认
                    //throw new ImmediateAcknowledgeAmqpException("消息消费失败");
            //消息手动弄ACK确认
            if(message.getMessageProperties().getHeaders().get("error") == null){

                try {
                    //手动ack应答
                    //告诉服务器收到这条消息 已经被我消费了 可以在队列删掉 这样以后就不会再发了
                    // 否则消息服务器以为这条消息没处理掉 后续还会在发，true确认所有消费者获得的消息
                    channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
                    log.info("消息消费成功：id：{}",message.getMessageProperties().getDeliveryTag());
                } catch (IOException e) {
                    e.printStackTrace();
                    //丢弃这条消息
                    try {
                        //最后一个参数是：是否重回队列
                        channel.basicNack(message.getMessageProperties().getDeliveryTag(), false,true);
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
                    log.info("消息消费失败：id：{}",message.getMessageProperties().getDeliveryTag());
                }
            }else {
                //处理错误消息，拒觉错误消息重新入队
                channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,false);
                channel.basicReject(message.getMessageProperties().getDeliveryTag(),false);
                log.info("消息拒绝");
            }
        });

        return container;
    }
```