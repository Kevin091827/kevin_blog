# 数据库专题复习（mysql约束，事务和视图）



## 一，MySql约束

> 约束的作用：其实约束就是相当于一种限制，用于限制表中的数据，为了保证表中的数据的准确性和可靠性

### 1，MySql六大约束



 #### 非空约束

> 非空约束，就是我们平常常见到的not null，该列数据不能为空，必须给定数据

例如我们在创建表时，可以通过界面操作，指定非空约束

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016134004874.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

又或者：在指定字段后边添加NOT NULL指定非空约束

```sql
create table table_NAME{
	id int(10) NOT NULL
}
```

添加了非空约束后，如果我们在插入或者更新数据时，设定了非空约束的字段如果传的值是空的那么将会插入或者更新失败。



#### 唯一约束

> 唯一约束就是unique key，保证了指定字段数据的唯一性，但是该字段可以为null

创建唯一约束：

```sql
create table table_name{
	id int(10) unique
}
```

##### 唯一约束和唯一索引的异同

- 共同点：
  - 两者都可以使指定字段的数据值唯一，该字段也可以为空
- 不同点：
  - 创建唯一约束，会自动创建一个同名的唯一索引，这个被自动创建的唯一索引不能独立删除，要想删除该唯一索引，则需要删除约束，唯一约束删除，唯一索引自动删除，唯一约束也是通过唯一索引来保证数据唯一性的
  - 创建一个唯一索引，这个手动创建的唯一索引就是独立的，可以单独删除
  - 如果希望一个列上的唯一索引和唯一约束都是独立的，可以先创建唯一索引，在创建唯一约束

#### 主键约束

> 主键约束相当于非空约束 + 唯一约束，可以唯一的标识该字段，且该主键字段不能为空，但是主键与刚刚提到的那两个约束又有区别，主键除了可以做到 not null and unique之外，默认还会添加索引，提高查询效率

```sql
create table table_name{
	id int(10) primary key
}
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016140105470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

在插入数据时，如果检查到要插入的数据的主键值在表中已经存在，那么此时就会插入失败。

> mysql为主键提供了自动递增的功能

在MySQL数据库提供了一个自增的数字，专门用来自动生成主键值，主键值不用用户维护，自动生成，自增数从1开始，以1递增(auto_increment)

### ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016140548253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)



#### 外键约束

> 外键约束主要用来维护两个表之间数据的一致性。

举个例子：

> 有个学生信息表：

![1571206813061](C:\Users\12642\AppData\Roaming\Typora\typora-user-images\1571206813061.png)

> 有个班级表:

![blogblog](https://img-blog.csdnimg.cn/20191016141939880.png)

为了保证学生信息表中的class字段的数据都来自于班级表，有必要给学生表中class字段添加外键约束

```sql
CREATE TABLE `tb_stu` (
  `id` int(11) NOT NULL,
  `stu_name` varchar(255) NOT NULL,
  `class` int(255) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `class_fk` (`class`),
  CONSTRAINT `class_fk` FOREIGN KEY (`class`) REFERENCES `tb_class` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20191016142133495.png)

那么添加了外键约束的表就是从表，主表就是班级表

> 再来看看，为什么不将两个表的数据整合起来做成一个大表呢？

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019101614240130.png)

> 原因：数据冗余，class_name数据重复太多，而且，建立了外键约束后，默认会建立一个同名的普通索引

![1571207199655](C:\Users\12642\AppData\Roaming\Typora\typora-user-images\1571207199655.png)

注意：

- 外键值可以为null
- 外键字段去引用一张表的某个字段的时候，被引用的字段必须具有unique约束
- 有了外键引用之后，表分为主表和从表
  - 创建时先创建主表
  - 删除时先删除从表数据
  - 插入时先插入主表数据

#### 默认约束

> 默认约束就是给指定的字段列设置一个默认值

指定用户表中的性别默认是男

```sql
CREATE TABLE `test`.`user`(  
  `id` INT(11) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `name` VARCHAR(225) COMMENT '姓名',
  `sex` TINYINT(1) DEFAULT 1 COMMENT '性别 1男 0女',
  PRIMARY KEY (`id`)
)
```

#### 总结

| 约束     |                             作用                             |
| -------- | :----------------------------------------------------------: |
| 非空约束 |                被约束的字段列数据值不能为null                |
| 唯一约束 |          被约束的字段列数据值不能重复，但可以为null          |
| 主键约束 |       唯一的标识,用于保证该字段的值具有唯一性并且非空        |
| 外键约束 | 用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值 |
| 默认约束 |                   被约束的字段列具有默认值                   |



## 二，MySql事务



### 1，事务概念

事务，就是一组对数据库做一系列操作的过程，要么全部执行，要么全部不执行，事务是并发控制的单位。



### 2，事务ACID特性

- **原子性**

  ​	**原子性是指一个事务包含的所有操作，要么全部执行，要么全部不执行**

