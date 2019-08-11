
# 一，mysql索引

mysql对索引的定义就是：索引是帮助高效获取数据的数据结构，

## 1.mysql中索引的使用

### mysql索引类型

**普通索引**
    
- 其实就是普通的键，能加快查询速度 

**唯一索引**

- 加快查询，但是索引列值必须唯一，可以为null

**主键索引**

- 唯一索引的特例，列值不可为null

**全文索引**

- 对文本内容进行查询时，能加快速度

**单列索引**

- 就是常见的一个列字段的索引

**组合索引**

- 多个列字段组成一个索引，在组合搜索中效率快

### mysql中的索引操作

1.创建索引
```sql
--创建普通索引
create index index_name on table_name(col_name)

--创建唯一索引
create unique index index_name on table_name(col_name)

--创建组合索引
--遵循“最左前缀”原则，把最常用作为检索或排序的列放在最左，依次递减，组合索引相当于建立了col1,col1col2,col1col2col3三个索引，而col2或者col3是不能使用索引的。
create index index_name on table_name(col_name1,col_name2,col_name3)

--创建唯一组合索引
create unique index index_name on table_name(col_name1,col_name2)
```

2.通过修改表结构创建索引
```sql
alter table table_name add index index_name(col_name) 

--创建全文索引
ALTER TABLE 'table_name' ADD FULLTEXT INDEX ft_index('col')；
```
3.删除索引
```sql
drop index index_name on table_name
```
# 二，mysql索引底层数据结构

mysql中的索引是在存储引擎中实现的而不是在服务器端实现的，所以，不同存储引擎支持的索引可能也会有所不同

我们在开头就提到，索引是mysql高效获取数据的数据结构，也就是说，索引其实归根揭底也是一种数据结构

## 1.猜想索引的数据结构

索引绝大多数情况都是用在查询操作当中，那么我们列举下我们知道的查找算法

**1. 顺序查找？**

    顺序查找复杂度是O(n)，在数据大的情况下，这种算法的查询表现是十分糟糕的

**2. 二叉树查找？**

    如果是二叉搜索树：有可能会出现深度过大的左偏树或者右偏树，这样时间复杂度就会有O(logN)变为O(N)

    如果是二叉搜索树的进化版平衡二叉树又或者红黑数呢：在数据量大的情况下，基于两个子树的二叉树还是会显得深度过大

**3.多叉树？**

    目前大部分数据库系统及文件系统都采用B-Tree或其变种B+Tree作为索引结构

## 2.基于多叉树的索引实现    

在mysql中不同存储引擎对索引的实现也不同，本文主要讨论MyISAM和InnoDB两个存储引擎的索引实现方式。

### MyIsam索引实现（非聚集索引）

MyIsam引擎使用B+树来实现索引，在B+树的叶子节点数据域中存放的mysql表数据记录的地址

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809210653928.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这里设表一共有三列，假设我们以Col1为主键，则图8是一个MyISAM表的主索引（Primary key）示意。可以看出MyISAM的索引文件仅仅保存数据记录的地址。在MyISAM中，主索引和辅助索引（Secondary key）在结构上没有任何区别，只是主索引要求key是唯一的，而辅助索引的key可以重复。如果我们在Col2上建立一个辅助索引，则此索引的结构如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809210820897.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

在进行sql查询时，会根据查询语句中的索引条件，在b+树中找到相应的叶子节点，取出相应的数据地址，然后获取相关表记录

### innerDB索引实现（聚集索引）

虽然innerDB也使用b+树作为索引结构，但是实现方式和myisam不同

在主索引中，b+树叶子节点数据域中存储的是数据库表相关列的完整记录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809211253940.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

在辅助索引中，b+树叶子节点存储的是主键的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190809211347423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这种就是聚集索引

- 聚集索引在进行主索引查询时效率很高，直接找到相关叶子节点就可以找到相关数据记录，非聚集索引还需要根据数据地址去寻找数据记录

- 但是聚集索引在辅助索引查询时效率低非聚集索引低，因为需要检索两次b+树，第一次找到主键值，第二次根据主键值找到相关记录

### 为什么要使用多叉树实现呢？

我们知道平衡的二叉查找树可以做到树的高度小于普通的二叉查找树且查询效率可以达到O(logN),但是，当在大量数据存储在磁盘时，我们并不能一下就将这些数据全部都加载到内存中，而是一页一页的加载，每一磁盘页的数据对应一个树的节点，当树的节点过多时，会造成大量磁盘IO操作，最坏情况下就是树的深度，平衡二叉树由于树深度过大而造成磁盘IO读写过于频繁，进而导致效率低下。 

所以，这就是使用平衡的多路查找树的原因，既可以降低树的高度，又能保证查询效率在O(logN)

#### 多叉树简述

##### b树

b树是最普通的一类多叉树，一个m阶的b树具有如下几个特征：

