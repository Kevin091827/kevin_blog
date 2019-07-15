# 一，公平锁和非公平锁

- 公平锁保证了在多线程环境下各个线程获取锁的顺序，先到的线程先获取到锁，也可以说，先对锁进行请求，请求等待时间最长的线程先获取到锁

- 非公平锁在线程获取时，各线程获取到的概率是随机的

- 如果要用同步队列器来讲的话，就是当第一个线程去请求获得锁时，如果成功获取到锁，则第二个线程只能进入同步队列等待第一个线程释放锁，然后在尝试获取锁，但是，如果此时来了第三个线程，如果是公平锁的话，会跟着进入同步队列排在第二个线程后，但是如果时非公平锁，则可能有一定概率会获取到锁，第二个线程仍然排在等待队列

- 公平锁往往没有非公平锁效率高

# 二，重入锁

## 1.概念
重入锁，就是支持重进入的锁，该锁能够支持一个线程对资源的重复加锁，还支持获取锁的公平和非公平性选择

### 什么是重进入？

重进入，是指任意线程在获取到锁后能够再次获取该锁而不被阻塞

### 实现重进入

实现重进入要解决两个问题：

- 线程再次获取到锁，锁需要去识别获取锁的线程是否是当前占据锁的线程，如果是，则获取锁成功

- 锁的最终释放，线程重复获取了n次锁，在第n次释放锁后，其他线程要能够获取到锁

#### 获取锁
ReentrantLock是通过组合自定义同步器来实现锁的获取和释放的

##### 非公平获取锁：

```java
        final boolean nonfairTryAcquire(int acquires) {
            //当前线程
            final Thread current = Thread.currentThread();
            //当前获取锁状态
            int c = getState();
            //未有线程获取锁的情况
            if (c == 0) {
                //CAS将当前线程设为成功获取锁状态
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //已有线程获取到锁
            //判断该获取到锁的线程是否是当前线程
            else if (current == getExclusiveOwnerThread()) {
                //同步状态值增加
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                //设置同步状态值
                setState(nextc);
                return true;
            }
            return false;
        }
```

##### 公平获取锁：

公平获取锁和非公平获取锁比较，只有一处不同，就是公平获取锁判断条件多了hasQueuedPredecessors()方法

```java
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                //判断在同步队列中当前线程节点是否有前驱节点
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```
如果当前线程节点前有前驱节点，则表示有线程比当前线程更早的请求获取锁，需要等该线程获取锁并且释放锁后才能继续获取锁


成功获取锁的线程再次获取锁时，只是简单的增加了同步状态值，那么在释放同步状态时必须减少同步状态值

#### 释放锁状态
```java
        protected final boolean tryRelease(int releases) {
            //减少同步状态值
            int c = getState() - releases;
            //当断是否是当前线程获取到锁
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            //同步状态为0，则成功释放锁，将线程移除
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
```
如果该锁被线程获取了n次，那么前n-1次tryRelease(int releases)必须返回false，而只有当同步状态完全释放，同步状态值减为0，才释放同步状态，返回true

#### 公平锁和非公平锁选择

在ReentrantLock中默认构造时非公平锁，但是也可以使用有参构造（传入true），实现公平锁
```java
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

**在非公平获取锁中，会出现一个线程连续获取锁的情况，这是为什么？**

在非公平获取锁中，一个线程请求锁时，只要获取了同步状态即成功获取锁，而且不会像非公平获取锁那样，在获取前会去判断同步队列中当前线程是否还有前驱节点，所以，在这个前提下，刚释放完锁的线程获得同步状态的几率会很大，使得其他线程还在队列中排队


#### 利与弊

- 公平锁保证了锁的获取按照同步队列的FIFO原则，但是代价是大量的线程切换

- 非公平锁可能会造成线程“饥饿”，但是极少的线程切换，开销小，保证了更大的吞吐量

**线程饥饿：**

　　指的是等待时间已经影响到进程运行，此时成为饥饿现象。如果等待时间过长，导致进程使命已经没有意义时，称之为“饿死”。