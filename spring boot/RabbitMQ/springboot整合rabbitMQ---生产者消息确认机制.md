#### rabbitMQ消息确认机制

当我们程序向rabbitMQ中间件发送消息时，如果程序没什么异常的话，一般都会成功发送消息
但是，我们并不知道，消息是否成功发送到相应交换机的相应队列中，此时，我们需要用到消息确认机制，
这也是rabbitMQ的一个功能点

**1.消息发送确认**

* 消息发送确认：
   
   当消息可能因为路由键不匹配或者发送不到指定交换机而导致无法发送到相应队列时
   确认消息发送失败，相反，确认消息发送成功


**2.两个接口**

**ConfirmCallback**

实现ConfirmCallback接口实现消息发送到交换机的回调

```java
@Slf4j
public class RabbitConfirmCallback implements RabbitTemplate.ConfirmCallback{
    /**
     * 发送到交换机上失败回调
     * 消息发送回调(判断是否发送到相应的交换机上)
     * @param correlationData
     * @param ack
     * @param cause
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

**ReturnCallback**

实现ReturnCallback接口实现消息失败回调，当消息路由不到指定队列时回调方法

```java

@Slf4j
public class RabbitReturnCallback implements RabbitTemplate.ReturnCallback {
    /**
     * 发送到队列失败后回调
     * 消息可以发送到相应交换机，但是没有相应路由键和队列绑定
     * @param message
     * @param i
     * @param s
     * @param s1
     * @param s2
     */
    @Override
    public void returnedMessage(Message message, int i, String s, String s1, String s2) {
        log.info("消息发送失败");
    }
}
```
最后只需要重新配置rabbitTemplate即可
```java
   /**
     * 定制rabbitMQ模板
     *
     * ConfirmCallback接口用于实现消息发送到RabbitMQ交换器后接收消息成功回调
     * ReturnCallback接口用于实现消息发送到RabbitMQ交换器，但无相应队列与交换器绑定时的回调  即消息发送不到任何一个队列中回调
     * @return
     */
    @PostConstruct
    public void initRabbitTemplate(){

        rabbitTemplate.setConfirmCallback(new RabbitConfirmCallback());
        rabbitTemplate.setReturnCallback(new RabbitReturnCallback());
    }
```
补充：

@PostConstruct和@PreConstruct。这两个注解被用来修饰一个非静态的void()方法.而且这个方法不能有抛出异常声明。

```java
    @PostConstruct                    //方式1
    public void someMethod(){
        ...
    }
    
    @PreConstruct 
    public void someMethod(){        //方式2
        ...  
    }
```
**@PostConstruct**

 被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。
 被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。
 
 **@PreConstruct**
 
 被@PreDestroy修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy修饰的方法会在destroy()方法之后运行，在Servlet被彻底卸载之前。
 
 #### 5.rabbitMQ消息持久化机制
 
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
项目所在github：https://github.com/Kevin091827/spring-boot-rabbitMq
