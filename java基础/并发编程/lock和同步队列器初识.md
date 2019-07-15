
>锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源，但是有些锁可以允许多个线程并发的访问共享资源（读写锁）

# 一，Lock接口

java se5前，java程序是依靠synchronized关键字实现锁功能的，但是java se5之后，可以使用lock接口实现锁功能

## 1.与synchronized异同之处

**同：**

- 都可以用来实现锁的功能

**异：**

- lock在使用时需要显式的获取锁和释放锁 

- synchronized隐式的获取锁和释放锁

- 虽然lock缺少了synchronized隐式的获取锁和释放锁，但是却拥有了锁释放和释放的可操作性，可中断的获取锁以及超市获取锁等多种特性


## 2.特性

|特性|描述|
|-|-|
|尝试非阻塞地获取锁|当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并持有锁
|能被中断的获取锁|获取到锁的线程能够响应中断，当获取到锁的线程被中断时，中断异常会被抛出，同时锁会被释放
|超时获取锁|在指定截止时间之前获取锁，如果截止时间到了仍旧无法获取锁，则返回


## 3.API表

```java
    
    //获取锁，调用该方法当前线程会获取锁
    void lock();
    
    //可中断的获取锁
    void lockInterruptibly();

    //非阻塞的获取锁
    boolean tryLock();
    
    //超时获取锁
    boolean tryLock(long time, TimeUnit timeUnit);
    
    //释放锁
    void unLock();
    
    //获取等待通知组件
    Condition newCondition();
    
```

# 二，队列同步器

## 1.简介
队列同步器是用来构建锁或者其他同步组件的基础框架，它是使用了一个int成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取线程的排队工作

## 2.使用

### 访问或修改同步状态

```java
    //获取当前同步状态
    int getState();
    
    //设置当前同步状态
    boolean setState(int newState);
    
    //设置当前状态
    boolean compareAndSetState(int expect,int update);
```
### 同步器可重写的方法

```java
    //独占式获取同步状态
    protected boolean tryAcquire(int arg);

    //独占式释放同步状态
    protected boolean tryRelease(int arg);

    //共享式获取同步状态
    protected int tryAcquireShared(int arg);

    //共享式释放同步状态
    protected boolean tryReleaseShared(int arg);

    //判断当前同步器是否在独占模式下被当前线程占用
    protected boolean isHeldExclusively();
```

### 同步器提供的模板方法
```java
    //独占式获取同步状态
     void acquire(int arg);

    //独占式获取同步状态(可中断)
    void acquireInterruptibly(int arg);

    //独占式获取同步状态（超时获取）
    boolean tryAcquireNanos(int arg,long nanos);

    //共享式获取同步状态
    void acquireShared(int arg);

    //共享式获取同步状态(可中断)
    void acquireSharedInterruptibly(int arg);

    //共享式获取同步状态（超时获取）
    boolean tryAcquireSharedNanos(int arg,long nanos);

    //独占式释放同步状态
    boolean release(int arg);

    //共享式释放同步状态
    boolean releaseShared(int arg);

    //获取等待在同步队列上的线程集合
    Collection<Thread> getQueuedThreads();
```

### 独占锁实现

同步器面向锁的实现

lock面向锁的使用

```java
/**
 * @Description:    独占锁
 * @Author:         Kevin
 * @CreateDate:     2019/6/19 23:51
 * @UpdateUser:     Kevin
 * @UpdateDate:     2019/6/19 23:51
 * @UpdateRemark:   修改内容
 * @Version: 1.0
 */
public class Mutex implements Lock {

    /**
     * 同步器
     */
    private static class Sync extends AbstractQueuedSynchronizer{
        /**
         * 获取锁
         * @param arg
         * @return
         */
        @Override
        protected boolean tryAcquire(int arg) {
            if(compareAndSetState(0,1)){
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        /**
         * 释放锁
         * @param arg
         * @return
         */
        @Override
        protected boolean tryRelease(int arg) {
           if(getState() == 0){
               try {
                   throw new Exception();
               } catch (Exception e) {
                   e.printStackTrace();
               }
               return false;
           }else {
               setExclusiveOwnerThread(null);
               setState(0);
               return true;
           }
        }

        /**
         * 是否处于占用状态
         * @return
         */
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }
    }

    private Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1,1);
    }

    @Override
    public void unlock() {
        sync.release(1);
    }

    @Override
    public Condition newCondition() {
       return (Condition) sync.getExclusiveQueuedThreads();
    }
}

```
#### 实现分析
队列同步器的实现依赖内部的同步队列来完成同步状态的管理。它是一个FIFO的双向队列，当线程获取同步状态失败时，同步器会将当前线程和等待状态等信息包装成一个节点并将其加入同步队列，同时会阻塞当前线程。当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态。

Node静态内部类的源码
```java
static final class Node {
        /** 标识一个这个节点是否为shared类型 */
        static final Node SHARED = new Node();
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;
 
        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3;
 
        /**
         *   等待状态值，又以下状态值:
		 *
         *   SIGNAL:	 值为-1 ，后续节点处于等待状态，而当前节点的线程如果
		 * 			 	 释放了同步状态或者取消等待，节点进入该状态不会变化          
         *   CANCELLED:  值为 1，由于在同步队列中等待的线程等待超时或者被中断
         *               需要从同步队列中取消等待，节点进入该状态将不会变化                    
         *   CONDITION:  值为-2，节点在等待队列中，节点线程等待在Condition上，
         *               当其他线程对Condition调用了signal()方法后，该节点将会
         *               从等待队里中转移到同步队列中，加入对同步状态的获取中
        
         *   PROPAGATE:  值为-3，表示下一次共享式同步状态获取将会无条件地被传播下去
         *            
         *   0:          初始化状态
         */
        volatile int waitStatus;
 
        /**
		 * 前驱节点，当节点加入同步队列时被设置
         */
        volatile Node prev;
 
         /**
		 * 后继节点
         */
        volatile Node next;
 
        /**
		 * 获取同步状态的线程
         */
        volatile Thread thread;
 
         /**
		 * 等待队列中的后继节点。如果当前节点是共享的，那么这个字段是一个shared常量，
		 * 也就是说节点类型（独占或共享）和等待队列中个后继节点共用同一个字段
		 */
        Node nextWaiter;
 
        /**
         * Returns true if node is waiting in shared mode.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }
 
        /**
         * Returns previous node, or throws NullPointerException if null.
         * Use when predecessor cannot be null.  The null check could
         * be elided, but is present to help the VM.
         *
         * @return the predecessor of this node
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
 
        Node() {    // Used to establish initial head or SHARED marker
        }
 
        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }
 
        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```

同步队列结构：

![](https://img-blog.csdn.net/20151219193538740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

同步队列遵循FIFO，首节点是获取同步状态成功的节点，首节点线程在释放同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点

![](https://img-blog.csdn.net/20151219195145118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

同步器包含了两个节点类型的引用，一个指向头节点，而另一个指向尾节点。
如果一个线程没有获得同步队列，那么包装它的节点将被加入到队尾，显然这个过程应该是线程安全的。因此同步器提供了一个基于CAS的设置尾节点的方法：compareAndSetTail(Node expect,Node update),它需要传递一个它认为的尾节点和当前节点，只有设置成功，当前节点才被加入队尾

![](https://img-blog.csdn.net/20151219194411123?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

头结点是通过获取同步状态成功的线程来设置的，只有一个线程，因此不需CAS设置

尾节点需要CAS设置，保证线程安全

