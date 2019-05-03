### 一，延迟队列

##### 应用场景

- **延迟消费**:

比如： 用户生成订单之后，需要过一段时间校验订单的支付状态，如果订单仍未支付则需要及时地关闭订单。

用户注册成功之后，需要过一段时间比如一周后校验用户的使用情况，如果发现用户活跃度较低，则发送邮件或者短信来提醒用户使用。

- **延迟重试**：

比如消费者从队列里消费消息时失败了，但是想要延迟一段时间后自动重试。

如果不使用延迟队列，那么我们只能通过一个轮询扫描程序去完成。这种方案既不优雅，也不方便做成统一的服务便于开发人员使用。但是使用延迟队列的话，我们就可以轻而易举地完成。

### 二，springboot实现rabbitMQ延时队列
#### 实现思路
###### RabbitMQ两大特性

**RabbitMQ消息的死亡方式：**

- 消息被拒绝，通过调用basic.reject或者basic.nack并且设置的requeue参数为false。
- 消息设置了存活时间
- 消息进入了一条已经达到最大长度的队列

**Time-To-Live Extensions**

rabbitmq允许我们为消息或者队列设置过期时间，也就是TTL，TTL的意思是一条消息在队列中最大的存活时间，单位是毫秒，当某条消息被设置了TTL或者当某条消息进入了设置了TTL的队列时，这条消息会在经过TTL秒后“死亡”，成为Dead Letter。如果既配置了消息的TTL，又配置了队列的TTL，那么较小的那个值会被取用。

**Dead Letter Exchange**

如果队列设置了Dead Letter Exchange（DLX），那么这些Dead Letter就会被重新publish到Dead Letter Exchange，通过Dead Letter Exchange路由到其他队列

##### 实现流程

* **延迟消费**

生产者产生的消息首先会进入缓冲队列（图中红色队列）。通过RabbitMQ提供的TTL扩展，这些消息会被设置过期时间，也就是延迟消费的时间。等消息过期之后，这些消息会通过配置好的DLX转发到实际消费队列（图中蓝色队列），以此达到延迟消费的效果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190419213745548.png)

* **延迟重试**

消费者发现该消息处理出现了异常，比如是因为网络波动引起的异常。那么如果不等待一段时间，直接就重试的话，很可能会导致在这期间内一直无法成功，造成一定的资源浪费。那么我们可以将其先放在缓冲队列中（图中红色队列），等消息经过一段的延迟时间后再次进入实际消费队列中（图中蓝色队列），此时由于已经过了“较长”的时间了，异常的一些波动通常已经恢复，这些消息可以被正常地消费。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190419213834279.png)

##### springboot实现rabbitMQ延迟消费

延迟消费相关配置

```java
@Configuration
public class DelayConfig {

    /**
     * 延时交换机 --- 交换机用于重新分配队列（接收死信队列中的过期消息，将其转发到需要延迟消息的模块队列）
     * @return
     */
    @Bean
    public DirectExchange exchange() {
        return new DirectExchange(DelayKeyInterface.DELAY_EXCHANGE);
    }

    /**
     * 实际消费队列
     * 用于延时消费的队列
     */
    @Bean
    public Queue repeatTradeQueue() {
        Queue queue = new Queue(DelayKeyInterface.DELAYMSG_RECEIVE_QUEUE_NAME,true,false,false);
        return queue;
    }

    /**
     * 绑定交换机并指定routing key（死信队列绑定延迟交换机和实际消费队列绑定延迟交换机的路由键一致）
     * @return
     */
    @Bean
    public Binding  repeatTradeBinding() {
        return BindingBuilder.bind(repeatTradeQueue()).to(exchange()).with(DelayKeyInterface.DELAY_KEY);
    }

    //死信队列
    @Bean
    public Queue deadLetterQueue() {
        Map<String,Object> args = new HashMap<>();
        args.put("x-message-ttl", DelayKeyInterface.EXPERI_TIME);
        args.put("x-dead-letter-exchange", DelayKeyInterface.DELAY_EXCHANGE);
        args.put("x-dead-letter-routing-key", DelayKeyInterface.DELAY_KEY);
        return new Queue(DelayKeyInterface.DELAY_QUEUE_NAME, true, false, false, args);
    }

}

```

发送消息
```java
    /**
     * 发送延迟消息
     * @return
     */
    @GetMapping("/send")
    public String sendDelayMsg(){
        rabbitTemplate.convertAndSend(DelayKeyInterface.DELAY_QUEUE_NAME,"hello");
        log.info("发送时间："+ LocalDateTime.now());
        return "success";
    }
```
接收延迟消息：
```java
    /**
     * 接收延迟消息
     * @param channel
     * @param json
     * @param message
     * @param map
     */
    @RabbitHandler
    @RabbitListener(queues = DelayKeyInterface.DELAYMSG_RECEIVE_QUEUE_NAME)
    public void receiveDelayMsg(Channel channel, String json, Message message,@Headers Map<String,Object> map){

        log.info("接收到的消息"+json);
        log.info("接收时间："+ LocalDateTime.now());
        //<P>代码为在消费者中开启消息接收确认的手动ack</p>
        //<H>配置完成</H>
        //<P>可以开启全局配置</p>
        AckUtils.ack(channel,message,map);
    }
```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190420011059332.png)