- b树中所有节点的的子节点数的最大值称为b树的阶
- 一个节点有k个孩子，那么必有k-1个关键字才能将子节点划分为k个子集
- 根结点至少有两个子女。
- 每个中间节点都包含k-1个元素和k个孩子，其中 ceil（m/2） ≤ k ≤ m
- 每一个叶子节点都包含k-1个元素，其中 ceil（m/2） ≤ k ≤ m
- 所有的叶子结点都位于同一层。
- 每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域划分
- 每个结点的结构为：（n，A0，K1，A1，K2，A2，…  ，Kn，An）
    其中，Ki(1≤i≤n)为关键字，且Ki<Ki+1(1≤i≤n-1)。
Ai(0≤i≤n)为指向子树根结点的指针。且Ai所指子树所有结点中的关键字均小于Ki+1。
n为结点中关键字的个数，满足ceil(m/2)-1≤n≤m-1。

例如：

三阶b树:最大子节点数量为3,节点关键字为2个

![](https://img-blog.csdn.net/2018032420113990?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**查询操作**

　　以上图为例：若查询的数值为５： 

　　第一次磁盘ＩＯ：在内存中定位（与17、35比较），比17小，左子树； 

　　第二次磁盘ＩＯ：在内存中定位（与８、12比较），比８小，左子树； 

　　第三次磁盘ＩＯ：在内存中定位（与3、5比较），找到5，终止。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190811112208920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

整个过程中，我们可以看出：比较的次数并不比二叉查找树少，尤其适当某一节点中的数据很多时，但是磁盘IO的次数却是大大减少。比较是在内存中进行的，相比于磁盘IO的速度，比较的耗时几乎可以忽略。所以当树的高度足够低的话，就可以极大的提高效率。相比之下，节点中的元素多点也没关系，仅仅是多了几次内存交互而已，只要不超过磁盘页的大小即可。现在重点关注的就是磁盘IO的次数

**插入操作**

若被插入节点的关键字个数 < m-1 ，则直接插入

若被插入节点的关键字个数等于m-1，则会引起该节点分裂，将节点中间值插入到该节点的父节点

重复上述工作，最坏情况一直分裂到根结点，建立一个新的根结点，整个B树增加一层。

eg：插入4

![](https://img-blog.csdn.net/20180324225635106?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

会引起节点分裂，将中间值4提到父节点,若扔引起分裂，重复操作

![](https://img-blog.csdn.net/20180324230149255?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

之所以要分裂，其实就是维护多叉树平衡的一个性质，相当于平衡树中的旋转

**删除操作**
Ｂ树中关键字的删除比插入更复杂，在这里说下跟b树插入节点类似的一种删除方法:

eg:删除11

![](https://img-blog.csdn.net/20180324232514619?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

但是删除后12只有右子节点了，需要判断其左右兄弟结点中有“多余”的关键字，即：原关键字个数n>=ceil(m/2) - 1

右子节点分裂

![](https://img-blog.csdn.net/20180324232544302?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


**补充：**

- B树主要用于文件系统以及部分数据库索引，例如： MongoDB。而大部分关系数据库则使用B+树做索引，例如：mysql数据库； 
- 从查找效率考虑一般要求B树的阶数m >= 3; 
- B-树上算法的执行时间主要由读、写磁盘的次数来决定，故一次I/O操作应读写尽可能多的信息。因此B-树的结点规模一般以一个磁盘页为单位。一个结点包含的关键字及其孩子个数取决于磁盘页的大小。


##### b+树

Ｂ＋树是Ｂ树的变种，有着比Ｂ树更高的查询效率

特征：

1. 有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据
都保存在叶子节点。

2. 所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小
自小而大顺序链接。

3. 所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

eg：三阶b+树

![](https://img-blog.csdn.net/20180325001555181?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**查找**

B+树通常有两个指针，一个指向根结点，另一个指向关键字最小的叶子结点。因些，对于B+树进行查找两种运算：一种是从最小关键字起顺序查找，另一种是从根结点开始，进行随机查找。

B+树相比B树的优势：

1. b+树中间节点存储的是数据在叶节点的索引，可以包含的关键字更多,单一节点存储更多的元素，使得查询的IO次数更少； 
2. 所有查询都要查找到叶子节点，查询性能稳定； 
3. 所有叶子节点形成有序链表，便于范围查询。

eg:范围查询

b树范围查询：

![](https://img-blog.csdn.net/20180325111523599?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

b+树：

![](https://img-blog.csdn.net/20180325111559207?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



### innerDB和myIsam的区别

|myIsam|innerDB|
|--|--|--|--|
|索引类型|非聚集索引|聚集索引|
|索引底层结构|b+树|b树|
|是否支持事务|否|是|
|是否支持表锁|是|是|
|是否支持行数|否|是|
|是否支持外键|否|是|
|是否支持全文索引|是|是|
|是否支持hash索引|否|否|
|适用操作|大量select|大量insert、delete和update下|



# 三，补充

## 哈希索引

只有memory（内存）存储引擎支持哈希索引，哈希索引用索引列的值计算该值的hashCode，然后在hashCode相应的位置存执该值所在行数据的物理位置，因为使用散列算法，因此访问速度非常快，但是一个值只能对应一个hashCode，而且是散列的分布方式，因此哈希索引不支持范围查找和排序的功能，但是hash适用于 = 或者 in的等值查询




