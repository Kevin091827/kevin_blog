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

