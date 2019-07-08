>在看并发编程这本书时，看到join()方法，似曾相识的感觉，但是一时间又想不出具体的作用，觉得还是写写代码记录下，加深记忆号

先看一个demo
```java
public class JoinAndSleepDemo {

   private static Thread thread1 = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("子线1程在执行----------1111111111");
        }
    });

   private static Thread thread2 = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println("子线程2在执行----------2222222222");
        }
    });

    public static void main(String[] args) throws InterruptedException {
        //主线程
        System.out.println("---------主线程------"+Thread.currentThread().getName());

        System.out.println("子线程执行");
        //--------子线程执行
        thread1.start();
       // thread1.join();
        thread2.start();
        //thread2.join();
        //主线程希望等待子线程执行完再执行
        System.out.println("主线程还在执行");
    }

}
```

现在先不给出正确的结果，先猜想这段代码的输出会是什么样，注意，join方法都被注释掉

猜想一：
```
---------主线程------main
子线程执行
子线1程在执行----------1111111111
子线程2在执行----------2222222222
主线程还在执行
```
显然，这种猜想跟程序执行顺序差不多，思路基本上可以确定是，主线程 -- 子线程 -- 主线程的顺序，也就是主线程是在子线程执行完时还在执行，主线程执行时间最长

猜想二：
```
---------主线程------main
子线程执行
主线程还在执行
子线1程在执行----------1111111111
子线程2在执行----------2222222222
```
这种猜想就是主线程的执行最早结束，不等待子线程执行完就已经结束了

实际上正确的结果也就是猜想二，对于这种情况，如果有时候我们在一些业务场景中主线程希望等待子线程执行完成，并且获取子线程执行结果，那么我们该如何处理呢？


### join()

还是上面那个demo，这次我把join()方法取消注释

```java
    public static void main(String[] args) throws InterruptedException {
        //主线程
        System.out.println("---------主线程------"+Thread.currentThread().getName());

        System.out.println("子线程执行");
        //--------子线程执行
        thread1.start();
        thread1.join();
        thread2.start();
        thread2.join();
        //主线程希望等待子线程执行完再执行
        System.out.println("主线程还在执行");
    }
```

那么会得到什么样的结果呢？
```
---------主线程------main
子线程执行
子线1程在执行----------1111111111
子线程2在执行----------2222222222
主线程还在执行
```
此时这种情况，主线程就会等待执行了join方法的两个子线程执行完。

表面看似这样，可实际原理是什么呢？

我们先观察线程状态在执行join方法前后会方法什么变化？
```java
    public static void main(String[] args) throws InterruptedException {
        //主线程
        System.out.println("主线程状态---------->"+Thread.currentThread().getState());
        System.out.println("---------主线程------"+Thread.currentThread().getName());
        System.out.println("主线程状态---------->"+Thread.currentThread().getState());
        System.out.println("子线程执行");
        //--------子线程执行
        thread1.start();
        System.out.println(Thread.currentThread().getName()+"线程状态---------->"+Thread.currentThread().getState());
        System.out.println("线程111111111状态-----------》"+thread1.getState());
        thread1.join();
        System.out.println("线程111111111状态-----------》"+thread1.getState());
        System.out.println(Thread.currentThread().getName()+"线程状态---------->"+Thread.currentThread().getState());
        thread2.start();
        System.out.println("线程222222222状态-----------》"+thread2.getState());
        thread2.join();
        System.out.println("线程222222222状态-----------》"+thread2.getState());
        System.out.println(Thread.currentThread().getName()+"线程状态---------->"+Thread.currentThread().getState());
        //主线程希望等待子线程执行完再执行
        System.out.println("主线程还在执行");
        System.out.println(Thread.currentThread().getName()+"线程状态---------->"+Thread.currentThread().getState());
    }
```

变化结果：

```
主线程状态---------->RUNNABLE
---------主线程------main
主线程状态---------->RUNNABLE
子线程执行
main线程状态---------->RUNNABLE
线程111111111状态-----------》RUNNABLE
子线1程在执行----------1111111111
线程111111111状态-----------》TERMINATED
main线程状态---------->RUNNABLE
线程222222222状态-----------》RUNNABLE
子线程2在执行----------2222222222
线程222222222状态-----------》TERMINATED
main线程状态---------->RUNNABLE
主线程还在执行
main线程状态---------->RUNNABLE
```
可以看到子线程在执行了join方法后线程状态由RUNNABLE变为TERMINATED

主线程一直都是RUNNABLE

所以，我们大概可以得到一个结论就是：

子线程thread1在调用join()方法后，thread1正常执行run，但是当前thread1所在的main线程就会被无限期阻塞，只有当thread1执行完，main线程才能恢复正常执行

因此，join() 方法使得调用该方法的那段代码所在的线程暂时阻塞