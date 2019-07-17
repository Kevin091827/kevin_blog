# 1、数组介绍

数组是数据结构中很基本的结构，很多编程语言中都内置数组

数组是一片连续的内存区域，数组由数值和位置索引组成，所以可以根据索引将内存中的数值取出来，只有类型相同的数据才能一起存储到数组中

数组的优缺点：

- 顺序存储，存储数据的内存是连续的
- 查找数据容易，增删数据效率低下

# 2、ArrayList源码分析

## 1.构造方法

相关参数
```java
    //默认容量
    private static final int DEFAULT_CAPACITY = 10;
    //默认空数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
    //默认空数组
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    //不参与序列化的数组
    transient Object[] elementData; 
```

空参构造
```java
//构造指定容量的数组（10）
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```
有参构造
```java
    //构造指定容量的数组
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    //可以传递一个集合
    public ArrayList(Collection<? extends E> c) {
        //将集合转为数组
        elementData = c.toArray();
        //判断数组长度是否是空
        if ((size = elementData.length) != 0) {
            //数组不空，传进来的是有元素的集合
            //判断是否是Object[]类型
            if (elementData.getClass() != Object[].class)
                //如果不是Object[]，就返回一个包含相同元素的Object[]类型数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            //否则则返回空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
## 2.数据插入

因为ArrayList是基于数组实现的，数组呢，又是固定容量的，那么，在数据添加的问题上必然会涉及到扩容的问题，毕竟，默认大小只有10而已

**先看看ArrayList的扩容操作**

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

在谈数组拷贝
```java
    elementData = Arrays.copyOf(elementData, newCapacity);

    //参数情况：原数组，新的数组长度
    public static <T> T[] copyOf(T[] original, int newLength) {
        return (T[]) copyOf(original, newLength, original.getClass());
    }

    //参数情况：原数组，新数组长度，数组类型
    public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
        @SuppressWarnings("unchecked")
        //判断当前数组类型是不是Object[]，根据指定类型构建数组（反射）
        T[] copy = ((Object)newType == (Object)Object[].class)
            ? (T[]) new Object[newLength]
            : (T[]) Array.newInstance(newType.getComponentType(), newLength);
        //本地方法：参数情况：（原数组， 原数组的开始位置， 目标数组， 目标数组的开始位置， 拷贝个数）
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
        return copy;
    }
```
copyOf方法里的copy已经是扩容完成的数组，但是还是一个没有任何元素的数组，而下边的
```java
        System.arraycopy(original, 0, copy, 0,
                         Math.min(original.length, newLength));
```
则将原数组中的元素拷贝到扩容完成的数组

