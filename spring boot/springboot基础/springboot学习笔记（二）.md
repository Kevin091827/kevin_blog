## 今天主要内容：

 - spring boot 配置文件
 - spring boot下的spring mvc
 - spring boot下集成jsp


## 一，spring boot下的配置文件

**1.spring boot的核心配置文件**

  核心配置文件当然是application.properties啦，但是，application可以是两种格式的文件

  可以是application.properties，也可以是application.yml，他们两种主要是在配置方式上有些许区别。

  分别是：

  application.properties是使用键值对的形式配置的

 ```application
    #spring Boot的核心配置文件，可以使用.yml也可以是.properties,两种文件同时存在，.properties优先使用
    #配置内嵌tomacat的端口号
    server.port=8080
    #配置上下文，根据上下文访问,原来的是server.context-path，但是这个已经被弃用了，现在是server.servlet.context-path=/project-name
    server.servlet.context-path=/01-springboot-web
```
application.yml

是一种以yaml格式的配置文件，主要采用空格，换行的格式排版进行配置（值和前面的冒号：配置项留有空格），他是一种能被计算机识别的数据序列化格式，类似xml，但是比xml简单，易读。
```yml
    server:
      port: 8080
      context-path: /****
```

如果.yml和.properties同时出现的话，.properties优先于.yml，在实际开发中可能有多个环境，比如：application-test.properties是测试时使用的，

application-product.properties是生产时使用的，这时候该怎么办呢？只需要在application.properties中配置

```yml
    #激活配置文件，多环境配置时使用，有激活的优先使用激活的。
    spring.profiles.active=test
    #这样就是使用测试环境的application-test.properties
```


**2.自定义配置文件**

我们可以在Spring boot的核心配置文件中自定义配置，然后采用如下注解去读取配置的属性值。

```application
    #自定义配置文件，要手动读取自定义配置文件
    boot.name=kevin
    boot.location=广东工业大学
```

读取自定义配置文件的方式有两种：

1.@value（"${属性名}"）逐个读取自定义配置文件

```java
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.ResponseBody;
    
    @Controller
    public class hellocontroller {
    
        @RequestMapping("/boot/hello")
        public @ResponseBody String hello(){
            return "hello spring boot";
        }
    
        @Value("${boot.name}")
        private String name;
    
        @Value("${boot.location}")
        private String location;
    
        @Autowired
        private Config config;
    
        @ResponseBody
        @RequestMapping("/boot/configInfo")
        public String configInfo(){
            String info1=name+"  "+location;
            String info2=config.getName()+"    "+config.getLocation();
            return info2;
        }
    
    }
```

2.@ConfigurationProperties(prefix = "属性名前缀")将整个文件映射成对象，然后通过对象在controller调用

```java
    package com.springboot.demo.config;
    
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.stereotype.Component;
    
    @Component
    @ConfigurationProperties(prefix = "boot")
    public class Config {
    
        private String name;
        private String location;
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public String getLocation() {
            return location;
        }
    
        public void setLocation(String location) {
            this.location = location;
        }
    }

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223011206940.png)


## 二，spring boot下的spring mvc

spring boot下的spring mvc和之前的spring mvc是完全一样，但是spring boot 帮我们简化了一些操作

@Controller-------即为Spring mvc的注解，处理http请求；

@RestController
 - Spring4后新增注解；
 - 是@Controller与@ResponseBody的组合注解；
 - 用于返回字符串或json数据；
	
	@GetMapping----RequestMapping 和 Get请求方法的组合；
	
	@PostMapping-----RequestMapping 和 Post请求方法的组合；
	
	@PutMapping------RequestMapping 和 Put请求方法的组合；
	
	@DeleteMapping---RequestMapping 和 Delete请求方法的组合；

下边我会先介绍完spring boot下集成jsp后在综合这点，做一个小例子

## 三，spring boot下集成jsp

在Spring boot中使用jsp，按如下步骤进行：

**1、在pom.xml文件中配置依赖项**
```xml
    <!--引入Spring Boot内嵌的Tomcat对JSP的解析包-->
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
    </dependency>
    <!-- servlet依赖的jar包start -->
    <dependency>
    	<groupId>javax.servlet</groupId>
    	<artifactId>javax.servlet-api</artifactId>
    </dependency>
    <!-- servlet依赖的jar包start -->
    
    <!-- jsp依赖jar包start -->
    <dependency>
    	<groupId>javax.servlet.jsp</groupId>
    	<artifactId>javax.servlet.jsp-api</artifactId>
    	<version>2.3.1</version>
    </dependency>
    <!-- jsp依赖jar包end -->
    
    <!--jstl标签依赖的jar包start -->
    <dependency>
    	<groupId>javax.servlet</groupId>
    	<artifactId>jstl</artifactId>
    </dependency>
    <!--jstl标签依赖的jar包end -->
```
**2、在application.properties文件配置spring mvc的视图展示为jsp：**
```application
    spring.mvc.view.prefix=/
    spring.mvc.view.suffix=.jsp
```
3、在src/main 下创建一个webapp目录，然后在该目录下新建jsp页面

maven中的build中要配置的信息，以便访问到web资源
```xml
         <resources>
    			<resource>
    				<directory>src/main/java</directory>
    				<includes>
    					<include>**/*.xml</include>
    				</includes>
    			</resource>
    			<resource>
    				<directory>src/main/resources</directory>
    				<includes>
    					<include>**/*.*</include>
    				</includes>
    			</resource>
    			<resource>
    				<directory>src/main/webapp</directory>
    				<targetPath>META-INF/resources</targetPath>
    				<includes>
    					<include>**/*.*</include>
    				</includes>
    			</resource>
    		</resources>
  ```
  
这样编译后才会这样

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223012215745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


下边复习下刚刚整理的知识点，做一个简单的登录

首先这是jsp的页面

登录页面---index.jsp

```html
    <%@ page language="java" contentType="text/html; charset=UTF-8"
             pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
    </head>
    <body>
    
    <form action="${pageContext.request.contextPath }/boot/dologin" method="post">
        用户名：<input type="text" name="userName"/><br/>
        密码：<input type="password" name="pwd"/><br/>
        <input type="submit" value="登录"/><br/>
    </form>
    
    </body>
    </html>
```
登录成功跳转页面---success.jsp
```html
    <%@ page language="java" contentType="text/html; charset=UTF-8"
             pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
    </head>
    <body>
    
    <%
        String msg = (String)request.getAttribute("msg");
        if(msg!=null)
            out.print(msg);
    %>
    
    </body>
    </html>
```
个人比较懒，就简化了些流程，直接写写controller层代码
  ```java
    package com.springboot.demo.controller;
    
    import org.springframework.stereotype.Controller;
    import org.springframework.ui.Model;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PostMapping;
    import org.springframework.web.bind.annotation.RequestParam;
    
    @Controller
    public class JspController {
    
    
        @PostMapping("/boot/dologin")
        public String dologin(Model model, @RequestParam("userName") String userName,@RequestParam("pwd") String pwd){
    
            if(userName.equals("123")&&pwd.equals("123")){
                model.addAttribute("msg","success to login");
                return "success";
            }
            else{
                return null;
            }
        }
    
    }
  ```
  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223012719304.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223012708342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181223012745551.png)