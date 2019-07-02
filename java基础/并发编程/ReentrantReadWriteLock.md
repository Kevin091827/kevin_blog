# 一，读写锁简介

之前说的ReentrantLock是一种排他锁，和synchronized类似，排他锁保证同一时刻只有一个线程进行访问，但是读写锁是维护一对读锁和写锁，读锁允许在同一时刻可以多个线程并发访问一个方法，写锁在访问时，所有读锁和其余写锁局不能访问，并发性比排他所高

# 二，ReentrantReadWriteLock简单使用

## 1.使用ReentrantReadWriteLock维护一个线程不安全的hashmap
```java
public class ReentrantReadWriteLockTest {

    //通过读写锁来维护一个线程不安全的hashmap
    static HashMap<String,Object> hashMap = new HashMap<>();
    //获取读写锁
    static ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    //获取读锁
    static Lock readLock = reentrantReadWriteLock.readLock();
    //获取写锁
    static Lock writeLock = reentrantReadWriteLock.writeLock();

    /**
     * 获取值  ----> 使用读锁，在同一时刻多个线程可以并发访问该方法，而不会被阻塞
     * @param key
     * @return
     */
    public static final Object getValue(String key){
        readLock.lock();
        try{
            return hashMap.get(key);
        }finally {
            readLock.unlock();
        }
    }

    /**
     * 写入map ------> 使用写锁，在同一时刻只有线程访问，只有当该线程释放锁，其他线程才可以访问
     * @param key
     * @param value
     */
    public static final void putValue(String key,Object value){
        writeLock.lock();
        try{
            hashMap.put(key, value);
        }finally {
            writeLock.unlock();
        }
    }
}
```

## 2.原理解析


ReentrantReadWriteLock实现了ReadWriteLock接口，ReadWriteLock接口定义了写锁和读锁两个方法用来分别获取写锁和读锁
```java
public interface ReadWriteLock {
    Lock readLock();

    Lock writeLock();
}
```
ReentrantReadWriteLock默认是非公平锁，但是也支持公平锁
```java
    public ReentrantReadWriteLock() {
        this(false);
    }
    public ReentrantReadWriteLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
        readerLock = new ReadLock(this);
        writerLock = new WriteLock(this);
    }
```
获取读锁和写锁


属性中的读锁和写锁是私有属性，通过这两个方法暴露出去。
```java
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

读写状态的设计

![](https://upload-images.jianshu.io/upload_images/2615789-6af1818bbfa83051.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/868/format/webp)

读写锁也是基于同步器实现的，所以其同步状态就是同步器的同步状态，但是读写锁是一对锁，不同于之前的排它锁，读写锁是通过位运算来维护读状态和写状态的。

读写锁需要在其自定义同步器上维护多个读线程和一个写线程

### 读状态

读状态由高16位维护，当前状态表示获取了一次读状态

假设当前同步器状态为S，则读状态为 S>>>16

获取一次读状态，则+1 ---> S+（1<<16）

### 写状态

写状态由低16位维护，当前状态表示获取了一次写状态，且重入了2次

写状态为：S&0x0000FFFF

每重入一次写状态，则+1

```java
        // 中间值
        static final int SHARED_SHIFT   = 16;
        // 由于读锁用高位部分，所以读锁个数加1，其实是状态值加 2^16
        static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
        // 写锁的可重入的最大次数、读锁允许的最大数量
        static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
        // 写锁的掩码，用于状态的低16位有效值
        static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

        //当前持有读锁的线程数 ---> c：一般传入getStatus()：同步状态值
        static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }

        //写锁重入次数 ---> c：一般传入getStatus()：同步状态值
        static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

### 读锁的获取和释放

读锁是一个支持重进入的共享锁，他能够被多个线程同时获取，在没有其他线程访问或者写状态为0时，读锁总能被成功的获取，且线程安全的增加读状态，如果当前线程已经获得了读锁，就增加读状态，如果当前线程在获取读锁时，写锁状态不为0，则进入等待状态

读状态的获取
```java
        protected final int tryAcquireShared(int unused) {
            
            Thread current = Thread.currentThread();
            int c = getState();
            //写状态！=0或者写锁被另一个线程持有，则失败
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            //获取持有读锁的线程数量    
            int r = sharedCount(c);
            //是否需要排队（是否是公平模式）&&是否小于最大线程数&&CAS增加读状态
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    //如果之前没有线程获取读锁，则记录第一个读者是当前线程
                    firstReader = current;
                    //第一个读者重入次数为1
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    //如果获取读锁的线程是第一个读者，则其重入状态+1
                    firstReaderHoldCount++;
                } else {
                    // 如果有线程获取了读锁且当前线程不是第一个读者
                    // 则从缓存中获取重入次数保存器
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    //重入+1    
                    rh.count++;
                }
                return 1;
            }
            // 通过这个方法再去尝试获取读锁（如果之前其它线程获取了写锁，一样返回-1表示失败）
            return fullTryAcquireShared(current);
        }
```

读状态的释放

读锁的每次释放都会减少读状态，减少的值是1<<16，释放读状态过程线程安全

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    // 清理firstReader缓存 或 readHolds里的重入计数
    //第一个读者是当前线程，且重入次数>0
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        //清空重入次数和移除读者
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        //第一个读者不是当前线程
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != current.getId())
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            // 完全释放读锁
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count; // 主要用于重入退出
    }
    // 循环在CAS更新状态值，主要是把读锁数量减 1，所以是线程安全的
    for (;;) {
        int c = getState();
        //每次减少值为：1<<16
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // 释放读锁对其他读线程没有任何影响，
            // 但可以允许等待的写线程继续，如果读锁、写锁都空闲。
            return nextc == 0;
    }
}
```

### 写锁的获取和释放

写锁是一个支持重入的排它锁，如果当前线程已经获取了写状态，则重入增加写状态，如果当前线程在获取写锁时，读锁已经被获取，此时读状态不为0，同步器状态不为0，或者该线程不是已经获取写锁的线程，则线程进入等待状态


写锁的获取
```java
        final boolean tryWriteLock() {
            //获取当前线程
            Thread current = Thread.currentThread();
            //获取同步器状态
            int c = getState();
            //同步器状态不为0
            if (c != 0) {
                //获取写锁重入次数
                int w = exclusiveCount(c);
                //存在读锁或者当前获得写锁的线程不是当前线程
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                //超过最大线程数量
                if (w == MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
            }
            //CAS设置同步器状态 ： 写状态+1
            if (!compareAndSetState(c, c + 1))
                return false;
            //设置当前获取写锁的线程是当前线程    
            setExclusiveOwnerThread(current);
            return true;
        }
```

写锁的释放

写锁的释放和排他锁的释放过程相似，每次释放都会减少写状态，当写状态为0时，表示写锁已经被成功释放

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            //同步队列
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
 
protected final boolean tryRelease(int releases) {
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            //写状态-1    
            int nextc = getState() - releases;
            //判断写状态是否为0
            boolean free = exclusiveCount(nextc) == 0;
            //如果为0，则写状态成功释放，释放线程
            if (free)
                setExclusiveOwnerThread(null);
            setState(nextc);
            return free;
        }
```

