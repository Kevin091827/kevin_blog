# ScheduledExecutorService

## 1.简介
ScheduledExecutorService是jdk对定时任务调度的线程池支持，由于Timer是单线程的，所以在解决并发任务时会存在缺陷，所有任务都由一个线程来管理，所有任务都是串行执行，同一时间只能执行一个任务。无法适应实际项目中任务定时调度的复杂度。所以，jdk5之后便退出了基于线程池的定时任务调度ScheduledExecutorService，每一个被调度的任务都会被线程池中的一个线程去执行，因此任务可以并发执行，而且相互之间不受影响。

## 使用
ScheduledExecutorService的使用方法和Timer差不多，区别就是ScheduledExecutorService不仅支持Runnable实现类还支持Callable实现类作为定时任务
```java
    /**
     * 定时任务（使用jdk自带的ScheduledExecutorService类）
     * @param args
     */
    public static void main(String[] args){
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                log.info("定时任务："+new Date());
            }
        };
        ScheduledExecutorService scheduledExecutorService = Executors.newSingleThreadScheduledExecutor();
//        TimeUnit  时间粒度转换
//        TimeUnit.DAYS          //天
//        TimeUnit.HOURS         //小时
//        TimeUnit.MINUTES       //分钟
//        TimeUnit.SECONDS       //秒
//        TimeUnit.MILLISECONDS  //毫秒
//        TimeUnit.NANOSECONDS   //毫微秒
//        TimeUnit.MICROSECONDS  //微秒
        //用法还是差不多
        //scheduledExecutorService.schedule(timerTask,10, TimeUnit.MILLISECONDS);
        scheduledExecutorService.scheduleAtFixedRate(timerTask,1,1,TimeUnit.SECONDS);
    }
```

结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506011032429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


