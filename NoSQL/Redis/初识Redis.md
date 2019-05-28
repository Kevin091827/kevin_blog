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

# 三，Redis数据类型

## String字符串类型

>常用命令: set,get,decr,incr,mget 等。

String数据结构是简单的key-value类型，value其实不仅可以是String，也可以是数字。 常规key-value缓存应用； 常规计数：微博数，粉丝数等


## hash哈希键值对类型

>常用命令： hget,hset,hgetall 等。

hash 是一个 string 类型的 field 和 value 的映射表，hash 特别适合用于存储对象，后续操作的时候，你可以直接仅仅修改这个对象中的某个字段的值。 比如我们可以 hash 数据结构来存储用户信息，商品信息等等。比如下面我就用 hash 类型存放了我本人的一些信息：
```json
key=JavaUser293847
value={
  “id”: 1,
  “name”: “SnailClimb”,
  “age”: 22,
  “location”: “Wuhan, Hubei”
}
```

## list表类型
>常用命令: lpush,rpush,lpop,rpop,lrange等

list 就是链表，Redis list 的应用场景非常多，也是Redis最重要的数据结构之一，比如微博的关注列表，粉丝列表，消息列表等功能都可以用Redis的 list 结构来实现。

Redis list 的实现为一个双向链表，即可以支持反向查找和遍历，更方便操作，不过带来了部分额外的内存开销。

另外可以通过 lrange 命令，就是从某个元素开始读取多少个元素，可以基于 list 实现分页查询，这个很棒的一个功能，基于 redis 实现简单的高性能分页，可以做类似微博那种下拉不断分页的东西（一页一页的往下走），性能高。


## Set集合类型

>常用命令： sadd,spop,smembers,sunion 等

set 对外提供的功能与list类似是一个列表的功能，特殊之处在于 set 是可以自动排重的。

当你需要存储一个列表数据，又不希望出现重复数据时，set是一个很好的选择，并且set提供了判断某个成员是否在一个set集合内的重要接口，这个也是list所不能提供的。可以基于 set 轻易实现交集、并集、差集的操作。

比如：在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis可以非常方便的实现如共同关注、共同粉丝、共同喜好等功能。这个过程也就是求交集的过程，具体命令如下：

>sinterstore key1 key2 key3     将交集存在key1内

## ZSet有序集合类型

>常用命令： zadd,zrange,zrem,zcard等
和set相比，sorted set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列。

举例： 在直播系统中，实时排行信息包含直播间在线用户列表，各种礼物排行榜，弹幕消息（可以理解为按消息维度的消息排行榜）等信息，适合使用 Redis 中的 Sorted Set 结构进行存储。

# 四，通过jedis操作redis

jedis封装了很多redis命令，可以参考我的github，暂时只是写了jedis那部分，如果觉得对读者有帮助，可以点个小星星支持一下哈，我也很荣幸能帮到有缘人

[github](https://github.com/Kevin091827/Java_Redis_Summary)
