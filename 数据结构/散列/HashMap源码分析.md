# 哈希表

**简介**

哈希表，也叫作散列表，是一种基于快速存取所设计的一种数据结构，也是一种经典的以空间获取时间的做法，该数据结构可以理解为一个线性表，其中的元素不是紧密排列，而是可能存在空隙

哈希表是根据关键字直接访问的数据结构，我们可以通过哈希函数将要存放的元素映射为对应哈希表一个地址的关键字，根据关键字获取元素的值

**哈希冲突**

当不同元素映射道散列的同一位置时，称为哈希冲突

**解决冲突**

- 分离链接法

- 开放地址法

    - 再散列
    - 双散列
    - 平方取中法
    - 折叠法

# HashMap源码分析

## 底层数据结构

HashMap底层是数组，链表和红黑树实现的一个复杂数据结构

对于每一个元素，在map中是以键值对Entry<K,V>的形式存储
```java
    static class Node<K,V> implements Map.Entry<K,V> {
        //哈希值
        final int hash;
        //键
        final K key;
        //值
        V value;
        //指向下一元素的指针
        Node<K,V> next;
    }    
```
那么整一个散列表就是一个Entry<K,V>的线性集合
```java
transient Node<K,V>[] table;
```
当出现哈希冲突时，会使用分离链接法，当链表长度大于等于8个节点时，会变成红黑树

大概结构示意图：

