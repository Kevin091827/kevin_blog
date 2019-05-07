# springboot集成quartz

## 一，quartz基本概念

### 1.Job

#### Job，JobDetail，JobBuilder，JobExecutionContext

>Job？

job英文单词的意思就是工作，意味着你需要调度器具体要做的事情，job是一个工作任务调度的接口，具体要被调度的任务要实现该接口，重写该接口定义的execute方法（具体任务）编写任务的业务逻辑。

job实例在Quartz中的生命周期：每次调度器执行job时，他在调用execute方法前都会创建一个job实例，调用方法完成后，关联的job实例被释放，然后垃圾回收。（即每次job的构造方法都会被执行）

>jobDetail？

通过传入job类型，JobBuilder会产生jobDetail实例，jobDetail存储着该类型的job的一些实例信息，调度器需要借助jobDetail来添加job实例

重要信息：
- name:任务名称
- group：任务组名称（会有默认组名）
- jobDataMap

>JobExecutionContext?

调度器在调用一个job的execute方法时，会将JobExecutionContext传递给该方法，job通过该JobExecutionContext对象可以访问到Quartz运行时候得环境和job本身的明细数据（job带有的信息）

>JobBuilder？

jobDetail生成器

#### 补充

- @DisallowConcurrentExecution：相同的jobDetail不能并发执行，不同的jobDetail可以并发执行，

- @PersistJobDataAfterExecution：将该注解加在job类上，告诉Quartz在成功执行了job类的execute方法后（没有发生任何异常），更新JobDetail中JobDataMap的数据，使得该job（即JobDetail）在下一次执行的时候，JobDataMap中是更新后的数据，而不是更新前的旧数据

示例：
```java
        //由job类生产jobDetail
        JobDetail jobDetail = newJob(TestJob.class) //产生Job实例，job信息存储在jobDetail中
                .usingJobData("key1","val1")//通过jobDataMap为job实例增加属性，然后再传递到调度器中去
                .usingJobData("key2","val2")
                .withIdentity("helloTask","task1")
                .build();//由jobBuilder生成jobDetail
```

### 2.Trigger
触发器

#### 相关属性
- jobKey属性：当trigger触发时被执行的job的身份；
- startTime属性：设置trigger第一次触发的时间；该属性的值是java.util.Date类型，表示某个指定的时间点
- endTime属性：表示trigger失效的时间点。
- 优先级（priority）如果你的trigger很多(或者Quartz线程池的工作线程太少)，Quartz可能没有足够的资源同时触发所有的trigger；这种情况下，你可能希望控制哪些trigger优先使用Quartz的工作线程，要达到该目的，可以在trigger上设置priority属性。比如，你有N个trigger需要同时触发，但只有Z个工作线程，优先级最高的Z个trigger会被首先触发。如果没有为trigger设置优先级，trigger使用默认优先级，值为5；priority属性的值可以是任意整数，正数、负数都可以。注意：只有同时触发的trigger之间才会比较优先级。10:59触发的trigger总是在11:00触发的trigger之前执行。如果trigger是可恢复的，在恢复后再调度时，优先级与原trigger是一样的。

#### 常用的触发器

##### SimpleTrigger
SimpleTrigger可以满足的调度需求是：在具体的时间点执行一次，或者在具体的时间点执行，并且以指定的间隔重复执行若干次。

示例：
```java
        //SimpleTrigger
        SimpleTrigger simpleTrigger = (SimpleTrigger) newTrigger()
                .withIdentity("trigger1","group1")//触发器标识
                .startAt(startTime)//开始触发时间
                .forJob("helloTask","task1")//绑定的任务组
                .withSchedule(simpleSchedule()//调度设定
                    .withRepeatCount(10)//重复次数
                    .withIntervalInMilliseconds(10))//执行频率
                .endAt(endTime)//结束时间
                .build();
```

##### CronTrigger
CronTrigger通常比Simple Trigger更有用，如果您需要基于日历的概念而不是按照SimpleTrigger的精确指定间隔进行重新启动的作业启动计划。使用CronTrigger，您可以指定号时间表，例如“每周五中午”或“每个工作日和上午9:30”，甚至“每周一至周五上午9:00至10点之间每5分钟”和1月份的星期五“。
即使如此，和SimpleTrigger一样，CronTrigger有一个startTime，它指定何时生效，以及一个（可选的）endTime，用于指定何时停止计划。

