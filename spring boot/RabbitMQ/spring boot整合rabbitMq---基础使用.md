#### 1. AMQP协议

**1.简介**

AMQP（Advanced Message Queuing Protocol，高级消息队列协议）是一个进程间传递异步消息的l网络协议

**2.AMQP模型**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411002317206.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**3.工作流程**

消息发布者发布消息给交换机，交换机根据传递过来的路由键按照路由规则将接收到的消息分发给和交换机绑定的相应的队列，最后交给消息消费者


#### 2. RabbitMQ

**1.简介**

RabbitMQ是实现AMQP（高级消息队列协议）的消息中间件的一种

**2.工作流程**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411001941123.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**相关概念**

* 虚拟主机：

	一个虚拟主机持有医嘱交换机，队列和绑定，rabbitMQ中一般建议有多个虚拟主机，为什么需要多个虚拟主机呢？很简单，rabbitMQ中，用户只能在虚拟主机的粒度进行权限控制，因此，如果需要禁止A组访问B组的交换机/队列/绑定，必须为A组合B租分别创建一个虚拟主机，默认虚拟主机“/“。
* 交换机：

	作用：转发消息，不会存储消息，如果没有队列绑定到交换机，交换机会直接丢弃生产者发来时的消息

* 路由键：

	消息发送到交换机的时候，交换机会根据路由键转发到对应的队列中

* 绑定：

	交换机和队列相绑定，一个交换机可绑定多个队列


**生产消费模型**

direct----匹配规则为：如果路由键匹配，消息就被投送到相关的队列

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019041100352220.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

fanout----交换器中没有路由键的概念，他会把消息发送到所有绑定在此交换器上面的队列中
。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411003531402.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

topic----交换器你采用模糊匹配路由键的原则进行转发消息到队列中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190411003541136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**3.使用场景**

RabbitMQ 即一个消息队列，主要是用来实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用。

#### 3.springboot 整合RabbitMQ

**1.引入依赖**

springboot集成RabbitMQ非常简单，如果只是简单的使用配置非常少，springboot提供了spring-boot-starter-amqp项目对消息各种支持。
```xml
	    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
```

**2.配置端口，账户信息**

```application
############  RabbitMQ  #################
#spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
#开启发布确认机制
spring.rabbitmq.publisher-confirms=true
#是否创建AmqpAdmin bean. 默认为: true
spring.rabbitmq.dynamic=true
spring.rabbitmq.cache.connection.mode=channel
#指定最小的消费者数量.
#spring.rabbitmq.listener.concurrency
#指定最大的消费者数量
#spring.rabbitmq.listener.max-concurrency
```
**3.设置队列，交换机**

```java
    /**
     * 设置队列
     * @return
     */
    @Bean
    public Queue queue(){
        return new Queue(DirectKeyInterface.DIRECT_QUEUE_NAME);
    }
    
    /**	
     * 设置交换机
     * @return
     */
    @Bean
    public DirectExchange directExchange(){
        return new DirectExchange(DirectKeyInterface.DIRECT_EXCHANGE_NAME);
    }
```
**4.绑定队列，交换机和路由键**

如果rabbitMQ管理平台中还没有，则会自动新增

```java
    /**
     * 在fanout模式下绑定交换机，队列，路由键
     * @param directExchange
     * @param queue
     * @return
     */
    @Bean
    public Binding binding_direct(DirectExchange directExchange,Queue queue){
        return BindingBuilder.bind(queue).to(directExchange).with(DirectKeyInterface.DIRECT_KEY);
    }
```
**5.生产者，消费者**

生产者 --- 消息发送
```java
    /**
     * 消息发送(点对点）
     * @param message
     */
    public void send(String message){
       amqpTemplate.convertAndSend(DirectKeyInterface.DIRECT_EXCHANGE_NAME,DirectKeyInterface.DIRECT_KEY,message);
       log.info("消息发送成功：{}",message);
    }
```

消费者 --- 接收消息
```java
    /**
     * 接收消息（自动监听）
     * @param message
     */
    @RabbitHandler
    @RabbitListener(queues = DirectKeyInterface.DIRECT_QUEUE_NAME)//监听指定队列
    public void recevice(String message){
        log.info("接收到消息：{}",message);
    }
```
**6.测试**
```java
    @Autowired
    private Producer producer;

    /**
     * 发送消息
     * @return
     */
    @GetMapping("/helloRabbitMQ")
    public String sendMsg(){
        producer.send("hello world!");
        //producer.sendOfFanout("hello!");
        //producer.sendOfTopic("hello topic!");
        return "success";
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190413013404781.png)

**补充：**

如果是同一个队列多个消费类型那么就需要针对每种类型提供一个消费方法，否则找不到匹配的方法会报错，如下：
```java
@Component
@Slf4j
@RabbitListener(
    bindings = @QueueBinding(
        exchange = @Exchange(value = RabbitMQConstant.MULTIPART_HANDLE_EXCHANGE, type = ExchangeTypes.TOPIC,
            durable = RabbitMQConstant.FALSE_CONSTANT, autoDelete = RabbitMQConstant.true_CONSTANT),
        value = @Queue(value = RabbitMQConstant.MULTIPART_HANDLE_QUEUE, durable = RabbitMQConstant.FALSE_CONSTANT,
            autoDelete = RabbitMQConstant.true_CONSTANT),
        key = RabbitMQConstant.MULTIPART_HANDLE_KEY
    )
)
@Profile(SpringConstant.MULTIPART_PROFILE)
public class MultipartConsumer {

    /**
     * RabbitHandler用于有多个方法时但是参数类型不能一样，否则会报错
     *
     * @param msg
     */
    @RabbitHandler
    public void process(ExampleEvent msg) {
        log.info("param:{msg = [" + msg + "]} info:");
    }

    @RabbitHandler
    public void processMessage2(ExampleEvent2 msg) {
        log.info("param:{msg2 = [" + msg + "]} info:");
    }

    /**
     * 下面的多个消费者，消费的类型不一样没事，不会被调用，但是如果缺了相应消息的处理Handler则会报错
     *
     * @param msg
     */
    @RabbitHandler
    public void processMessage3(ExampleEvent3 msg) {
        log.info("param:{msg3 = [" + msg + "]} info:");
    }


}
```
注解将消息和消息头注入消费者方法

在上面也看到了@Payload等注解用于注入消息。这些注解有：

* @Header 注入消息头的单个属性
* @Payload 注入消息体到一个JavaBean中
* @Headers 注入所有消息头到一个Map中

这里有一点主要注意，如果是`com.rabbitmq.client.Channel,org.springframework.amqp.core.Message`和`org.springframework.messaging.Message`这些类型，可以不加注解，直接可以注入。

如果不是这些类型，那么不加注解的参数将会被当做消息体。不能多于一个消息体。如下方法ExampleEvent就是默认的消息体：

    public void process2(@Headers Map<String, Object> headers,ExampleEvent msg);

项目所在github：https://github.com/Kevin091827/spring-boot-rabbitMq