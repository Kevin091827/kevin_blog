# 一，spring开启自动热部署 

在实际开发中，我们修改某些代码逻辑功能或页面都需要重启应用，这无形中降低了开发效率；热部署是指当我们修改代码后，服务能自动重启加载新修改的内容，这样大大提高了我们开发的效率；

spring boot 开启自动热部署需要添加一个插件实现，在maven下添加以下依赖

```xml
<!-- springboot 开发自动热部署 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
```

这样就完成了我们的热部署 ，但是有一点需要注意的就是，该热部署插件在实际使用中会有一些小问题，明明已经重启，但没有生效，这种情况下，手动重启一下程序；


# 二，spring boot 集成Redis

## spring boot集成Redis的步骤如下：

### 1、在pom.xml中配置相关的jar依赖
 ```xml
	    <!-- 加载spring boot redis包 -->
        <!--这是一个起步依赖，会自动加载很多相关依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

### 2、在Springboot核心配置文件application.properties中配置redis连接信息：

```application
    #配置下redis的连接信息
    spring.redis.host=
    spring.redis.port=6379
    spring.redis.password=
```
### 3.配置了上面的步骤，Spring boot将自动配置RedisTemplate，在需要操作redis的类中注入redisTemplate;

```java
    	//这个模板是spring boot自动注入的，泛型类只能是<string ,string >,<Object,Object>
        @Autowired
        private RedisTemplate<Object,Object> redisTemplate;
    
        @Override
        public List<User> selectAllUser()
        {
            //因为将数据序列化存入Redis时key也是序列化进去的，可读性不好
            //RedisSerializer是Redis的序列化器，是一个接口
            RedisSerializer redisSerializer = new StringRedisSerializer();
    
            //设置一下key的序列化器
            redisTemplate.setKeySerializer(redisSerializer);
    
            //查询缓存
            List<User> list = (List<User>)redisTemplate.opsForValue().get("alluser");
    
            //判断缓存是否为空
            if(list==null){
                //缓存为空则查询数据库
                list=userdao.selectAllUser();
                //将查询结果写进缓存
                redisTemplate.opsForValue().set("alluser",list);
            }
            return list;
        }

```
注意，因为写入缓存是需要序列化的，所以pojo包下的实体类也要实现序列化的接口

```java
    public class User implements Serializable {
        private int id;
        private String name;
    
        public int getId() {
            return id;
        }
    
        public void setId(int id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }
   ```
因为key也是序列化进缓存的，为了可读性，所以采用了Redis的序列化器，设置key的序列化
```java
            //因为将数据序列化存入Redis时key也是序列化进去的，可读性不好
            //RedisSerializer是Redis的序列化器，是一个接口
            RedisSerializer redisSerializer = new StringRedisSerializer();
    
            //设置一下key的序列化器
            redisTemplate.setKeySerializer(redisSerializer);

```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181224201211232.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
这就是区别（第三个和第四个，第三个是采用了序列化器的）

### 4，测试工作
#### 1.先开启Redis服务
#### 2.测试
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122420132259.png)

成功写入缓存
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181224201513599.png)