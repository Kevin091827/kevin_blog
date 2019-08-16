本来今天是想看一下Stack的源码的，但是在看到Stack的父类结构时

```java
public class Stack<E> extends Vector<E>
```
我想到了我之前还没怎么看过Vector的源码，甚至乎还很少用，我之前对他的了解大概就是停留在跟ArrayList很相似，是线程安全的ArrayList，先总结下ArrayList和Vector的不同之处，然后带着结论去看源码，找原因

# ArrayList和Vector对比：

- 相同之处：

    - 都是基于数组
    - 都支持随机访问
    - 默认容量都是10
    - 都支持动态扩容
    - 都支持fail—fast机制

- 不同之处：

    - Vector历史比ArrayList久远，Vector是jdk1.0，ArrayList是jdk1.2
    - Vector是线程安全的，ArrayList线程不安全
    - Vector动态扩容默认扩容两倍，ArrayList是1.5倍


# 底层数据结构

Vector底层是基于数组实现的
```java
protected Object[] elementData;
```
ArrayList底层数据结构也是数组
```java
private static final Object[] 
```

其他相关属性

```java
    //数组中元素数量
    protected int elementCount;
    //增长量
    protected int capacityIncrement;
```

# 构造方法

无参构造，数组容量默认是10
```java
    //无参构造，数组容量默认是10
    public Vector() {
        this(10);
    }
```
ArrayList默认数组容量也是10
```java
    //默认容量
    private static final int DEFAULT_CAPACITY = 10;

    //构造指定容量的数组（10）
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
Vcetor其他构造方法：

指定容量和增长量构造
```java
    //创建指定容量大小的数组，设置增长量。
    public Vector(int initialCapacity, int capacityIncrement) {
        super();
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        //构造指定容量的数组
        this.elementData = new Object[initialCapacity];
        //设置增长量
        this.capacityIncrement = capacityIncrement;
    }
```
指定容量和增长量为0的构造：
```java
    public Vector(int initialCapacity) {
        this(initialCapacity, 0);
    }
```
传入指定集合构造
```java
    public Vector(Collection<? extends E> c) {
        //转成数组，赋值
        elementData = c.toArray();
        elementCount = elementData.length;
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        //如果不是Object[]，要重建数组
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
    }
