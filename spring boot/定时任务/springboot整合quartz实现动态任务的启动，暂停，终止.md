# 一，springboot整合quartz实现动态任务管理

近期在学习quartz，自己也动手用springboot和mybatis整合了quartz，实现对任务的动态开启和暂停，恢复和终止。

自己手动封装了一个相当于工具类的方法吧，通过传入任务名，任务组，还有相关job和cron表达式就可动态获取任务调度器

```java 
//StdSchedulerFactory工厂
    private static StdSchedulerFactory schedulerFactory = new StdSchedulerFactory();

    /**
     * 获取一个调度器
     * 通过 具体执行的任务器的.class类型，任务组名和任务名，还有cron表达式动态获取该任务的调度器
     * @param group
     * @param name
     * @param clazz
     * @param cron
     * @return
     */
    @Override
    public Scheduler startJob(String group,String name,Class clazz,String cron) {

        //封装信息
        JobDataMap jobDataMap = new JobDataMap();
        jobDataMap.put("group",group);
        jobDataMap.put("name",name);

        //创建jobDetail
        JobDetail jobDetail = newJob(clazz)
                .usingJobData(jobDataMap)
                .build();

        //创建cronScheduleBuilder
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(cron);

        //创建trigger
        CronTrigger cronTrigger = (CronTrigger) TriggerBuilder.newTrigger()
                .withIdentity(name, group)
                .withSchedule(cronScheduleBuilder)
                .build();

        //创建scheduler
        Scheduler scheduler = null;

        //配置scheduler，绑定任务和触发器
        try {
            scheduler = schedulerFactory.getScheduler();
            //绑定任务和触发器
            scheduler.scheduleJob(jobDetail,cronTrigger);
            //配置全局任务监听器
            scheduler.getListenerManager().addJobListener(new QuartzListener(),EverythingMatcher.allJobs());
        } catch (SchedulerException e) {
            e.printStackTrace();
        }

        return scheduler;
    }
```
### 开启定时任务

```java
//开启任务
scheduler.start();
```

### 暂停定时任务

```java
JobKey jobKey = new JobKey(name, group);
//暂停任务
TriggerKey triggerKey = new TriggerKey(name, group);
// 停止触发器
scheduler.pauseTrigger(triggerKey);
scheduler.pauseJob(jobKey);
```

### 恢复定时任务
```java

JobKey jobKey = new JobKey(name, group);
//恢复任务
TriggerKey triggerKey = new TriggerKey(name, group);
scheduler.resumeTrigger(triggerKey);
scheduler.resumeJob(jobKey);
```

### 终止定时任务

```java
JobKey jobKey = new JobKey(name, group);
TriggerKey triggerKey = new TriggerKey(name, group);
// 停止触发器
scheduler.pauseTrigger(triggerKey);
//移除触发器
scheduler.unscheduleJob(triggerKey);
//终止定时任务
scheduler.deleteJob(jobKey);
```

### 更新任务频度(cron)
```java
// 更新定时任务
scheduler.rescheduleJob(triggerKey, trigger);
```

完整项目地址：

[springboot整合quartz实现动态任务管理](https://github.com/Kevin091827/springboot_Quartz_mybatis)
