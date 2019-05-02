## 1.spring-Boot简介

 1. Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置
 2. Spring Boot是spring家族中的一个全新框架，它用来简化spring应用程序的创建和开发过程，即他能简化我们之前采取的Spring+Spring MVC+Mybatis框架进行开发的过程
 3. 在以前我们采用ssm（Spring+Spring MVC+Mybatis）框架进行开发的时候，往往在搭建和整合三大框架时需要做很多工作，例如配置web.xml，配置spring，mybatis，然后将他们整合的到一起，然而Spring-Boot对此进行了颠覆，他抛弃了繁琐的xml配置，采用大量的默认配置简化了我们的开发过程
 4. Spring Boot让编码变得简单，让配置变得简单，让部署变得简单，让监控变得简单
 
## 2.四大特性

**1.快速**

 -    能够快速创建基于Spring的应用程序

**2.简单**

 - 能够直接使用java main方法启动内嵌的Tomcat, Jetty 服务器运行Spring boot程序，不需要部署war包文件
 - 提供约定的starter POM来简化Maven配置，让Maven的配置变得简单
 - 根据项目的Maven依赖配置，Spring boot自动配置Spring、Spring mvc等
 - 基本可以完全不使用XML配置文件，采用注解配置

**3.监控**

 - 提供了程序的健康检查等功能


## 3.四大核心

 1. 自动配置------简化xml配置
 2. 起步依赖------简化maven配置
 3. Actuator-------深入了解spring boot程序内部信息
 4. 命令行界面---这是Spring Boot的可选特性，主要针对 Groovy 语言使用
 
 


## 4.开发环境

 - 如果是使用 eclipse，推荐安装 Spring Tool Suite (STS) 插件
 - 如果使用 IDEA 旗舰版，自带了Springboot插件


## 5.第一个Spring Boot项目

1.项目创建
在这里，笔者采用的是idea的spring Boot自带插件创建项目

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222132417652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222132511975.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

根据项目需要修改![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222132547348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

项目创建成功，相关的依赖正在下载

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122213264370.png)

.mvn下是他插件自己创建的，里边是maven的jar包和他的配置文件，如果本地没有maven可以使用idea自带的maven插件编译和打包我们的SNAPSHOT，这个文件夹也可以直接删除

main下的Application是程序的入口main方法
resource下static存放静态资源，template是模板页面，application.properties是配置文件，以后会用到，现在是空的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222133256811.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这个是pom.xml，spring boot自动帮我们配置，我们可以直接添加相关依赖

```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <modelVersion>4.0.0</modelVersion>
    
        <!--继承spring Boot的父级项目的依赖-->
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.0.7.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <groupId>com.springBoot.demo</groupId>
        <artifactId>01-springboot-web</artifactId>
        <version>1.0.0</version>
        <name>01-springboot-web</name>
        <description>Demo project for Spring Boot</description>
    
        <!--属性配置-->
        <properties>
            <project.build.sourceEncording>UTF-8</project.build.sourceEncording>
            <project.reporting.outputEncording>UTF-8</project.reporting.outputEncording>
            <java.version>1.8</java.version>
        </properties>
    
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
        </dependencies>
    
        <build>
            <plugins>
    
                <!--spring boot提供的项目编译打包插件-->
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                </plugin>
            </plugins>
        </build>
    
    </project>
```

这是spring boot入口main方法

```java
    package com.springboot.demo;
    
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    
    //spring Boot的入口main方法
    @SpringBootApplication
    public class Application {
    
        public static void main(String[] args) {
    
            //启动了spring Boot程序，启动spring容器，内嵌的tomcat
            SpringApplication.run(Application.class, args);
        }
    
    }

```
现在我们来写一个controller

   
```java
        package com.springboot.demo.controller;
        import org.springframework.stereotype.Controller;
        import org.springframework.web.bind.annotation.RequestMapping;
        import org.springframework.web.bind.annotation.ResponseBody;
        
        @Controller
        public class hellocontroller {
        
            @RequestMapping("/boot/hello")
            public @ResponseBody String hello(){
                return "hello spring boot";
            }
     }
```
在main方法那里右键运行一下

这是spring boot的标志，括号里边是版本号

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222134816554.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

启动内嵌的Tomcat

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222134847411.png)

这时就可以在浏览器上,访问我们的项目啦

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181222135050441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
