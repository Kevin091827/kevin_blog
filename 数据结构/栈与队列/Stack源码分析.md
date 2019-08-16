之前我们看过Vector，知道Vector是基于数组实现的，本文我们来看看Stack（栈）这种数据结构

栈是一种先进后出的数据结构，可以基于数组实现，也可以基于链表实现

java集合中的Stack是基于Vector实现的,对于Vector可以具体看[Vector源码分析](https://blog.csdn.net/weixin_41922289/article/details/99680701)
```java
public
class Stack<E> extends Vector<E> 
```
# 底层数据结构

因为是基于Vector实现，是Vector的子类，所以同样，Stack的底层数据结构是数组
```java
public
class Stack<E> extends Vector<E> 

protected Object[] elementData;
```
# 构造方法

只有一个默认的无参构造方法，调用构造一个空栈
```java
    public Stack() {
    }
```

# 进栈
进栈操作操作的数组的末端，数组的末端相当于栈顶
```java
    public E push(E item) {
        addElement(item);

        return item;
    }
```
调用的是vector的添加元素的方法，默认是数组末端添加
```java
    public synchronized void addElement(E obj) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = obj;
    }
```

# 获得栈顶元素

peek()方法是线程安全的，也使用到了同步关键字，通过数组下标直接定位到栈顶元素，从而获取
```java
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        //通过数组下标直接定位到栈顶元素    
        return elementAt(len - 1);
    }

    //返回当前数组元素个数，这是Vector的API，在这里是栈中元素个数
    public synchronized int size() {
        return elementCount;
    }

    //Vector的API，根据下标获取值
    public synchronized E elementAt(int index) {
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " + elementCount);
        }

        return elementData(index);
    }
```
# 出栈
也是调用Vector的API直接点根据下标删除栈顶元素
```java
    public synchronized E pop() {
        E       obj;
        int     len = size();

        //获取栈顶元素,返回用
        obj = peek();
        //根据下标删除栈顶元素
        removeElementAt(len - 1);

        return obj;
    }

    //也是调用Vector的API直接点根据下标删除栈顶元素
    public synchronized void removeElementAt(int index) {
        modCount++;
        if (index >= elementCount) {
            throw new ArrayIndexOutOfBoundsException(index + " >= " +
                                                     elementCount);
        }
        else if (index < 0) {
            throw new ArrayIndexOutOfBoundsException(index);
        }
        int j = elementCount - index - 1;
        if (j > 0) {
            System.arraycopy(elementData, index + 1, elementData, index, j);
        }
        elementCount--;
        elementData[elementCount] = null; /* to let gc do its work */
    }
```
# 查询元素在栈中位置（下标）

```java
    public synchronized int search(Object o) {
        //从后往前找，在这里可以看做从栈顶往栈底
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }

    //第二个参数是查询的开始下标
    public synchronized int lastIndexOf(Object o) {
        return lastIndexOf(o, elementCount-1);
    }

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
上面就是jdk中Stack的源码，Stack在jdk中是基于顺序存储结构的数组实现的，也可以基于双向链表的LinkedList实现，下边看看如何用LinkedList实现栈

# linkedList实现栈

## 底层数据结构

```java
    List<Integer> list;

    /** Initialize your data structure here. */
    public MyStack() {
       list = new LinkedList<>();
    }
```

## 进栈
进栈操作中，Vector操作的是数组末端，那么linkedList也很类似，操作的是尾节点，链表末端就是栈顶
```java
    /** Push element x onto stack. */
    public void push(int x) {
        list.add(x);
    }

    public boolean add(E e) {
        linkLast(e);
        return true;
    }
```
## 出栈

```java
    public int pop() {
       int length = list.size();
       return list.remove(length);
    }
    
    //找到相关节点后，删除节点，解除链接关系
    public E remove(int index) {
        checkElementIndex(index);
        return unlink(node(index));
    }
```

## 获得栈顶元素

```java
    public int top() {
        int length = list.size();
        return list.get(length);
    }

    //通过下标找到相关节点后获取节点值
    public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
```

## 是否是空栈
```java
    public boolean empty() {
        return list.isEmpty();
    }
```