- **一致性：**

  ​	**一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。**

  　　拿转账来说，假设用户A和用户B两者的钱加起来一共是5000，那么不管A和B之间如何转账，转几次账，事务结束后两个用户的钱相加起来应该还得是5000，这就是事务的一致性，不会说，A的钱减少了，B的钱没有增加，状态变化对于事务中的单元来说，应该是一致的，A减少B应该增加。

- **隔离性：**

  ​	当多个用户并发访问数据库时，数据库为每一个用户开启一个事务，但是，这些并发事务之间要相互隔离，事务1不受事务2影响，或者说，**一个事务所做的修改在最终提交以前，对其他事务是不可见的。**

- **持久性：**

  ​	**一旦事务被提交，则该事务所做的修改会永久保存到数据库中，那么数据库中数据的改变就是永久性的。**



### 3，事务隔离级别

> 事务有四种隔离级别

- 读未提交

  一个事务读到另一个事务未提交的数据

- 读已提交（不可重复读）

   在同一事务中, 多次读取同一数据返回的结果有所不同,事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。

- 可重复读

   在同一事务中多次读取数据时, 能够保证所读数据一样, 也就是后续读取不能读到另一事务已提交的更新数据，但能读取到另一个事务已提交的insert数据

- 可串行化

  事务A和事务B，事务A在操作数据库时，事务B只能排队等待

> 总结：

| 隔离级别   | 脏读 | 不可重复读 | 幻读 |
| ---------- | ---- | ---------- | ---- |
| 读未提交   | 会   | 会         | 会   |
| 不可重复读 | 不会 | 会         | 会   |
| 可重复读   | 不会 | 不会       | 会   |
| 可串行化   | 不会 | 不会       | 不会 |



### 4，事务的使用

### MYSQL 事务处理主要有两种方法：

1、用 BEGIN, ROLLBACK, COMMIT来实现

- **BEGIN** 开始一个事务
- **ROLLBACK** 事务回滚
- **COMMIT** 事务确认

2、直接用 SET 来改变 MySQL 的自动提交模式:

- **SET AUTOCOMMIT=0** 禁止自动提交
- **SET AUTOCOMMIT=1** 开启自动提交

### 5.mysql存储引擎对事务的支持

innodb是mysql中支持事务存储引擎，支持事务处理和ACID事务特性，实现了sql标准的四种隔离级别

并且可以利用事务日志进行数据恢复，但是实现了ACID的存储引擎，相比没有实现ACID的存储引擎，通常需要更强的CPU处理能力，更大的内存和更多的磁盘空间。



### 6.事务死锁

事务的死锁是指，两个或者多个事务在同一个资源上相互占用，并请求锁定对方占用的资源，从而导致恶性循环的现象，当多个事务试图以不同的顺序锁定资源时，就可能会产生死锁，多个事务同时锁定同一个资源时也会产生死锁

例如：

事务1：

```sql
begin
update table1 set hight = 20 where wight = 1
update table1 set hight = 21 where wight = 2
commit
```

事务2：

```sql
begin
update table1 set hight = 20 where wight = 2
update table1 set hight = 21 where wight = 1
commit
```

两个事务都执行了第一条update语句，同时分别对wight = 1 和wight = 2这两行加了行级锁，那么当事务1和事务2继续进行下去时就会发现他们想要更新的行被锁住，两个事务都在等待对方释放锁，但是又同时持有对方需要的锁，此时则进入了死锁的状态



> 如何解决？

innoDB目前解决死锁的方法是，将持有最少行级排他锁的事务进行回滚



## 三，MySql视图



### 1，视图概念

视图，虚拟表，从一个表或多个表中导出来的表，作用和真实表一样，包含一系列带有行和列的数据 视图中，用户可以使用SELECT语句查询数据，也可以使用INSERT，UPDATE，DELETE修改记录，视图可以使用户操作方便，并保障数据库系统安全



**优点**

简单化，数据所见即所得

安全性，用户只能查询或修改他们所能见到得到的数据

逻辑独立性，可以屏蔽真实表结构变化带来的影响

**缺点**

性能相对较差，简单的查询也会变得稍显复杂

修改不方便，特变是复杂的聚合视图基本无法修改



### 2，视图使用

创建一个视图:

格式：

```sql
create view "view_name" as sql查询语句
```

示例：

```sql
create VIEW test as select * from tb_class
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191017142004649.png)

查询视图：

```sql
select * from test
```

![1571293271875](C:\Users\12642\AppData\Roaming\Typora\typora-user-images\1571293271875.png)

视图是存储在数据库中的查询的sql 语句，它主要出于两种原因：安全原因，视图可以隐藏一些数据，如：社会保险基金表，可以用视图只显示姓名，地址，而不显示社会保险号和工资数等，另一原因是可使复杂的查询易于理解和使用。相当于将复杂查询sql封装成一个函数调用，可以做到复用的效果

### 3.

