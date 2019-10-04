#  hibernate

> hibernate是一种orm框架，可以实现对数据库的crud操作，底层就是jdbc实现，是一个开源轻量级的框架

## 1.搭建hibernate开发环境



## 2.hibernate核心api

![](https://segmentfault.com/img/remote/1460000013568234?w=1009&h=758)





## 3.hibernate中的缓存



### 1.一级缓存

- 类似mybatis中的一级缓存，也就是session缓存，默认开启
- 一级缓存仅仅对当前session可见，不可以跨session，一旦session关闭，一级缓存失效
- session缓存是由hibernate控制，用户只能通过clear或者evit操作缓存

一级缓存相关方法：
- session.flush(); 让一级缓存与数据库同步
- session.evict(arg0); 清空一级缓存中指定的对象
- session.clear(); 清空一级缓存中缓存的所有对象

### 2.二级缓存

- 二级缓存和mybatis中的二级缓存也很相似
- 二级缓存是基于应用程序的缓存，所有的Session都可以使用
- 二级缓存默认
- Hibernate提供的二级缓存有默认的实现，且是一种可插配的缓存框架，如果用户觉得hibernate提供的框架框架不好用，自己可以换其他的缓存框架或自己实现缓存框架都可以。

### 缓存架构

![](https://segmentfault.com/img/remote/1460000013614494?w=1065&h=465)


##  4.HQL和QBC查询

# JPA

> JPA的出现主要是为了简化持久层开发以及整合ORM技术，结束Hibernate、TopLink、JDO等ORM框架各自为营的局面。JPA是在吸收现有ORM框架的基础上发展而来，易于使用，伸缩性强。总的来说，JPA包括以下3方面的技术：
>
> - **ORM映射元数据**： 支持XML和注解两种元数据的形式，元数据描述对象和表之间的映射关系
> - **API**： 操作实体对象来执行CRUD操作
> - **查询语言**： 通过面向对象而非面向数据库的查询语言（`JPQL`）查询数据，避免程序的SQL语句紧密耦合

## 1.jpa和hibernate的关系

jpa是java持久化api，是一种规范，hibernate是jpa的一种实现

## 2.jpa基础知识总结（基于注解版开发）

### 1. Repository接口及其实现类说明

### 2.JPQL

### 3.多表映射

##### 单向多对一

> 例如：一个用户可以有多个订单，但是一个订单只能属于一个用户



##### 单向一对多

> 例如：

##### 双向一对一

> 例如：公司停车位和员工，公司停车位一次能为一个员工停车，所以，一个停车位

##### 双向一对多



##### 双向多对多









## 3. springboot2.x整合JPA




