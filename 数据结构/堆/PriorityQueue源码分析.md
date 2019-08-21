PriorityQueue是一个优先队列，传统的队列是先入先出FIFO模型，但是优先队列跟传统队列不一样，优先队列会根据优先级来决定谁先出队，优先队列的底层数据结构是[堆](https://blog.csdn.net/weixin_41922289/article/details/99880857)用堆的好处是，每次入队和出队后，调整堆的时间复杂度为log（n）。

# 底层数据结构

上边提到优先队列的底层数据结构是堆，那么堆的底层数据结构是数组
```java
    //队列底层是数组
    transient Object[] queue; 
    //队列元素数量
    private int size = 0;
    //决定优先级的比较器
    private final Comparator<? super E> comparator;
    //支持fast-fail机制
    transient int modCount = 0;
```
# 构造方法

默认构造是数组长度为11的堆，jdk优先队列中默认是最大堆，即实现的自然排序是升序，待会在具体说
```java
    //默认队列长度
    private static final int DEFAULT_INITIAL_CAPACITY = 11;

    public PriorityQueue() {
        this(DEFAULT_INITIAL_CAPACITY, null);
    }
```
指定长度或指定构造器构造
```java
    //指定长度构造
    public PriorityQueue(int initialCapacity) {
        this(initialCapacity, null);
    }
    //指定构造器构造
    public PriorityQueue(Comparator<? super E> comparator) {
        this(DEFAULT_INITIAL_CAPACITY, comparator);
    }

    //同时指定两者
    public PriorityQueue(int initialCapacity,
                         Comparator<? super E> comparator) {
        // Note: This restriction of at least one is not actually needed,
        // but continues for 1.5 compatibility
        if (initialCapacity < 1)
            throw new IllegalArgumentException();
        this.queue = new Object[initialCapacity];
        this.comparator = comparator;
    }
```
指定集合构造
```java
    public PriorityQueue(Collection<? extends E> c) {
        // a.有序的情况
        if (c instanceof SortedSet<?>) {
            SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
            this.comparator = (Comparator<? super E>) ss.comparator();
            initElementsFromCollection(ss);
        }
        // b.是PriorityQueue或者它的子类
        else if (c instanceof PriorityQueue<?>) {
            PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
            this.comparator = (Comparator<? super E>) pq.comparator();
            initFromPriorityQueue(pq);
        }
        // c.无序情况
        else {
            this.comparator = null;
            initFromCollection(c);
        }
    }
```
对于有序情况的处理是这样的，直接复制数组赋值到queue
```java
    private void initElementsFromCollection(Collection<? extends E> c) {
        Object[] a = c.toArray();
        // If c.toArray incorrectly doesn't return Object[], copy it.
        if (a.getClass() != Object[].class)
            a = Arrays.copyOf(a, a.length, Object[].class);
        int len = a.length;
        if (len == 1 || this.comparator != null)
            for (int i = 0; i < len; i++)
                if (a[i] == null)
                    throw new NullPointerException();
        this.queue = a;
        this.size = a.length;
    }
```
如果是PriorityQueue或者它的子类
```java
private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
    if (c.getClass() == PriorityQueue.class) {
        // 如果是PriorityQueue的实例
        this.queue = c.toArray();
        this.size = c.size();
    } else {
        // 可能子类的结构和父类会有变化，所以和无序的处理方式一样
        initFromCollection(c);
    }
}
```
对于无序情况，则需要重新构建堆
```java
private void initFromCollection(Collection<? extends E> c) {
    // 初始化数组
    initElementsFromCollection(c);
    // 调整堆
    heapify();
}
```

# 建堆

jdk优先队列中默认是最大堆，需要通过堆的子节点和父节点相互比较，交换，构建堆结构

```java
    private void heapify() {
        for (int i = (size >>> 1) - 1; i >= 0; i--)
            siftDown(i, (E) queue[i]);
    }
```
默认没有自定义比较器的是构建最小堆，本文主要讨论默认最小堆的构建
```java
    private void siftDown(int k, E x) {
        if (comparator != null)
            siftDownUsingComparator(k, x);
        else
            siftDownComparable(k, x);
    }
```
下滤
```java
    private void siftDownComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>)x;
        //相当于size / 2，half相当于第一个叶子节点  
        int half = size >>> 1;        // loop while a non-leaf
        //如果小于half，则为非叶子节点，需要进行调整
        while (k < half) {
            //相当于 child = k * 2 + 1 ：child是左子节点 
            int child = (k << 1) + 1; // assume left child is least
            Object c = queue[child];
            //右子节点
            int right = child + 1;
            //如果右子节点在数组内，且右子节 > 左节点
            if (right < size &&
                ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
                //如果右节点 < 左节点则child指向右节点，child原来指向左节点
                c = queue[child = right];
            //如果当前x即父节点 < 较小子节点c，直接跳出，不用交换 
            if (key.compareTo((E) c) <= 0)
                break;
            //否则，较小子节点上升，父节点k下滤到c位置    
            queue[k] = c;
            //从子节点继续循环
            k = child;
        }
        //找到合适位置
        queue[k] = key;
    }
```
注意，建堆是通过下滤调整，从根到叶

# 入队

```java
    public boolean offer(E e) {
        if (e == null)
            throw new NullPointerException();
        //在迭代的时候要使用迭代器的出队和入队方法
        modCount++;
        //队列元素数目
        int i = size;
        //如果大于当前队列长度，则需要扩容
        if (i >= queue.length)
            grow(i + 1);
        //入队成功后的元素数目    
        size = i + 1;
        if (i == 0)
        //空队列的情况下，放堆根
            queue[0] = e;
        else
        //调整堆，入队的重点操作
            siftUp(i, e);
        return true;
    }
```
调整堆是从叶节点开始向上调整的，新插入的节点会放于数组最后（最右叶节点），然后通过上滤完成调整

选择比较规则，这里还是分析默认的比较规则。
```java
    private void siftUp(int k, E x) {
        if (comparator != null)
            siftUpUsingComparator(k, x);
        else
            siftUpComparable(k, x);
    }
```
上滤
```java
    private void siftUpComparable(int k, E x) {
        Comparable<? super E> key = (Comparable<? super E>) x;
        while (k > 0) {
            //相当于 (k - 1)/2，是父节点
            int parent = (k - 1) >>> 1;
            Object e = queue[parent];
            //此时key处于的是叶子节点。如果比父节点e大，则不用交换，直接退出循环
            if (key.compareTo((E) e) >= 0)
                break;
            //否则，则交换    
            queue[k] = e;
            //继续向上上滤
            k = parent;
        }
        //找到合适位置
        queue[k] = key;
    }
```

add(val)其实就是offer(val)的封装
```java
    public boolean add(E e) {
        return offer(e);
    }
```
# 出队
出队操作就是拿到了最优元素之后，取数组最后一个元素放于堆根，删除数组最后一个元素即最后一个叶子节点，重新调整堆
```java
    public E poll() {
        if (size == 0)
            return null;
        int s = --size;
        modCount++;
        //出队的永远是堆根，即最优元素，默认则是最小元素
        E result = (E) queue[0];
        //出队操作就是拿到了最优元素之后，取数组最后一个元素放于堆根，删除数组最后一个元素即最后一个叶子节点，重新调整堆
        E x = (E) queue[s];
        queue[s] = null;
        //出队之后要从根节点向下下滤调整堆
        if (s != 0)
            siftDown(0, x);
        return result;
    }
```
 
上面提到add()其实也是入队，但是remove()却不是出队
```java  
    public boolean remove(Object o) {
        //找到指定元素下标
        int i = indexOf(o);
        if (i == -1)
            return false;
        else {
            //根据下标直接移除
            removeAt(i);
            return true;
        }
    }
```    

# 扩容

复习一下，

- 之前在arrayList中，扩容默认是扩容1.5倍
- 在vector中也是基于数组实现，也需要扩容，默认是扩容2倍

在优先队列中，其实比前两者要简单

扩容：
- 默认扩容：
    - 如果队列长度小于64，则新长度 = 旧长度 * 2 + 2
    - 队列长度大于64，则新长度 = 旧长度 * 1.5
    - 如果新长度大于最大数组长度，则使用最大整形数值

```java
    private void grow(int minCapacity) {
        int oldCapacity = queue.length;
        // Double size if small; else grow by 50%
        //如果队列长度小于64，则新长度 = 旧长度 * 2 + 2
        //队列长度大于64，则新长度 = 旧长度 * 1.5
        int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
        // overflow-conscious code
        //如果新长度大于最大数组长度，则使用最大整形数值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        queue = Arrays.copyOf(queue, newCapacity);
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
# 其他方法


## 获得队头元素值

```java
    public E peek() {
        //其实就是取堆根
        return (size == 0) ? null : (E) queue[0];
    }
```

