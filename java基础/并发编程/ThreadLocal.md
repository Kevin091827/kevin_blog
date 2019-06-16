## 一，ThreadLocal

>ThreadLocal是线程变量，是一个以ThreadLocal对象为键，任何对象为值得map，这个map附带在线程上，每一个线程都可以根据这个key找到附带在这个线程上的一个值

```java
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;
```

ThreadLocal是一个本地线程副本变量工具类。主要用于将私有线程和该线程存放的副本对象做一个映射，各个线程之间的变量互不干扰，在高并发场景下，可以实现无状态的调用，特别适用于各个线程依赖不通的变量值完成操作的场景。

### 1.内部结构图

![](https://upload-images.jianshu.io/upload_images/7432604-ad2ff581127ba8cc.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/806/format/webp)

### 2.内部核心机制

ThreadLocal内部核心机制：

- 每个Thread线程内部都有一个map

- map里边存储线程本地对象作为key，线程的变量副本作为value

- 但是线程内部的map是由ThreadLocall维护的，由ThreadLocal负责向map获取和设置线程的变量值。

- 对于不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。

## 二，深入ThreadLocal

### 1.核心方法

```java
//get()方法用于获取当前线程的副本变量值。
public T get()

//set()方法用于保存当前线程的副本变量值。
public void set(T value)

//remove()方法移除当前前程的副本变量值。
public void remove()

//initialValue()为当前线程初始副本变量值。
protected T initialValue()
```

### 2.源码分析

#### get()

```java
    public T get() {
        //获得当前线程对象
        Thread t = Thread.currentThread();
        //从map中获取线程存储的K-V Entry节点。
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            //如果map不空，则根据当前对象指定entry
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                //取值
                T result = (T)e.value;
                return result;
            }
        }
        //否则则回到初始化值
        return setInitialValue();
    }
```

#### set()

```java
    public void set(T value) {
         //获得当前线程对象
        Thread t = Thread.currentThread();
        //获取threadlocalmap
        ThreadLocalMap map = getMap(t);
        //设置
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

#### initialValue()

默认没有实现，可以根据自己需要重写，设置初始化值
```java
    protected T initialValue() {
        return null;
    }
```

#### remove()

```java
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

## 三，探究ThreadLocalMap的几个问题

### 1.简介
ThreadLocalMap是ThreadLocal的内部类，没有实现Map接口，用独立的方式实现了Map的功能，其内部的Entry也独立实现

![](https://upload-images.jianshu.io/upload_images/7432604-5bbe090d46789084.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/576/format/webp)

### 2.Hash冲突怎么解决

和HashMap的最大的不同在于，ThreadLocalMap结构非常简单，没有next引用，也就是说ThreadLocalMap中解决Hash冲突的方式并非链表的方式，而是采用线性探测的方式，所谓线性探测，就是根据初始key的hashcode值确定元素在table数组中的位置，如果发现这个位置上已经有其他key值的元素被占用，则利用固定的算法寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。

#### 解决方法

ThreadLocalMap解决Hash冲突的方式就是简单的步长加1或减1，寻找下一个相邻的位置。
```java
/**
 * Increment i modulo len.
 */
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}

/**
 * Decrement i modulo len.
 */
private static int prevIndex(int i, int len) {
    return ((i - 1 >= 0) ? i - 1 : len - 1);
}
```

ThreadLocalMap采用线性探测的方式解决Hash冲突的效率很低，如果有大量不同的ThreadLocal对象放入map中时发送冲突，或者发生二次冲突，则效率很低。

所以这里引出的良好建议是：每个线程只存一个变量，这样的话所有的线程存放到map中的Key都是相同的ThreadLocal，如果一个线程要保存多个变量，就需要创建多个ThreadLocal，多个ThreadLocal放入Map中时会极大的增加Hash冲突的可能。

### 3.内存泄漏问题

```java
        static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```
Entry继承自WeakReference（弱引用，生命周期只能存活到下次GC前），但只有Key是弱引用类型的，Value并非弱引用。

这样会导致一个内存泄漏的问题：

由于ThreadLocalMap的key是弱引用，而Value是强引用。这就导致了一个问题，ThreadLocal在没有外部对象强引用时，发生GC时弱引用Key会被回收，而Value不会回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。

#### 如何避免内存泄漏

既然Key是弱引用，那么我们要做的事，就是在调用ThreadLocal的get()、set()方法时完成后再调用remove方法，将Entry节点和Map的引用关系移除，这样整个Entry对象在GC Roots分析后就变成不可达了，下次GC的时候就可以被回收。

```java
ThreadLocal<Session> threadLocal = new ThreadLocal<Session>();
try {
    threadLocal.set(new Session(1, "kevin的博客"));
    // 其它业务逻辑
} finally {
    threadLocal.remove();
}
```