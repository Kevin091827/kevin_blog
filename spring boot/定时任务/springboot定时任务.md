# springboot定时任务

## 1.Timer和Quartz的缺陷

- jdk自带的Timer定时器，虽然简单轻量容易使用，但是Timer是单线程的，无法管理多线程环境下的定时任务调度，同时一时间只能有一个任务在执行

- Quatz：Quartz的使用相当广泛，它是一个功能强大的调度器，但是配置复杂

## 2.spring自带的定时器

Spring自带的定时任务Schedule，其实可以把它看作是一个简化版的，轻量级的Quartz，使用起来也相对方便很多。

## 3.使用方法

创建定时任务类

```java
@Component
@Slf4j
public class Task {

//    cron表达式
//    秒，分，时，日，月，星期
//    eg：(cron = 0/5 )
//
    @Scheduled(cron = "0/5 * * * * ?")
//  @Scheduled(fixedRate = 5000) 每隔5秒执行一次定时任务
//  @Scheduled(fixedRate = 5000) ：上一次开始执行时间点之后5秒再执行
//  @Scheduled(fixedDelay = 5000) ：上一次执行完毕时间点之后5秒再执行
//  @Scheduled(initialDelay=1000, fixedRate=5000) ：第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
    public void timeTask(){
        log.info("定时任务："+new Date());
    }

}
```
开启定时任务
```java
@SpringBootApplication
@EnableScheduling //开启定时任务
public class SpringbootMqApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootMqApplication.class, args);
    }

}
```

### 重点；

#### cron表达式

##### 1.简介
cron表达式一个字符串形式的表达式，又6到7个域组成，每个域以空格分开并且一个域代表一个含义

##### 2.使用详解

**域**

- 秒：可出现", - * /"四个字符，有效范围为0-59的整数  

- 分：可出现", - * /"四个字符，有效范围为0-59的整数  

- 时：可出现", - * /"四个字符，有效范围为0-23的整数  

- 日：可出现", - * / ? L W C"八个字符，有效范围为0-31的整数  

- 月：可出现", - * /"四个字符，有效范围为1-12的整数或JAN-DEc  

- 星期：可出现", - * / ? L C #"四个字符，有效范围为1-7的整数或SUN-SAT两个范围。1表示星期天，2表示星期一， 依次类推

**字符含义**

```application
* : 表示匹配该域的任意值，比如在秒*, 就表示每秒都会触发事件。；

? : 只能用在每月第几天和星期两个域。表示不指定值，当2个子表达式其中之一被指定了值以后，为了避免冲突，需要将另一个子表达式的值设为“?”；

- : 表示范围，例如在分域使用5-20，表示从5分到20分钟每分钟触发一次  

/ : 表示起始时间开始触发，然后每隔固定时间触发一次，例如在分域使用5/20,则意味着5分，25分，45分，分别触发一次.  

, : 表示列出枚举值。例如：在分域使用5,20，则意味着在5和20分时触发一次。  

L : 表示最后，只能出现在星期和每月第几天域，如果在星期域使用1L,意味着在最后的一个星期日触发。  

W : 表示有效工作日(周一到周五),只能出现在每月第几日域，系统将在离指定日期的最近的有效工作日触发事件。注意一点，W的最近寻找不会跨过月份  

LW : 这两个字符可以连用，表示在某个月最后一个工作日，即最后一个星期五。 

# : 用于确定每个月第几个星期几，只能出现在每月第几天域。例如在1#3，表示某月的第三个星期日。

```
**spring官方例子**
```xml
    "0 0 * * * *"                    表示每小时0分0秒执行一次

    " */10 * * * * *"                表示每10秒执行一次

    "0 0 8-10 * * *"                 表示每天8，9，10点执行

    "0 0/30 8-10 * * *"              表示每天8点到10点，每半小时执行

    "0 0 9-17 * * MON-FRI"           表示每周一至周五，9点到17点的0分0秒执行

    "0 0 0 25 12 ?"                  表示每年圣诞节（12月25日）0时0分0秒执行
```
上述整一个过程下来后，简单的定时任务就搭建好了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506150046388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

但是，spring自带的定时器默认情况下还是单线程的

**源码**
```java
//    springboot的定时任务默认是单线程的，源码如下：
    if (this.taskScheduler == null) {
        this.localExecutor = Executors.newSingleThreadScheduledExecutor();
        this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
    }
    //  定时任务的线程池大小只有1，即单线程线程池
      public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
        return new Executors.DelegatedScheduledExecutorService
                (new ScheduledThreadPoolExecutor(1));
      }
```
所以我们要更改定时任务的配置

具体做法只需要新建一个定时任务配置类，实现SchedulingConfigurer接口，重新接口方法设置线程池线程数量即可
```java
@Configuration
public class ScheduleConfig implements SchedulingConfigurer {

    //所有的定时任务都放在一个线程池中，定时任务启动时使用不同都线程。
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        //springboot定时任务默认是单线程，现在将线程池改为10个线程的线程池
        //设定一个长度10的定时任务线程池
        taskRegistrar.setScheduler(Executors.newScheduledThreadPool(10));
    }

//    springboot的定时任务默认是单线程的，源码如下：
//    if (this.taskScheduler == null) {
//        this.localExecutor = Executors.newSingleThreadScheduledExecutor();
//        this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
//    }
//      定时任务的线程池大小只有1，即单线程线程池
//      public static ScheduledExecutorService newSingleThreadScheduledExecutor() {
//        return new Executors.DelegatedScheduledExecutorService
//                (new ScheduledThreadPoolExecutor(1));
//      }

}

```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506143558810.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

如果不想每次一更改执行频率就重启程序，还可以这样玩
```java
    /**
     * 静态定时任务每次更改执行频率，就得重启程序
     * 动态定时任务将执行频率放于数据库中，只需更改数据库数据就可
     * 配置动态定时任务
     * @param taskRegistrar
     */
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addTriggerTask(
                
                //1.添加任务内容(Runnable)
                () -> System.out.println("执行动态定时任务: " + LocalDateTime.now().toLocalTime()),
                //2.设置执行周期(Trigger)
                triggerContext -> {
                    //2.1 从数据库获取执行周期
//                    String cron = cronMapper.getCron();
//                    //2.2 合法性校验.
//                    if (StringUtils.isEmpty(cron)) {
//                        // Omitted Code ..
//                    }
//                    //2.3 返回执行周期(Date)
//                    return new CronTrigger(cron).nextExecutionTime(triggerContext);
                }
        );
    }
```
