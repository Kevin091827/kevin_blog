## 一，volatile

### 1.作用
保证变量在多线程环境下的线程可见性，其他线程能感知到该变量的每一次修改，读到变量的值都是最新的


### 2.原理
在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更轻量级的同步机制。

![](https://images2015.cnblogs.com/blog/731716/201607/731716-20160708224602686-2141387366.png)


对于非volatile修饰的变量，每个线程可以拥有这个变量的一份拷贝，放在cpu缓存中

对于volatile修饰的变量，每个线程则是直接访问内存来访问这个变量，并且每一次对该变量的修改都要访问内存中，保证变量是最新的

## 二，synchronized

### 1.synchronized基本用法

synchronized是实现同步的基础，他是一种重量级的锁，相对于volatile来说

他保证了在多线程环境下，对方法的访问在某个时刻只有一个线程进入到方法体内

#### 修饰实例方法

```java
    private int i = 0;
    /**
     * synchronized修饰实例方法   内置锁是当前类对象
     * @return
     */
    public synchronized int test1(){
        return i++;
    }
```

#### 修饰静态方法

```java
    private static int j = 0;
    /**
     * synchronized修饰静态方法   内置锁是当前类的class对象
     * @return
     */
    public synchronized  static int test2(){
        return j++;
    }
```

#### 修饰代码块
```java
    /**
     * synchronized修饰同步代码块  内置锁是自定义的对象
     */
    public void test3(){
        synchronized (Integer.valueOf(i)){
            i++;
        }
    }
```

### 2.浅谈synchronized原理


#### synchronized执行过程

每个对象都有一个对象监视器monitor，当在多线程环境下时，一个线程只有获得了该对象的对象监视器才能进入到同步代码块或者同步方法中，没有获得对象监视器的线程则从等待队列进入到同步队列中（等待状态 --> 阻塞状态），只有当刚刚获得对象监视器的线程释放了该对象监视器，其他线程才有机会去获得到该对象监视器从而进入同步方法或者同步代码块中

**过程图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190614144715643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

#### 对象头

**上面我们刚刚说过任何对象都可以作为锁，那么，锁信息又存在对象的什么地方呢？**

synchronized锁信息是存在对象头中的

对象头：

|内容|说明|
|--|---|
|Mark Word|存储对象的hashcode或锁信息|
|Class Metadata Address|存储指向对象数据类型的指针|
|Array Length|数组长度（如果当前对象是数组）|

java对象头中的MarkWord中默认存储对象的Hashcode，分代年龄和锁标记位

|锁状态|25bit|4bit|是否是偏向锁|锁标记位|
|--|--|--|--|--|
|无锁状态|对象hashcode|对象分代年龄|0|01|
|轻量级锁|指向栈中锁记录的指针|||00
|重量级锁|指向互斥量（重量级锁）的指针|||10
|偏向锁|线程ID|Epoch|对象分代年龄|1| 01|

### 3.锁升级和对比

java1.6为了减少获得锁带来的性能消耗，引入了偏向锁和轻量级锁

**锁的四种状态**

* 无锁状态
* 轻量级锁
* 重量级锁
* 偏向锁

锁可以随着竞争状态升级但是不能降级

#### 偏向锁

##### 出现原因

在多线程环境下，锁在大多数情况下不仅不存在竞争，而且还几乎都是被同一个线程多次获得，为了让线程获得锁的性能代价更低，引入了偏向锁


##### 偏向锁获得锁过程

当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁）：如果没有设置，则使用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190615003103314.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 偏向锁释放锁过程

偏向锁使用一种等到竞争出现才会释放锁的机制，只有等到其他线程需要竞争偏向锁时，持有偏向锁的线程才会开始释放锁，释放过程：该线程会先暂停，等到全局安全状态（无字节码执行）时会先检查当前持锁线程是否存活，如果不存活则直接进入无锁状态，如果存活，则对象头偏向于其他线程或者恢复为无锁状态，最后唤醒线程

#### 轻量级锁


##### 轻量级锁加锁
线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁

##### 轻量级锁释放锁
轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。
