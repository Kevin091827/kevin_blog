# quartz中misfirm处理机制

## 一，为什么需要misfirm处理机制？

在利用quartz做任务调度时，当在多任务的情况下，我们有时候很难保证每一个任务都能在准确时间准确的执行

比如：

- 线程池中线程数量不足导致某些任务没有可用线程执行从而错过执行时间，导致任务激活失败（misfirm）

- 又或者任务暂停后重新恢复执行，从暂停到恢复执行这一段时间错过的任务，该如何处理，这也是misfirm考虑处理的问题

所以，misfirm机制更主要就是为了让我们处理任务与任务，任务与当前实际情况的之间的冲突

## 二，misfirm处理机制详解

### 如何判断激活失败？

在quartz.properties配置文件中有一个属性是misfireThreshold（单位为毫秒），用来指定调度引擎设置触发器超时的"临界值"。也就是说Quartz对于任务的超时是有容忍度的，超过了这个容忍度才会判定misfire。比如说，某触发器设置为，10:15首次激活，然后每隔3秒激活一次，无限次重复。然而该任务每次运行需要10秒钟的时间。可见，每次任务的执行都会超时，那么究竟是否会引起misfire，就取决于misfireThreshold的值了。以第二次任务来说，它的运行时间已经比预定晚了7秒，那么如果misfireThreshold>7000，说明该偏差可容忍，则不算misfire，该任务立刻执行；如果misfireThreshold<=7000，则判定为misfire，根据相关配置策略进行处理。

如果需要修改quartz.properties配置文件修改默认配置的话，基于springboot项目可以在Resource文件夹下新建quartz.properties文件，指定自己的配置即可

```properties
######################## 调度器设置 ###########################
org.quartz.scheduler.instanceName: DefaultQuartzScheduler
org.quartz.scheduler.rmi.export: false
org.quartz.scheduler.rmi.proxy: false
org.quartz.scheduler.wrapJobExecutionInUserTransaction: false

######################## 线程池设置 ###########################
org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true

######################## job工作设置 ###########################
##这个时间大于10000（10秒）会导致MISFIRE_INSTRUCTION_DO_NOTHING不起作用。
org.quartz.jobStore.misfireThreshold: 5000
org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```

### 详解misfirm机制

根据触发器类型分为两种misfirm处理机制

#### 1.SimpleTrigger的misfire机制（可以去参考网上资料，用的不多）

```java
SimpleScheduleBuilder ssb = SimpleScheduleBuilder.simpleSchedule();

ssb.withMisfireHandlingInstructionFireNow();//1
ssb.withMisfireHandlingInstructionIgnoreMisfires();//2
ssb.withMisfireHandlingInstructionNextWithExistingCount();//3
ssb.withMisfireHandlingInstructionNextWithRemainingCount();//4
ssb.withMisfireHandlingInstructionNowWithExistingCount();//5
ssb.withMisfireHandlingInstructionNowWithRemainingCount();//6
```
*  withMisfireHandlingInstructionFireNow 

*  withMisfireHandlingInstructionIgnoreMisfires

* withMisfireHandlingInstructionNextWithExistingCount


* withMisfireHandlingInstructionNextWithRemainingCount  

* withMisfireHandlingInstructionNowWithExistingCount

* withMisfireHandlingInstructionNowWithRemainingCount



#### 2.CronTrigger的misfire机制（重点）

主要有这3种：
```java
        //创建cronScheduleBuilder
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder
                .cronSchedule(cron)
                
                //misfirm机制
                .withMisfireHandlingInstructionFireAndProceed()//1
                .withMisfireHandlingInstructionFireAndProceed()//2
                .withMisfireHandlingInstructionDoNothing();//3

        //创建trigger
        CronTrigger cronTrigger = (CronTrigger) TriggerBuilder.newTrigger()
                .withIdentity(triggerName, triggerGroup)
                .withSchedule(cronScheduleBuilder)
                .build();
```

##### withMisfireHandlingInstructionDoNothing

* 不触发立即执行，前面错过的激活失败的不重做
* 等待下次Cron触发频率到达时刻开始按照Cron频率依次执行

##### withMisfireHandlingInstructionFireAndProceed

* 以当前时间为触发频率立刻触发一次执行
* 然后按照Cron频率依次执行

##### withMisfireHandlingInstructionIgnoreMisfires

* 以错过的第一个频率时间立刻开始执行，重做错过的任务
* 当下一次触发频率发生时间大于当前时间后，再按照正常的Cron频率依次执行