![](https://user-gold-cdn.xitu.io/2017/8/19/57636de3b826c76e8d05c53e27afad7d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## 属性

```java
//负载因子，默认是0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//最大容量 2的30次方
static final int MAXIMUM_CAPACITY = 1 << 30;

//加载因子，用于计算哈希表元素数量的阈值。  threshold = 哈希表.length * loadFactor;
final float loadFactor;

//哈希表内元素数量的阈值，当哈希表内元素数量超过阈值时，会发生扩容resize()。
int threshold;
```


## 构造方法

指定负载因子和初始容量构造hashmap

- 初始容量不能小于0
- 负载因子必须大于0
```java
    public HashMap(int initialCapacity, float loadFactor) {
        //初始容量不能小于0
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        //如果初始容量大于最大容量则，初始容量就赋值为最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        //判断负载因子是否<=0，是否为空     
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
         //设置阈值为  》=初始化容量的 2的n次方的值
        this.threshold = tableSizeFor(initialCapacity);
    }
```
指定初始容量构造
```java
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```
默认构造，初始负载因子是0.75f
```java
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```
## put方法

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
**实际的添加函数**

关于onlyIfAbsent参数：默认是false，即每一次执行put，都会覆盖原来key对应的value

```java
        Map<Integer,Integer> map = new HashMap<>();
        map.put(1,1);
        System.out.println(map.get(1));
        map.put(1,2);
        System.out.println(map.get(1));
```
输出：
```
1
2
```

**hashmap扩容机制**


扩容函数
```java
    final Node<K,V>[] resize() {
        //oldTab 为当前表的哈希桶
        Node<K,V>[] oldTab = table;
        //当前哈希表的容量 length
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        //当前的阈值
        int oldThr = threshold;
        //初始化新的容量和阈值为0
        int newCap, newThr = 0;
        //如果当前容量大于0
        if (oldCap > 0) {
            //如果当前容量已经到达上限
            if (oldCap >= MAXIMUM_CAPACITY) {
                //则设置阈值是2的31次方-1
                threshold = Integer.MAX_VALUE;
                //同时返回当前的哈希表长度，不再扩容
                return oldTab;
            }else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                //否则，新容量 = 旧容量 * 2
                //   新阈值 = 旧阈值 * 2
                newThr = oldThr << 1; // double threshold
        }
        // 当前容量 <= 0 如果当前阈值 > 0,这种情况就是当前表是空表，但是但是有阈值
        else if (oldThr > 0) // initial capacity was placed in threshold
            //那么新容量 = 旧的阈值
            newCap = oldThr;
        //当前表是空的，而且也没有阈值。代表是初始化时没有任何容量/阈值参数的情况 
        else {               // zero initial threshold signifies using defaults
            //新容量 = 默认容量16
            newCap = DEFAULT_INITIAL_CAPACITY;
            //新阈值 = 默认容量16 * 默认负载因子0.75 = 12 
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        //如果新阈值等于0
        if (newThr == 0) {
            //就根据新表容量 * 负载因子得出新阈值
            float ft = (float)newCap * loadFactor;
            //新阈值如果是新容量大于最大容量则使用整形最大数
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        //更新阈值
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //根据新的容量构造新的哈希表
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //更新哈希表为新表    
        table = newTab;
        //旧表不是空表的情况，要将当前哈希桶中的所有节点转移到新的哈希桶中
        if (oldTab != null) {
            //遍历旧表
            for (int j = 0; j < oldCap; ++j) {
                //存储节点
                Node<K,V> e;
                //当前对应哈希地址有元素，赋值给e
                if ((e = oldTab[j]) != null) {
                    //旧哈希表gc
                    oldTab[j] = null;
                    //该哈希地址链表只有一个元素
                    if (e.next == null)
                        //直接根据哈希地址赋值给新表指定位置
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        //当前哈希地址链表超过8个节点，要转成红黑树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        //当前链表少于8个元素
                        //因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位=  low位+原哈希表容量
                        //低位头尾节点
                        Node<K,V> loHead = null, loTail = null;
                        //高位头尾节点
                        Node<K,V> hiHead = null, hiTail = null;
                        //下一个节点指针
                        Node<K,V> next;
                        do {
                            next = e.next;
                            // 利用哈希值 与 旧的容量，可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位
                            if ((e.hash & oldCap) == 0) {
                                //尾节点加入
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                //尾节点加入
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                            //循环到链表结束
                        } while ((e = next) != null);
                        if (loTail != null) {
                            //将低位链表存放在原index处即（j），
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            //高位存放在新index处（j+旧容量）
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        //返回新表
        return newTab;
    }
```
**扩容流程图：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190822170046298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

再回到具体的添加函数

```java
    //onlyIfAbsent:true：那么不会覆盖相同key的值value                   ，false：则会覆盖相同key的值value
    //如果evict是false。那么表示是在初始化时调用的
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        // tab：当前散列表
        // p：新增的节点
        // n：散列表长度
        // i：散列表地址（数组下标）          
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //当前的散列表示空表
        if ((tab = table) == null || (n = tab.length) == 0)
            //空表需要初始化，初始化后将散列表长度赋值给n
            n = (tab = resize()).length;
        // 判断当前新增的节点插入时是否存在哈希冲突    
        if ((p = tab[i = (n - 1) & hash]) == null)
            //没有发生哈希冲突，直接构造节点
            tab[i] = newNode(hash, key, value, null);
        else {
            //发生哈希冲突
            Node<K,V> e; K k;
            //哈希值一样，key一样则更新为新节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                // 红黑树的情况
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //链表的情况
                //直接遍历链表，添加到链表尾节点
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //找到尾节点，构造新节点
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //如果当前链表节点数超过或等于8，则转为红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    //找到了要覆盖的节点
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    //原来没有这个节点就添加到链表的尾节点    
                    p = e;
                }
            }
            //如果该节点原来已经存在，则更新value，返回原来的value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

**添加流程**


![在这里插入图片描述](https://img-blog.csdnimg.cn/2019082217093154.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


哈希函数

- key可以为null，当为null时，哈希值是0
- key的hash值，并不仅仅只是key对象的hashCode()方法的返回值，还会经过扰动函数的扰动，以使hash值更加均衡。
- key的哈希值 = key.hashCode() 对 哈希表长度取余
```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```


## get方法

```java
    public V get(Object key) {
        Node<K,V> e;
        //找不到就返回null，否则返回对应value
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }
```

真正get方法

```java
    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
                //不是空表且存在该节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                //判断链表第一个节点哈希值和key是否匹配，匹配则返回，成功找到
                return first;
            if ((e = first.next) != null) {
                //否则就遍历链表
                if (first instanceof TreeNode)
                    //从红黑树中找
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    //遍历前8个节点，找到就返回
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        //找不到就返回null
        return null;
    }
```

从此可以看出，key可以为null，value也可以为null

所以，基于这个方法，也可以用于封装判断有无相关key
```java
    public boolean containsKey(Object key) {
        return getNode(hash(key), key) != null;
    }
```

java8新增，带默认值的get方法
以key为条件，找到了返回value。否则返回defaultValue
```java    
    @Override
    public V getOrDefault(Object key, V defaultValue) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? defaultValue : e.value;
    }
```

## remove方法

```java
    //以key为条件
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hash(key), key, null, false, true)) == null ?
            null : e.value;
    }

    //key和value为条件
    public boolean remove(Object key, Object value) {
        //这里传入了value 同时matchValue为true
        return removeNode(hash(key), key, value, true, true) != null;
    }
```


真正的remove

- 从哈希表中删除某个节点， 如果参数matchValue是true，则必须key 、value都相等才删除。
- 如果movable参数是false，在删除节点时，不移动其他节点

```java
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        // p 是待删除节点的前置节点
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        //如果哈希表不为空，则根据hash值算出的index下 有节点的话。
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            //node是待删除节点
            Node<K,V> node = null, e; K k; V v;
            //如果链表头的就是需要删除的节点
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;//将待删除节点引用赋给node
            else if ((e = p.next) != null) {//否则循环遍历 找到待删除节点，赋值给node
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            //如果有待删除节点node，  且 matchValue为false，或者值也相等
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)//如果node ==  p，说明是链表头是待删除节点
                    tab[index] = node.next;
                else//否则待删除节点在表中间
                    p.next = node.next;
                ++modCount;//修改modCount
                --size;//修改size
                afterNodeRemoval(node);//LinkedHashMap回调函数
                return node;
            }
        }
        return null;
    }
```

红黑树还没看，之后再写