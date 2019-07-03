# 一，condition简介
任何一个java对象都拥有一对监视器方法（定义在Object上）主要包括：wait()/wait(long timeOut)，notify(),notifyAll(),这些方法和synchronized配合实现等待/通知机制

Condition接口也提供了类似的监视器方法，配合Lock实现等待/通知机制

# 二，使用

## 1.简单使用

- 调用lock.newCondition()获得condition对象
- 配合lock使用
- 支持响应中断和对中断不敏感的进入等待状态
- 支持唤醒一个或多个线程
- 支持当前线程释放锁并进入超时等待状态

```java
public class ConditionDemo {

    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    /**
     * 线程等待（对中断敏感）
     * <h> condition.await() </h>
     * <p>
     *    等待：
     *          当前线程进入等待状态直到被通知或被中断
     *    唤醒：
     *          1.signal()，signalAll()
     *          2.其他线程调用interrupt()中断该线程（对中断敏感）
     * </p>
     */
    public void awaitThread(){
        //执行condition相关方法先获取到锁（lock）
        lock.lock();
        try{
            //执行condition相关方法
            condition.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }finally {
            //释放锁
            lock.unlock();
        }
    }

    /**
     * 线程等待（对中断不敏感）
     * <h> condition.awaitUninterruptibly() </h>
     * <p>
     *    等待：
     *          当前线程进入等待状态直到被通知
     *    唤醒：
     *          1.signal()，signalAll()
     * </p>
     */
    public void awaitUninterruptiblyThread(){
        //执行condition相关方法先获取到锁（lock）
        lock.lock();
        try{
            //执行condition相关方法
            condition.awaitUninterruptibly();
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            //释放锁
            lock.unlock();
        }
    }


    /**
     * 唤醒当前等待的线程
     * <h> condition.signal() </h>
     */
    public void signalThread(){
        lock.lock();
        try {
            condition.signal();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }


    /**
     * 唤醒所有等待的线程
     * <h> condition.signalAll() </h>
     */
    public void signalAllThread(){
        lock.lock();
        try {
            condition.signalAll();
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.unlock();
        }
    }

}
```

## 2.基于condition的有界队列demo

```java
public class BoundedQueue {

    //基于数组的简单有界队列
    private Integer[] array ;

    /**
     * <params>
     *     addIndex : 添加索引
     *     removeIndex ： 移除索引
     *     count ：当前队列元素数量
     * </params>
     */
    private int addIndex,removeIndex,count;

    private Lock lock = new ReentrantLock();

    //当队列满时的等待/通知机制
    private Condition fullCondition = lock.newCondition();

    //当队列空时的等待/通知机制
    private Condition emptyCondition = lock.newCondition();

    public BoundedQueue(int size){
        array = new Integer[size];
    }

    /**
     * 向队列添加一个元素，当队列满时阻塞线程，不能添加
     * @param i
     * @throws InterruptedException
     */
    public void add(int i) throws InterruptedException {
        lock.lock();
        try{
            //判断队列是否已满,有空位才能跳出循环完成添加
            while(count == array.length){
                fullCondition.await();
            }
            array[addIndex] = i;
            //如果刚好达到队列满时，重置
            if(++addIndex == array.length){
                addIndex = 0;
            }
            ++count;
            fullCondition.signal();
        }finally {
            lock.unlock();
        }
    }

    /**
     * 移除队列元素，当队列是空时，线程被阻塞
     * @return
     */
    public Integer remove(){
        lock.lock();
        try{
            while(count == 0){
                emptyCondition.awaitUninterruptibly();
            }
            Integer x = array[removeIndex];
            array[removeIndex] = null;
            if(++removeIndex == array.length){
                removeIndex = 0;
            }
            --count;
            emptyCondition.signal();
            return x;
        }finally {
            lock.unlock();
        }
    }
}
```

# 三，实现分析

每个condition对象都包含一个等待队列，这个等待队列是实现等待/通知机制的重点

## 1.await()

源码：
```java
        public final void await() throws InterruptedException {
            //不响应中断
            if (Thread.interrupted())
                throw new InterruptedException();
            //构造节点，加入到等待队列中
            Node node = addConditionWaiter();
            //释放该节点对应线程的同步状态
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            //判断节点是否还在同步队列
            while (!isOnSyncQueue(node)) {
                //在同步队列则阻塞该线程
                LockSupport.park(this);
                //如果中断则跳出循环
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            //在被别的线程唤醒后, 将刚刚这个节点放到 AQS 队列中.接下来就是那个节点的事情了,比如抢锁.
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

每次节点加入到等待队列尾部，不需要CAS添加

因为在调用await()时线程已经获得了锁，通过锁来保证线程安全


## 2.signal()


```java
        public final void signal() {
            //检查是否获取了锁
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            //从等待队列移除首节点    
            Node first = firstWaiter;
            if (first != null)
                doSignal(first);
        }

        private void doSignal(Node first) {
            do {
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                    first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }

        final boolean transferForSignal(Node node) {
            //CAS设置节点状态
            if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
                return false;
            //进入同步队列    
            Node p = enq(node);
            int ws = p.waitStatus;
            //唤醒该节点对应线程
            if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
                LockSupport.unpark(node.thread);
            return true;
        }
```