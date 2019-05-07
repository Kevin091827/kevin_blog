# 定时任务Timer

## 1.简介

Timer是jdk自带的一个定时器工具，使用的时候会在主线程之外另起一个新的线程执行定时任务，定时任务可以指定执行一次，也可以反复执行多次。

其中，Timer在使用过程中也会经常用到TimerTask，TimerTask是一个实现了Runnable接口的实现类（抽象类），代表一个可以被Timer执行的定时任务

## 2.使用

### 开启一个简单定时任务

步骤：

* 新建TimerTask并且重写run方法，在run方法中设置定时任务 
* 新建Timer定时器，设置定时任务（具体可以查看api文档）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506005029572.png)

参数说明：
```yaml
task：所要执行的任务，需要extends TimeTask override run()

time/firstTime：首次执行任务的时间

period：周期性执行Task的时间间隔，单位是毫秒

delay：执行task任务前的延时时间，单位是毫秒
```

实现：
```java
    /**
     * 定时任务（使用jdk自带的timer类）
     * @param args
     */
    public static void main(String[] args){
        TimerTask timerTask = new TimerTask() {
            @Override
            public void run() {
                log.info("定时任务："+new Date());
            }
        };
        Timer timer = new Timer();
        //定时任务延迟启动时间10毫秒
        //定时任务执行频率3秒
        timer.schedule(timerTask,10,3000);
    }
```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190506002910100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

但是，默认情况下，Timer创建出来的新线程会一直执行，也就是说，上述方法中的定时任务会一直以3秒一次3秒一次的执行下去，直到你把程序停掉

### 终止定时任务

调用timer的cancle方法来终止定时任务

## 总结
- 每一个Timer仅对应唯一一个线程,所以Timer在管理并发任务时会存在缺陷，所有任务都是由同一个线程来调度，所有任务都是串行执行，意味着同一时间只能有一个任务得到执行，而前一个任务的延迟或者异常会影响到之后的任务。

- 如果TimeTask抛出RuntimeException，那么Timer会停止所有任务的运行！

- Timer不保证任务执行的十分精确。

- Timer类的线程安全的。