CronTrigger重点就是cron表达式，类似于spring自带的定时器中的cron表达式
可以参考这个：[spring自带定时器](https://blog.csdn.net/weixin_41922289/article/details/89883659)

示例：
```java
        //CronTrigger
        CronTrigger cronTrigger = (CronTrigger) newTrigger()
                .withIdentity("trigger2","group2")//触发器标识
                .startAt(startTime)//开始触发时间
                .forJob("helloTask1","task2")//绑定的任务组
                .withSchedule(cronSchedule("0 42 10 * * ?")//cron表达式 --- 每天10点42分执行
                        .withMisfireHandlingInstructionFireAndProceed())//错过触发 -- 一般是智能模式
                .endAt(endTime)//结束时间
                .build();

```

### 3.Scheduler
配置好任务器和触发器就可以开始调度了

>调度器工厂（SchedulerFactory）

所有的调度器示例都需要经过SchedulerFactory创建，而SchedulerFactory是根据quartz.properties文件定义的属性来创建和初始化Quartz Scheduler

```properties

org.quartz.scheduler.instanceName: DefaultQuartzScheduler
org.quartz.scheduler.rmi.export: false
org.quartz.scheduler.rmi.proxy: false
org.quartz.scheduler.wrapJobExecutionInUserTransaction: false

org.quartz.threadPool.class: org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount: 10
org.quartz.threadPool.threadPriority: 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread: true

org.quartz.jobStore.misfireThreshold: 60000

org.quartz.jobStore.class: org.quartz.simpl.RAMJobStore
```
示例：
```java
        //获取调度工厂
        SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();
        //从工厂中获取调度器
        Scheduler sched = schedFact.getScheduler();
        //开始调度
        sched.start();
        //挂起
        //sched.standby();
        //关闭任务调度
        //1.等待当前任务执行完毕在关闭任务调度
        //sched.shutdown(true);
        //2.直接关闭任务调度
        //sched.shutdown(false);
        //传入调度任务和任务触发器
        sched.scheduleJob(jobDetail,simpleTrigger);

```

### 监听器

#### 1.任务监听器
```java
/**
 * @Description:    任务监听器
 * @Author:         Kevin
 * @CreateDate:     2019/5/8 2:37
 * @UpdateUser:     Kevin
 * @UpdateDate:     2019/5/8 2:37
 * @UpdateRemark:   修改内容
 * @Version: 1.0
 */
public class TestJobListener implements JobListener {

    /**
     * 获取监听器名
     * @return
     */
    @Override
    public String getName() {
        return null;
    }

    /**
     * 执行任务前调用
     * @param jobExecutionContext
     */
    @Override
    public void jobToBeExecuted(JobExecutionContext jobExecutionContext) {

    }

    /**
     * 执行任务前，但是该任务被拒绝执行时调用
     * @param jobExecutionContext
     */
    @Override
    public void jobExecutionVetoed(JobExecutionContext jobExecutionContext) {

    }

    /**
     * 执行任务后调用
     * @param jobExecutionContext
     * @param e
     */
    @Override
    public void jobWasExecuted(JobExecutionContext jobExecutionContext, JobExecutionException e) {

    }
}

```
配置监听器
```java
        //全局任务监听配置
        sched.getListenerManager().addJobListener(new TestJobListener(), EverythingMatcher.allJobs());
        //局部任务监听配置
        sched.getListenerManager().addJobListener(new TestJobListener(), KeyMatcher.keyEquals(new JobKey("helloTask","task1")));
```
触发器监听器和调度器监听器使用和配置方法十分相似，省略不写

## 完整代码：
```java
    public static void test(Date startTime,Date endTime) throws SchedulerException {
        //由job类生产jobDetail
        JobDetail jobDetail = newJob(TestJob.class) //产生Job实例，job信息存储在jobDetail中
                .usingJobData("key1","val1")//通过jobDataMap为job实例增加属性，然后再传递到调度器中去
                .usingJobData("key2","val2")
                .withIdentity("helloTask","task1")
                .build();//由jobBuilder生成jobDetail

        //SimpleTrigger
        SimpleTrigger simpleTrigger = (SimpleTrigger) newTrigger()
                .withIdentity("trigger1","group1")//触发器标识
                .startAt(startTime)//开始触发时间
                .forJob("helloTask","task1")//绑定的任务组
                .withSchedule(simpleSchedule()//调度设定
                    .withRepeatCount(10)//重复次数
                    .withIntervalInMilliseconds(10))//执行频率
                .endAt(endTime)//结束时间
                .build();

        //CronTrigger
        CronTrigger cronTrigger = (CronTrigger) newTrigger()
                .withIdentity("trigger2","group2")//触发器标识
                .startAt(startTime)//开始触发时间
                .forJob("helloTask1","task2")//绑定的任务组
                .withSchedule(cronSchedule("0 42 10 * * ?")//cron表达式 --- 每天10点42分执行
                        .withMisfireHandlingInstructionFireAndProceed())//错过触发 -- 一般是智能模式
                .endAt(endTime)//结束时间
                .build();

        //获取调度工厂
        SchedulerFactory schedFact = new org.quartz.impl.StdSchedulerFactory();
        //从工厂中获取调度器
        Scheduler sched = schedFact.getScheduler();
        //开始调度
        sched.start();
        //挂起
        //sched.standby();
        //关闭任务调度
        //1.等待当前任务执行完毕在关闭任务调度
        //sched.shutdown(true);
        //2.直接关闭任务调度
        //sched.shutdown(false);
        //传入调度任务和任务触发器
        sched.scheduleJob(jobDetail,simpleTrigger);

        //全局任务监听配置
        sched.getListenerManager().addJobListener(new TestJobListener(), EverythingMatcher.allJobs());
        //局部任务监听配置
        sched.getListenerManager().addJobListener(new TestJobListener(), KeyMatcher.keyEquals(new JobKey("helloTask","task1")));
    }
```