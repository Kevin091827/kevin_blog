前文我们已经基本了解了Quartz中的一些基础概念[Quartz基础](https://blog.csdn.net/weixin_41922289/article/details/89944227)

现在我们来看看如何在springboot使用quartz

## 引入依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-quartz</artifactId>
        </dependency>
```

## 编写job
可以实现Job接口，也可以继承QuartzJobBean

```java

@DisallowConcurrentExecution    //相同定义的jobDetail不能并发执行
@PersistJobDataAfterExecution   //jobDataMap数据保存
@Slf4j
public class TestQuartzJob extends QuartzJobBean {

    public TestQuartzJob(){

    }
    /**
     * 具体任务
     * @param jobExecutionContext
     * @throws JobExecutionException
     */
    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("springboot整合定时任务Quartz:"+new Date());
    }
}

```

## 编写触发器和调度器

```java
@Configuration
public class TestQuartzConfig {

    /**
     * 创建jobDetail
     * @return
     */
    @Bean
    public JobDetail getJobDetail(){
        //由job类生产jobDetail
        JobDetail jobDetail = newJob(TestQuartzJob.class) //产生Job实例，job信息存储在jobDetail中
                .usingJobData("key1","val1")//通过jobDataMap为job实例增加属性，然后再传递到调度器中去
                .usingJobData("key2","val2")
                .withIdentity("h1","t1")
                .requestRecovery(true)
                .storeDurably()
                .build();//由jobBuilder生成jobDetail
        return jobDetail;
    }

    /**
     * 获取cronTrigger和设置调度器
     * @return
     */
    @Bean
    public CronTrigger getCronTrigger(){
       CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder
               .cronSchedule(" 0/5 * * * * ? ");

       return TriggerBuilder.newTrigger().forJob(getJobDetail())
               .withIdentity("testQuartz")
               .withSchedule(cronScheduleBuilder)
               .build();
    }

}

```

测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190509013849958.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

接下来我还会实现动态的定时任务管理，实现任务动态开启，暂停，删除的一个小demo
可以先参考这个[SpringBoot 集成Quartz发布、修改、暂停、删除定时任务](https://www.jianshu.com/p/e9d482dccd79)
