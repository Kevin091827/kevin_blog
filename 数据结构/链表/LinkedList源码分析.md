
# 1、linkedList底层数据结构

linkedList和ArrayList在底层的实现上有所不同，ArrayList底层实现是数组，linkedList底层实现是双向链表

![](https://images2015.cnblogs.com/blog/616953/201603/616953-20160322214504120-1558870057.png)

##链表介绍##
链表是一种物理存储单元上非连续的，非顺序的储存结构，链表由节点和指针构成，节点可以在运行时动态生成


## 2.源码分析

### 1.节点类

我们希望从linkedList中维护的内部类来进一步证实linkedList底层是一个双向链表

linkedList内部维护了一个节点类，很多操作都基于节点进行
```java
    private static class Node<E> {
        //数值域
        E item;
        //后继指针
        Node<E> next;
        //前驱指针
        Node<E> prev;
        //节点构造方法
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
### 2.基本属性

```java
    //实际元素数量
    transient int size = 0;
    //头结点
    transient Node<E> first;
    //尾节点
    transient Node<E> last;
```

### 3.构造方法
```java
    //linkedList构造方法
    public LinkedList() {
    }
    //linkedList构造方法
    public LinkedList(Collection<? extends E> c) {
        this();
        addAll(c);
    }
```

### 4.找出指定下标的元素或节点
找出指定下标构造节点
```java
    //找出指定下标构造节点
    Node<E> node(int index) {
        // assert isElementIndex(index);
        //如果index位于前一半
        if (index < (size >> 1)) {
            //x：头结点
            Node<E> x = first;
            //找到指定下标index，返回index节点
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            //从尾节点向前找，返回index节点
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

### 4.元素的添加

在头部增加指定元素
```java
    private void linkFirst(E e) {
        //当前头结点 -- f
        final Node<E> f = first;
        //构造一个新节点，前驱节点是null，后继节点是f,数值是e
        final Node<E> newNode = new Node<>(null, e, f);
        //头部增加，新构造节点变为头结点
        first = newNode;
        //判断是不是只有原头结点是否是空节点
        if (f == null)
            //空节点，那么现在只有一个节点，头结点和尾节点都是新构造节点
            last = newNode;
        else
            //否则，新节点变为原头结点的前驱节点
            f.prev = newNode;
        //节点数量增加    
        size++;
        //修改数增加
        modCount++;
    }

    public void addFirst(E e) {
        linkFirst(e);
    }
```

在尾部增加指定元素
```java
    //思路差不多，只需修改原来尾节点的后继节点为新节点
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
        //原来尾节点的后继节点为新节点
            l.next = newNode;
        size++;
        modCount++;
    }

    public void addLast(E e) {
        linkLast(e);
    }

    //默认尾部添加
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```

在指定节点前增加节点
```java
    void linkBefore(E e, Node<E> succ) {
        // assert succ != null;
        final Node<E> pred = succ.prev;
        //以pred为前驱，succ为后继构造一个新节点
        final Node<E> newNode = new Node<>(pred, e, succ);
        //succ前驱就是新节点
        succ.prev = newNode;
        if (pred == null)
            first = newNode;
        else
            //新节点前驱不为null，则pred后继（原succ前驱）为新节点
            pred.next = newNode;
        size++;
        modCount++;
    }
```
在指定节点添加指定元素
```java
    public void add(int index, E element) {
        //下标检查
        checkPositionIndex(index);
        //如果添加下标刚好在元素最后则，尾部添加
        if (index == size)
            linkLast(element);
        else
            //否则则在指定下标的节点前增加指定节点，原index节点变成新增节点的后继节点，下标为index+1
            linkBefore(element, node(index));
    }
```

### 5.元素删除

移除头结点，默认移除头结点
```java
    public E remove() {
        return removeFirst();
    }

    public E removeFirst() {
        final Node<E> f = first;
        if (f == null)
            throw new NoSuchElementException();
        return unlinkFirst(f);
    }

    private E unlinkFirst(Node<E> f) {
        // assert f == first && f != null;
        //获取删除节点数据域，以便返回
        final E element = f.item;
        //获取删除节点后继节点
        final Node<E> next = f.next;
        //gc，删除节点
        f.item = null;
        f.next = null; // help GC
        //设置原头结点的后继节点是新的头结点
        first = next;
        //节点为空则，则删除前是只有一个节点的情况
        if (next == null)
            last = null;
        else
            //该节点是头结点，前驱为null
            next.prev = null;
        //节点数量 - 1    
        size--;
        //修改数 + 1
        modCount++;
        //返回被删除元素
        return element;
    }
```

移除指定下标的节点

```java
    //传入下标，找出指定下标节点，删除节点
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }

    //移除指定节点
    E unlink(Node<E> x) {
        // assert x != null;
        //该节点数值域
        final E element = x.item;
        //该节点后继节点
        final Node<E> next = x.next;
        //该节点前驱节点
        final Node<E> prev = x.prev;
        //若前驱节点是null，则删除x节点后，x节点的后继节点变成头结点
        if (prev == null) {
            first = next;
        } else {
            //前驱节点不为null，则修改指针关系，前驱节点的后继指针指向后继节点
            prev.next = next;
            //gc，解除指针关系
            x.prev = null;
        }
        //若x的后继节点为null
        if (next == null) {
            //则删除x后，x前驱节点变为新的尾节点
            last = prev;
        } else {
            //原x后继节点的前驱指针指向x的前驱节点
            next.prev = prev;
            //解除指向关系
            x.next = null;
        }
        //归结删除逻辑：
        //前驱节点不为null，则修改指针关系，前驱节点的后继指针指向后继节点
        //prev.next = next;
        //gc，解除指针关系
        //x.prev = null;
        //原x后继节点的前驱指针指向x的前驱节点
        //next.prev = prev;
        //解除指向关系
        //x.next = null;

        x.item = null;
        //节点数量 - 1
        size--;
        //修改数 + 1
        modCount++;
        //返回被删除元素
        return element;
    }
```
移除尾节点

```java
    //移除尾节点
    public E removeLast() {
        final Node<E> l = last;
        if (l == null)
            throw new NoSuchElementException();
        return unlinkLast(l);
    }

    //移除尾节点： 被删除节点的前驱节点变成新的尾节点
    private E unlinkLast(Node<E> l) {
        // assert l == last && l != null;
        final E element = l.item;
        final Node<E> prev = l.prev;
        l.item = null;
        l.prev = null; // help GC
        last = prev;
        if (prev == null)
            first = null;
        else
            prev.next = null;
        size--;
        modCount++;
        return element;
    }
```
移除指定元素
```java
    public boolean remove(Object o) {
        if (o == null) {
            //如果要删除的节点是null，则从头结点开始找，找出null元素，删除
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
            //从这可以看出，linkedList可以存null元素
        } else {
            //不为null，则找出指定元素，比对数据域，删除
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

### 6.是否包含指定元素

是否包含指定元素
```java
    //判断是否包含指定元素，可判断该元素下标存不存在
    public boolean contains(Object o) {
        return indexOf(o) != -1;
    }
```

找出指定元素的下标
```java
    public int indexOf(Object o) {
        int index = 0;
        //若指定元素为null，则从头结点开始找，返回下标
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            //也是从头开始找，找到指定元素后返回器下标
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        //没有该元素则返回 -1
        return -1;
    }
```

从尾节点开始找指定元素，并返回器下标,思路同上
```java
    public int lastIndexOf(Object o) {
        int index = size;
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (x.item == null)
                    return index;
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                index--;
                if (o.equals(x.item))
                    return index;
            }
        }
        return -1;
    }
```

可以看出，其实linkedList在插入删除操作时间复杂度可以达到O(1),但是在查找上时间复杂度是O(N)
