
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




