# spring boot集成mybatis以及配置事务


## 一，spring boot集成mybatis

之前我们是使用idea spring项目来创建spring boot项目，今天我们用第二种方式来创建下spring boot项目，我们首先创建一个maven项目，然后慢慢一步一步将他改造成一个spring boot项目

### 第一步，先将集成mybatis的一些依赖加进去

   ```xml
    <dependencies>

        <!--spring Boot 开发web项目的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--测试的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- 加载mybatis整合springboot -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>

        <!-- MySQL的jdbc驱动包 -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

    </dependencies>

```

但是因为此前是继承了父级依赖，所以我们也要把父级依赖添加进去

```xml
	<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.7.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

### 第二步，我们还需要手动创建一个application.properties的springboot的核心配置文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122321455562.png)

接下来在application.properties下配置数据库连接和mapper的路径

```application
    #mapper文件路径
    mybatis.mapper-locations=classpath:com/springboot/mapper/*.xml
    
    #配置数据库连接的相关信息
    spring.datasource.username=root
    spring.datasource.password=123
    spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=GMT%2B8
    spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

注意，jdbc版本在8.0及其之上时，如果不配置时间区间serverTimezone=GMT%2B8的话，在使用mybatis时将会报一下错误：

```shell
    Exception in thread "main" java.sql.SQLException: The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone. You must configure either the server or JDBC driver (via the serverTimezone configuration property) to use a more specifc time zone value if you want to utilize time zone support.
  ```

### 第三步，写一个springboot程序的入口main方法

```java
    @SpringBootApplication
    public class application {
        public static void main(String[] args) {
            SpringApplication.run(application.class,args);
        }
    }
 ```


application类应该包含子包

### 第四步，完善工程代码

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223215146215.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

User实体类

```java
    package com.springboot.pojo;
    public class User {
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
dao层
```java
    /**
     * 添加了@Mapper注解之后这个接口在编译时会生成相应的实现类
     * 
     * 需要注意的是：这个接口中不可以定义同名的方法，因为会生成相同的id
     * 也就是说这个接口是不支持重载的
     */
        @Repository
        @Mapper
        public interface UserDao {
        
            List<User> getuser();
        }
 ```

如果不在dao层使用@mapper注解，也可以在main方法使用@MapperScan注解并且配置要扫描的包即dao接口所在的包

```java
    @MapperScan("com.springboot.dao")
```

Service层

```java
    public interface UserService {
    
        List<User> getuser();
    }
 ```

Service实现类

```java
    @Service("UserService")
    public class UserServiceImpl implements UserService {
    
        @Autowired
        private UserDao userDao;
    
        @Override
        public List<User> getuser() {
            return userDao.getuser();
        }
    }
```

Controller层

```java
    @RestController
    public class MybatisController {
    
        @Autowired
        private UserService userService;
    
        @GetMapping("/boot/getuser")
        public Object getUser(){
            return userService.getuser();
        }
    }
```

### 第五步 写mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>

    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.springboot.dao.UserDao">
        <!-- 查询所有用户信息 -->
        <select id="getuser" resultType="com.springboot.pojo.User">
    		SELECT
    		* FROM  user
    	</select>
    
    </mapper>
```

### 第六步，配置文件的读取

这一步十分地关键，如果不进行资源文件的配置，会造成mapper.xml文件没有被加载到classes中。
在pom.xml中加入这段代码

```xml
    <build>
            <resources>
                <resource>
                         <directory>src/main/java</directory>
                    <includes>
                        　<include>**/*.xml</include>
                    </includes>
                </resource>
                <resource>
                    　　　<directory>src/main/resources</directory>
                </resource>
            </resources>
        </build>

```
**运行结果如下**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223220527342.png)

数据库信息如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223220734868.png)



## 二，配置事务处理

spring boot配置事务主要涉及两个注解

第一个		@EnableTransactionManagement
在入口main方法中配置这个注解，表示开启spring boot事务处理

```java
    @EnableTransactionManagement//开始spring boot事务支持
    @MapperScan("com.springboot.demo.dao")
    @SpringBootApplication//spring boot的核心注解，作用：开启spring的自动配置
    public class Application {
    
        public static void main(String[] args) {
    
            //启动了spring Boot程序，启动spring容器，内嵌的tomcat
            SpringApplication.run(Application.class, args);
        }
    
    }
```

第二个	@Transactional
在service实现类的某个方法上添加这个注解，就表示这个方法需要事务支持


首先，找到你的service实现类，加上@Transactional 注解，如果你加在类上，那该类所有的方法都会被事务管理，如果你加在方法上，那仅仅该方法符合具体的事务。当然我们一般都是加在方法上。因为只有增、删、改才会需要事务

```java

    @Service("UserService")
    public class UserSerivceimpl implements UserService {
    
        @Resource
        private UserDao userdao;
    
        @Override
        public List<User> selectAllUser() {
            return userdao.selectAllUser();
        }
    
        @Transactional//表示这个方法需要事务支持
        @Override
        public void update() {
            User user=new User();
            user.setName("james");
            user.setId(222);
            userdao.update(user);
            int i=10/0;//这个是错误点
        }
    }
```

controller层

```java
    @RequestMapping("/boot/update")
        public String update(){
    
            userService.update();
            return "update";
        }

```

**运行结果：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122322183960.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

虽然报错，但是数据库信息没有更新，信息还是回滚到更新之前的状态

```xml
   		<update id="update" parameterType="com.springboot.demo.pojo.User">
    		update user set name=#{name},id=#{id} where id=222
    
    	</update>
 ``` 	

这次把错误点注释掉重新运行


![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223222130352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223222159292.png)


运行成功！