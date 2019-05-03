# Spring Boot高级技术------缓存

springboot版本1.5

## 缓存简单介绍，应用

缓存是分布式系统中的重要组件，主要解决高并发，大数据场景下，热点数据访问的性能问题。提供高性能的数据快速访问。主要作为中间件被应用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121005811250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

缓存媒介：

常用中间件：Varnish，Ngnix，Squid，Memcache，Redis，Ehcache等；

缓存的内容：文件，数据，对象；

缓存的介质：CPU，内存（本地，分布式），磁盘（本地，分布式）

意义：

应用系统需要通过 Cache 来缓存不经常改变的数据以提高系统性能和增加系统吞吐量，避 免直接访问数据库等低速的存储系统。缓存的数据通常存放在访问速度更快的内存中或者是低 延迟存取的存储器、服务器上。 应用系统缓存通常有以下作用： 缓存 Web 系统的输出，如伪静态页面。 缓存系统中不经常改变的业务数据，如用户权限、字典数据、配置信息等。

## 1.JSR-107

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121010112370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![!\[在这里插入图片描述\](https://img-blog.csdnimg.cn/20190121010352877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70](https://img-blog.csdnimg.cn/20190121012958699.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

除此之外，还有

* Cache Hit，从 Cache 中取得期望的缓存项，我们通常称之为缓存命中。如果没有命中 则称之为 Cache Miss，意味着需要从数据来源处重新取出井放回 Cache 中。
* Cache Miss，缓存丢失，根据 Key 没有从缓存中找到对应的缓存项。
* Cache Evication，缓存清除操作。 
 *  Hot Data，热点数据，缓存系统能调整算法或者内部存储方式，使得最有可能频繁访 问的数据能被尽快访问到。
   * On-Heap, Java 分配对象都是在堆内存中，有最快的获取速度。由于虚拟机的垃圾回 收管理机制，缓存放入过多的对象会导致垃圾回收时间过长，从而有可能影响性能。
   *  0ff -Heap，堆外内存，对象存放在虚拟机分配的堆外内存中，因此不受垃圾回收管理 机制的管理，不影响系统性能，但堆外内存的对象要被使用，还要序列化成堆内对象。 很多缓存工具会把不常用的对象放到堆外，把热点数据放到堆内 。

## 2.spring boot 缓存抽象技术

 Spring Boot 应用通过注解的方式使用统一的缓存，只需在方法上使用缓存注解即可，其缓存的具体实现依 赖于选择的目标缓存管理器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121013014968.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121013106100.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121013402413.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121013340856.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

```java
        @Repository
        @Mapper
        public interface StudentDao {
        
            @Select("select * from student_info where id=#{id}")
            Student selectAllStudent(int id);
        
            @Update("update student_info set username = #{stu.username} where id = #{stu.id}")
            void update (@Param("stu") Student student);
        
            @Delete("DELETE from student_info where id = #{id}")
            void delete(int id );
            
        }
  ```
   service层
    
 ```java
     //@CacheConfig(cacheNames = "student")
        @Service
        public class StudentService {
        
            @Autowired
            StudentDao studentDao;
        
            /**
             * @Cacheable
             * cacheNames：指定缓存组件的名字，将方法返回值放入缓存，是数组类型，可以指定多个缓存
             * key：缓存数据所用到的key，默认是方法参数值，但也可以指定使用spel表达式
             * keyGenerator:key的生成器，可以自定义
             * cacheManager:指定缓存管理器
             * condition:指定符合情况的条件下才缓存   condition = "#id>2" id>2的情况下才缓存，可以写很多判断条件
             * unless:排除某些缓存情况，unless = "#id==4" id=4的情况下不缓存
             * sync ：异步，默认是false，异步情况下不支持 unless
             *
             * 先访问缓存，没有缓存才执行方法访问数据库，执行完方法后存入缓存
             * @param id
             * @return
             */
            @Cacheable(cacheNames = "student",key = "#id"/*key = "#root.method.name+'['+#id+']'"*/,condition = "#id>2",unless = "#id==4")
            public Student select(int id){
                Student student = studentDao.selectAllStudent(id);
                System.out.println("查询"+id+"号学生");
                return student;
            }
        
            /**
             * @CachePut
             * 既调用方法也更新缓存，用于新增或者修改
             * 先调用目标方法，将目标方法的返回结果存入缓存
             * @param id
             * @return
             */
            @CachePut(cacheNames = "student",key = "#id")
            public Student testCachePut(int id){
                Student student = new Student();
                student.setUsername("kidD");
                student.setId(4);
                studentDao.update(student);
                return student;
            }
        
            /**
             * @CacheEvict
             * 缓存删除，数据库数据记录删除后，原先的缓存也被删除
             * allEntries = true:清除所有的缓存
             * beforeInvocation = true/false :缓存的清除默认是在方法之后执行（false）
             * @param id
             * @return
             */
            @CacheEvict(cacheNames = "student",key = "#id")
            public void testCacheEvict(int id){
                //studentDao.delete(id);
                System.out.println("删除"+id+"的缓存");
            }
        }
```
controller层
 ```java

       @RestController
        public class StudentController {
        
            @Autowired
            StudentService studentService;
        
        @GetMapping("/select/{id}")
        public Object select(@PathVariable("id") int id){
            return studentService.select(id);
        }
    
        @GetMapping("/update/{id}")
        public Object update(@PathVariable("id") int id){
            return studentService.testCachePut(id);
        }
    
        @GetMapping("/delete/{id}")
        public Object delete(@PathVariable("id") int id){
            studentService.testCacheEvict(id);
            return "success";
        }
    }
```
  pom.xml
   ```xml			

   			    <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-cache</artifactId>
                </dependency>
```
## 3.spring boot整合redis
### 1.导入依赖

```xml
   		    <!-- 加载spring boot redis包 -->
            <!--这是一个起步依赖，会自动加载很多相关依赖-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-redis</artifactId>
            </dependency>

```
### 2.配置redis

```shell
    #配置下redis的连接信息
    spring.redis.host=127.0.0.1
    spring.redis.port=6379
    spring.redis.password=123456
    #默认是Simple，Simple适合单体或者开发环境使用
    spring.cache.type=redis
```
StringRedisTemplate 是 Spring Boot 默认提供的 Redis 操作接口 ， 适合 Key 和 Value 都是字 符串的情况，这种形式对于 Redis 来说（相对于 Redis 支持的二进制 Key-Value），容易阅读。 Spring Boot 也可以采用 JDK 序列化的方式来序列化 Key 和 Value 的 RedisTemplate 类，这 是一个通用类， 其实现可以提供不同的序列化方式。Redis允许key和value都是任意的二进制形式，我认为最好还是使用字符串作为key-value形式更好，这样通过redis客户端可以方便我们查看和管理，json也是一种不错的序列化方式，默认的序列化器是jdk的序列器，可读性不佳，可以自定义序列化器和CacheManager

通过查看源码：发现StringRedisTemplate继承RedisTemplate

```java
    public class StringRedisTemplate extends RedisTemplate<String, String> {
        public StringRedisTemplate() {
            RedisSerializer<String> stringSerializer = new StringRedisSerializer();
            this.setKeySerializer(stringSerializer);
            this.setValueSerializer(stringSerializer);
            this.setHashKeySerializer(stringSerializer);
            this.setHashValueSerializer(stringSerializer);
        }
```
在看看RedisTemplate
```java
     private RedisSerializer<?> defaultSerializer;
     if (this.defaultSerializer == null) {
                this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
            }
默认的序列化是采用jdk的序列化，因此，自定义序列化策略只需要修改defaultSerializer 


    @Configuration
    public class RedisConfig {
        /**
         * 修改redis序列化方式为json
         * 自定义cacheManager
         * @param redisConnectionFactory
         * @return
         * @throws UnknownHostException
         */
    
       //修改序列化方式
    @Bean
    public RedisTemplate<Object,Student> redisTemplate(RedisConnectionFactory redisConnectionFactory)throws UnknownHostException {
        RedisTemplate<Object,Student> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        template.setDefaultSerializer(new Jackson2JsonRedisSerializer<Student>(Student.class));
        return template;
    }

    //自定义缓存管理器
    @Bean
    public RedisCacheManager cacheManager(RedisTemplate<Object, Student> redisTemplate){
        RedisCacheManager redisCacheManager = new RedisCacheManager(redisTemplate);
        //使用前缀，将cacheName作为前缀
        redisCacheManager.setUsePrefix(true);
        return redisCacheManager;
    }
    }
```



