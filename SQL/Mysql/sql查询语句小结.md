今天在做作业的时候，发现自己sql语句基础并不是特别牢固，所以晚上得时候又重新看了一次查询的sql语法，复习复习基础，个人觉得查询sql语句在增删改查中个人觉得是最重要的，

**分类整理**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403012057922.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

下边操作以下列表为主

student_info表
```sql
CREATE TABLE `student_info` (
  `username` varchar(255) NOT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sex` varchar(255) DEFAULT NULL,
  `profession` varchar(255) DEFAULT NULL,
  `password` varchar(255) NOT NULL,
  `role` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=164 DEFAULT CHARSET=utf8;
```
userinfo表
```sql
CREATE TABLE `userinfo` (
  `username` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `perm` varchar(255) DEFAULT NULL,
  `role` varchar(255) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
user_tb表
```sql
CREATE TABLE `user_tb` (
  `_userName` varchar(255) DEFAULT NULL,
  `_id` int(11) NOT NULL AUTO_INCREMENT,
  `_sex` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`_id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
```

**普通查询** 

```sql
#基本语法
select * | filed_need...from table_name 

SELECT * FROM student_info 
```
**条件查询**

limit -- 返回设定数量的记录 

```sql
#返回student_info表的前五条记录
select * from student_info limit 5
```
offset -- 在查询过程中，跳过指定数量的记录，一般和limit一起使用
```sql
#跳过select * from student_info查询结果的前10条记录，返回接下来的5条记录
select * from student_info limit 5 offset 10
```

除此之外，最经常用到就是where了
```sql
# 返回id=1的记录
select * from student_info where id = 1

#返回id>10且sex性别是男的数据
select * from student_info where id>10 and sex='男'

#返回id>10或者sex是男的记录中的id列，sex列和role列
select id,sex,role from student_info where id>10 or sex ="男"

#查询id是1,2,3,4,6,7的记录
select * from student_info where id in(1,2,3,4,6,7)
#查询性别是男或者女的记录
select * from student_info where sex in('男','女')

#查询id在1到10之前的记录
select * from student_info where id between 1 and 10

# LIKE :模糊查询 一般和where搭配使用，
# 	通配符：
# 		%：任意字符 eg：%com ：代表以com结尾的任意结果
#					   %com%：代表中间字符为com的任意结果
#					   com%： 代表以com开头的任意结果
# 		_ :单一字符 eg：_1_ :代表三位且中间位是1的任意结果
#					   1__ :代表三位且第一位是1的任意结果
SELECT * FROM student_info WHERE sex LIKE '%11'
SELECT * FROM student_info WHERE sex LIKE '%11%'
SELECT * FROM student_info WHERE sex LIKE '11%'
SELECT * FROM student_info WHERE sex LIKE '_1_'
SELECT * FROM student_info WHERE sex LIKE '1__'
```
* where相当于程序语言中的if
	* and：相当于 &&
	* or：相当于 ||


**= 和 in 的区别？**

* 后接参数数量
	*  =	1个
	*  in 多个 
* in可以后接返回多个结果的一个子查询

**分组 --- 可以对重复出现的字段根据他们进行分组**
```sql
#根据性别（sex）分组
SELECT * FROM student_info GROUP BY sex
```

**排序**
```sql
# DESC 降序
SELECT * FROM student_info ORDER BY id DESC

#AES升序
SELECT * FROM student_info ORDER BY id AES
```

**UNION**

用于连接两个以上的select语句的结果组合到一个结果集合中，默认会删除结果中的重复元素
```sql
#ALL连接全部数据，不去除重复值
SELECT username FROM student_info 
UNION
 ALL 
SELECT username FROM userinfo

#DISTINCT 连接数据，去除重复值
SELECT username FROM student_info WHERE sex LIKE '1%'
UNION
DISTINCT 
SELECT username FROM userinfo
```

**连接查询**

****等值连接**---（内连接）**

读取两张表中符合的数据，即两张表的交集

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403012844156.png)

```sql
SELECT U.email,I._userName FROM `user` U INNER JOIN `user_tb` I ON U.id=I._id
```
**左连接**

读取左边数据表符合的全部数据，即便右边表无对应数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403012948577.png)

```sql
SELECT U.email,I._userName FROM `user` U LEFT JOIN `user_tb` I ON U.id=I._id
```

**右连接**

读取右边数据表的全部符合的数据，即使左表无对应数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403013046625.png)

```sql
SELECT U.email,I._userName FROM `user` U RIGHT JOIN `user_tb` I ON U.id=I._id
```