```
# 扩容机制

在了解添加元素之前我们需要理清Vector的扩容机制是怎样的，其实跟ArrayList的扩容机制也很相似

**1.计算最小容量：**

最小容量 = 当前数组元素数量 + 1，此举的目的就是判断是否需要扩容，最小容量就是相当于成功添加了一个元素后的新的数组元素数量，如果这个新的数组元素数量大于数组长度，那么肯定需要扩容
```java
    private void ensureCapacityHelper(int minCapacity) {
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

**2.传入最小容量开始扩容：**

- 如果当前数组的增长量 > 0则新数组容量 = 旧数组容量 + 增长量
- 否则，则新数组容量 = 2 * 旧数组容量
- 求出新数组容量后，如果新数组容量 < 最小容量，那么新数组容量 = 最小容量 
- 如果新数组容量 > 最大数组容量，则新数组容量 = 整数最大值     
```java
    //最小数组容量
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    //扩容
    private void grow(int minCapacity) {
        //旧的数组容量
        int oldCapacity = elementData.length;
        //新数组容量
        //如果当前数组的增长量 > 0则新数组容量 = 旧数组容量 + 增长量
        //否则，则新数组容量 = 2 * 旧数组容量
        int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                         capacityIncrement : oldCapacity);
        //求出新数组容量后，如果新数组容量 < 最小容量，那么新数组容量 = 最小容量                                  
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果新数组容量 > 最大数组容量，则新数组容量 = 整数最大值
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //真正扩容，实际上就是数组的复制和移动    
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

    //判断是取最大数组容量还是整数最大值
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

**4.扩容实际：数组复制和移动**
```java
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //params0：original：原数组
        //param1：srcPos：原数组开始位置
        //param2：copy：新数组
        //param3：destPost：新数组开始位置
        //param4：copyLength：要copy的数组的长度    
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
ArrayList的扩容机制其实和Vector很相似，至少原理是一致的，但是在扩容大小上不一样

因为ArrayList没有增长量这一概念，所以ArrayList默认扩容1.5倍
```java
    private void grow(int minCapacity) {
        // 原来的数组容量 = 数组长度
        int oldCapacity = elementData.length;
        // 新的数组容量 = 原数组容量+原数组容量/2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //判断下传进来的最小容量 （最小容量 = 当前数组元素数目 + 1）
        // 如果比当前新数组容量小，则使用最容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果判断当前新容量是否超过最大的数组容量 MAX_ARRAY_SIZE =  Integer.MAX_VALUE - 8
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //开始扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    
    //如果判断当前新容量是否超过最大的数组容量 MAX_ARRAY_SIZE =  Integer.MAX_VALUE - 8
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        //如果超多最大数组容量则使用Integer的最大数值，否则还是使用最大数组容量
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```

ArrayList扩容流程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190719181922137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 添加元素

**数组尾部添加指定元素**

可以看到添加方法上带有synchronized同步关键字，保证了在添加元素时的线程安全，但是也会带来获取锁和释放锁的效率问题
```java
    public synchronized void addElement(E obj) {
        modCount++;
        //判断是否需要扩容
        ensureCapacityHelper(elementCount + 1);
        //直接根据下标添加
        elementData[elementCount++] = obj;
    }
```

**指定位置添加指定元素**

```java
    public synchronized void insertElementAt(E obj, int index) {
        modCount++;
        if (index > elementCount) {
            throw new ArrayIndexOutOfBoundsException(index
                                                     + " > " + elementCount);
        }
        //判断是否需要扩容
        ensureCapacityHelper(elementCount + 1);
        //数组移动和复制，腾出index位置 index后的元素向后移动一位
        System.arraycopy(elementData, index, elementData, index + 1, elementCount - index);
        //下标添加
        elementData[index] = obj;
        //元素数量+1
        elementCount++;
    }
```

**添加指定集合**


```java
    public synchronized boolean addAll(Collection<? extends E> c) {
        modCount++;
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityHelper(elementCount + numNew);
         //扩容，复制到数组后面
        System.arraycopy(a, 0, elementData, elementCount, numNew);
        elementCount += numNew;
        return numNew != 0;
    }
```

# 删除元素

**删除指定下标元素**

```java
    public synchronized E remove(int index) {
        modCount++;
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);
        //原来该下标对应的元素值    
        E oldValue = elementData(index);
        //index后面元素的数量
        int numMoved = elementCount - index - 1;
        //如果该元素不是最后一个元素
        if (numMoved > 0)
            //该元素后面的元素向前移动一位，覆盖删除
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //数组最后多余的一位为null，gc                     
        elementData[--elementCount] = null; // Let gc do its work

        return oldValue;
    }

    //和上边方法其实思路是一样的
    public synchronized void removeElementAt(int index) {
        modCount++;
        //检查
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
             //该元素后面的元素向前移动一位，覆盖删除
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        //数组最后多余的一位为null，gc    
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```

**删除指定元素**

```java
    public synchronized boolean removeElement(Object obj) {
        modCount++;
        //找到该元素下标
        int i = indexOf(obj);
        if (i >= 0) {
            //下标正确则根据下标删除
            removeElementAt(i);
            return true;
        }
        return false;
    }
```

**删除所有元素**

```java
    public synchronized void removeAllElements() {
        modCount++;
        //循环删除每一个元素，gc
        for (int i = 0; i < elementCount; i++)
            elementData[i] = null;

        elementCount = 0;
    }
```

**删除指定范围的元素**

```java
    protected synchronized void removeRange(int fromIndex, int toIndex) {
        modCount++;
        //原理还是数组的移动，将toIndex后的元素向前移动 toIndex - fromIndex
        int numMoved = elementCount - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // Let gc do its work
        int newElementCount = elementCount - (toIndex-fromIndex);
        while (elementCount != newElementCount)
            elementData[--elementCount] = null;
    }
```

# 查询


**从指定下标开始找到指定元素第一次出现的下标**

从前往后找
```java
    public synchronized int indexOf(Object o, int index) {
        if (o == null) {
            for (int i = index ; i < elementCount ; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index ; i < elementCount ; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
从后往前找
```java
    public synchronized int lastIndexOf(Object o, int index) {
        if (index >= elementCount)
            throw new IndexOutOfBoundsException(index + " >= "+ elementCount);

        if (o == null) {
            for (int i = index; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = index; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
**返回指定下标的元素**

```java
    E elementData(int index) {
        return (E) elementData[index];
    }

    public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }
```

**是否包含指定元素**

```java
    //看没有该下标
    public boolean contains(Object o) {
        return indexOf(o, 0) >= 0;
    }
```

# 迭代器

迭代器和ArrayList相比是差不多的，包括实现也是，可以参考[ArrayList源码分析](https://blog.csdn.net/weixin_41922289/article/details/96414531)

