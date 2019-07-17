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

扩容1.5倍


**扩容流程图**

判断是否需要扩容，传入最小容量 = 当前数组容量 + 1
```java
    ensureCapacityInternal(size + 1); 

    //判断是否需要扩容
    private void ensureCapacityInternal(int minCapacity) {
        //先计算容量
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }
```
计算容量
```java
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        //如果是空数组，则返回默认容量10和传进来的最小容量的最大值
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }
```
开始扩容
```java
    private void ensureExplicitCapacity(int minCapacity) {
        //修改数增+1 快速失败机制
        modCount++;

        //如果增加后的数组容量（size+1）> 原数组容量则需要扩容
        if (minCapacity - elementData.length > 0)
            //扩容
            grow(minCapacity);
    }
```
然后具体扩容的方法和步骤就走上面扩容的流程


**了解完扩容，接下来看数据的添加会容易理解些**

再线性表末端增加指定元素
```java
    //最坏时间复杂度：O(1)
    public boolean add(E e) {
        //判断是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //线性表末端新增元素
        elementData[size++] = e;
        return true;
    }
```

在线性表指定位置增加指定元素
```java
    //最坏时间复杂度：O(N)
    public void add(int index, E element) {
        //检查指定下标是否越界
        rangeCheckForAdd(index);
        //判断是否需要扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //移动线性表中指定下标后一位开始到最后一个元素中的元素，移动一位
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        //空出来的位置即index下标位置则插入新元素
        elementData[index] = element;
        //数组容量+1
        size++;
    }
```

## 3.数据删除

指定下标删除
```java
    //最坏时间复杂度：O(N)
    public E remove(int index) {
        //检测下标是否越界
        rangeCheck(index);
        //修改数+1
        modCount++;
        //先取出要删除的元素（根据索引找出元素）
        E oldValue = elementData(index);
        //要移动的元素数量，即将index后得元素（不包括index）先向index方向移动一位，覆盖要被删除的index下标对应的元素
        int numMoved = size - index - 1;
        //要移动的元素数量>0则表示有元素要删除
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,numMoved);
        //清除最后一个元素
        elementData[--size] = null; // clear to let GC do its work
        //返回被删除元素的数值
        return oldValue;
    }
```
指定下标快速删除
```java
    //最坏时间复杂度：O(N)
    //只是一个私有方法，会跳过越界检查并且不会返回被删除元素的数值
    private void fastRemove(int index) {
        //思路和根据指定下标删除的remove方法一样，只是缺少了越界的检查
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }
```
指定元素删除  
```java
    //最坏时间复杂度：O(N^2)
    public boolean remove(Object o) {
        //判断要删除的元素是否是null,侧面反映ArrayList的add方法可以增加null元素
        if (o == null) {
            //如果是空，则遍历线性表找出所有null元素对应下标，并且根据下标快速删除
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            //否则则找出指定元素的下标，在根据下标快速删除指定元素
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        //没有匹配元素则返回null
        return false;
    }
```

## 4.ArrayList的迭代器

**先从一个遍历问题开始看起**

java集合的遍历细分可以有三种方式

1.简单循环遍历
```java
List<Integer> list = new ArrayList<>();
for(int i = 0;i<list.size();i++){
    //遍历
}
```
2.使用迭代器遍历
```java
List<Integer> list = new ArrayList<>();
Iterator iterator = list.iterator();
while(iterator.hasNext()){
    iterator.next();//遍历
}
```
3.使用增强for循环遍历
```java
List<Integer> list = new ArrayList<>();
for(int i : list){
    //遍历
}
```

在看使用迭代器遍历和使用增强for循环遍历的差异

能够使用增强for循环遍历的都是实现了Iterable接口的实现类
该接口是对迭代器即Iterator的一个包装
```java
public interface Iterable<T> {
    Iterator<T> iterator();
}
```

为什么需要对其进行包装？

因为Iterator是迭代器类，核心方法是hasNext(),next(),remove()都依赖于当前位置，如果集合类直接实现迭代器接口，则集合类需要维护多一个变量来指向当前迭代位置，不然当集合在方法间传递时，next执行的值即下一个迭代的值不能确定，但是，如果在Iterable的包装下，每次迭代都会返回一个新的从头开始的迭代器


**获取迭代器**
```java
public Iterator<E> iterator() {
        return new Itr();
    }
```
ListIterator是Iterator的一个子接口
```java
public ListIterator<E> listIterator() {
        return new ListItr(0);
    }
```

ArrayList中的迭代器是以内部类的形式存在

先关注几个重点变量
```java
        int cursor;       // 指向下一个被迭代元素的指针（或者说是其下标）
        int lastRet = -1; // 指向迭代器最后一次取出的元素的位置。或没有元素时取-1
        int expectedModCount = modCount;//修改期望值
```
关于expectedModCount和modCount的补充

在集合的add，remove等操作时，modCount都会+1，而在集合遍历中只有保证expectedModCount = modCount才算是合法的遍历，不然会报错ConcurrentModificationException

```java
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

看看合法遍历和不合法的遍历

不合法遍历，在遍历过程中使用集合的add，remove方法
```java
List<Integer> list = new ArrayList<>();
Iterator iterator = list.iterator();
while(iterator.hasNext()){
    int i = iterator.next();//遍历
    if(i == 12){
        list.remove(12);//集合的add，remove不能保证expectedModCount = modCount
    }
}
```
合法遍历，在遍历过程中使用迭代器的add，remove方法
```java
List<Integer> list = new ArrayList<>();
Iterator iterator = list.iterator();
while(iterator.hasNext()){
    iterator.next();//遍历
    if(i == 12){
        iterator.remove(); //迭代器的remove可以保证expectedModCount = modCount
    }
}
```
所以在进行集合元素的遍历时最好还是使用迭代的remove和add方法，更加符合规范和安全


**深究迭代器的remove和hasNext，和next方法**

hasNext()
```java
        public boolean hasNext() {
            //判断指向当前迭代位置的（下标）的指针是否已经指向数组末尾
            return cursor != size;
        }
```
next()
```java
        public E next() {
            checkForComodification();
            // i = 当前迭代位置下标
            int i = cursor;
            //检查i的合法性
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            //每迭代一次，i往后移动一位，cursor+1
            cursor = i + 1;
            //根据下标取元素
            return (E) elementData[lastRet = i];
        }
```


remove()
```java
        public void remove() {
            //检查前驱指针是否合法
            if (lastRet < 0)
                throw new IllegalStateException();
            //检查期望修改值是否等于修改值
            checkForComodification();

            try {
                //其实还是使用arrayList的remove(),此时expectedModCount和modCount是不相等的
                ArrayList.this.remove(lastRet);
                //移动指针，后继指针指向刚刚删除元素的前驱
                cursor = lastRet;
                lastRet = -1;
                //更新expectedModCount，保证其相等
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```
看完最基本的迭代器，可以看看list自身自定义的一个迭代器
```java
private class ListItr extends Itr implements ListIterator<E> {
        ListItr(int index) {
            super();
            cursor = index;
        }
        ...
```
可以看出，该迭代器是一个从指定位置开始进行迭代的迭代器，而且该迭代器还实现了基本迭代器没有的set 和 add 方法

```java
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }

        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                ArrayList.this.add(i, e);
                cursor = i + 1;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

ArrayList源码中的基本知识算是整理完了吧，可能会有不全，但是还是想说，长文警告！