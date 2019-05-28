# 一，初始Redis
## 什么是Redis？

### 简介
>Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

### 特点
* 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。

* 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。

* 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。

* 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## Redis安装
redis的安装可以参考 [菜鸟教程](https://www.runoob.com/redis/redis-install.html)


在Redis安装完成之后，便可以启动redis

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190527104039475.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


# 二，初始Jedis

## 什么是jedis

jedis是redis官方推荐的一个面向java的客户端，java程序可以通过jedis操作redis，jedis对redis的很多命令都进行了封装，方便开发者调用

## 通过jedis连接redis

导包
```xml
        <!-- https://mvnrepository.com/artifact/redis.clients/jedis -->
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.0.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-pool2 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>2.5.0</version>
        </dependency>
```

简单获取Jedis连接

```java
    /**
     * 连接Jedis --- 单实例连接
     * @param ip
     * @param port
     */
    public Jedis connectToJedis(String ip,int port){
        Jedis jedis = new Jedis(ip,port);
        System.out.println("Jedis连接成功！");
        return jedis;
    }

```

基于连接池获取Jedis连接

基于连接池的连接是基于commons-pool实现

```java
    /**
     * 配置jedis连接池
     * @return
     */
    @Bean
    public JedisPoolConfig getJedisPoolConfig(){
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        //连接池中最大连接数
        jedisPoolConfig.setMaxTotal(10);
        //连接池中最大空闲连接数
        jedisPoolConfig.setMaxIdle(5);
        return jedisPoolConfig;
    }

    /**
     * 基于连接池获取jedis连接
     * @param ip
     * @param port
     * @return
     */
    public Jedis getJedisFromJedisPool(String ip,int port){
        JedisPool jedisPool = new JedisPool(getJedisPoolConfig(),ip,port);
        Jedis jedis = jedisPool.getResource();
        System.out.println("获取Jedis连接");
        return jedis;
    }
```


