今日内容

 - 认识Thymeleaf
 - Spring boot 集成 Thymeleaf
 - Thymeleaf 的标准表达式
 - Thymeleaf 的常见属性
 - Thymeleaf 字面量和字符串拼接
 - Thymeleaf 三元运算判断和关系判断
 - Thymaleaf 表达式基本对象
 - Thymaleaf 表达式功能对象

# 1.认识Thymeleaf

 - Thymeleaf是一个流行的模板引擎，该模板引擎采用Java语言开发
 - 模板引擎是一个技术名词，是跨领域跨平台的概念，在Java语言体系下有模板引擎，在C#、PHP语言体系下也有模板引擎，甚至在JavaScript中也会用到模板引擎技术
 - Java生态下的模板引擎有 Thymeleaf 、Freemaker、Velocity、Beetl（国产） 等
 - Thymeleaf模板既能用于web环境下，也能用于非web环境下，在非web环境下，它能直接显示模板上的静态数据，在web环境下，它能像JSP一样从后台接收数据并替换掉模板上的静态数据
 - Thymeleaf 它是基于HTML的，以HTML标签为载体，Thymeleaf 要寄托在HTML的标签下实现对数据的展示
 - Thymeleaf的官方网站：http://www.thymeleaf.org
 - Spring boot 集成了thymeleaf模板技术，并且spring boot官方也推荐使用thymeleaf来替代JSP技术
 - thymeleaf是另外的一种模板技术，它本身并不属于springboot，springboot只是很好地集成这种模板技术，作为前端页面的数据展示
 
# 2.Spring boot 集成 Thymeleaf

1.在maven中加入如下配置
```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
```
  2.在Spring boot的核心配置文件application.properties中对Thymeleaf进行配置：
  
````shell
 关闭thymeleaf的模板缓存，建议开发的时候关闭
    spring.thymeleaf.cache=false
    #使用遗留的html5去除严格的验证（在使用springboot的过程中，如果使用thymeleaf作为模板文件，则要求HTML格式必须为严格的html5格式，必须有结束标签，否则会报错；）
    spring.thymeleaf.mode=LEGACYHTML5
````


注意：如果是spring boot 2.0一下的如果不想对标签进行严格的验证，使用spring.thymeleaf.mode=LEGACYHTML5去掉验证，去掉该验证，则需要引入如下依赖，否则会报错：
```xml
    <dependency>
        <groupId>org.unbescape</groupId>
        <artifactId>unbescape</artifactId>
        <version>1.1.5.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.unbescape</groupId>
        <artifactId>unbescape</artifactId>
        <version>1.1.5.RELEASE</version>
    </dependency>
```
NekoHTML是一个Java语言的 HTML扫描器和标签补全器 ,这个解析器能够扫描HTML文件并“修正”HTML文档中的常见错误。

NekoHTML能增补缺失的父元素、自动用结束标签关闭相应的元素，修复不匹配的内嵌元素标签等；

3.写一个Controller去映射到模板页面（和SpringMVC基本一致）

```java
    @RequestMapping("/boot/demo1")
        public String demo1(Model model){
            model.addAttribute("msg","hello thymeleaf");
            return "index";
        }
```
4.在src/main/resources的templates下新建一个index.html页面用于展示数据

```html
    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
    <head>
    <meta charset="UTF-8"/>
    <title>Spring boot集成 Thymeleaf</title>
    </head>
    <body>
    <p th:text="${msg}">Spring boot集成 Thymeleaf</p>
    </body>
    </html>
```
5.运行测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230212436717.png)

Springboot使用thymeleaf作为视图展示，约定将模板文件放置在src/main/resource/templates目录下，静态资源放置在src/main/resource/static目录下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230212528480.png)

# 3.Thymeleaf 的标准表达式

Thymeleaf 的标准表达式主要有三类：

 * 1. 标准变量表达式
 * 2. 选择变量表达式
 * 3. URL表达式

## 1.标准变量表达式
controller层
```java
     @RequestMapping(value="/userinfo")
        public String userinfo (Model model) {
            User user = new User();
            user.setNick("kevin");
            user.setPhone("13700020000");
            user.setEmail("kevin@xxx.com");
            user.setAddress("北京");
            model.addAttribute("user", user);
            return "index";
        }
```
index.html
```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    
    <!--原先使用jsp的${}从后台获取数据-->
    <!--现在使用模板获取-->
    <p th:text="${msg}">
        静态显示的内容
    </p>
    <!--
    1.${} 是 thymeleaf的标准变量表达式
       变量表达式用于访问容器（tomcat）上下文环境中的变量，功能和 JSTL 中的 ${} 相同
       Thymeleaf 中的变量表达式使用 ${变量名} 的方式获取其中的数据
        eg：
        后台返回一个model.addAttribute("user",user)//user是一个对象
        <span th:text="${user.nick}">x</span>
        <span th:text="${user.phone}">137xxxxxxxx</span>
        <span th:text="${user.email}">xxx@xx.com</span>
        <span th:text="${user.address}">北京.xxx</span>
    
        如果我只是把这个html当做静态页面打开，不启动tomacat，那么显示的是x
                                                                    137xxxxxxxx
                                                                    xxx@xx.com
                                                                    北京.xxx
        但是如果我启动tomacat，他就显示我后台传送到前台的数据，替代了静态的内容
        其中，th:text="" 是Thymeleaf的一个属性，用于文本的显示；
    -->
    <p>
        <span th:text="${user.nick}">xxx</span>
        <span th:text="${user.phone}">*******</span>
        <span th:text="${user.email}">123@xx.com</span>
        <span th:text="${user.address}">xxxx</span>
    </p>
    </head>
    <body>
```
运行：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230213749914.png)

## 2.选择变量表达式
```html
    index.html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
    </head>
    <body>
    
    <!--原先使用jsp的${}从后台获取数据-->
    <!--现在使用模板获取-->
    <p th:text="${msg}">
        静态显示的内容
    </p> 
    <!--2.选择变量表达式 *{}
          选择变量表达式，也叫星号变量表达式
          选择表达式首先使用th:object来邦定后台传来的User对象，然后使用 * 来代表这个对象，后面 {} 中的值是此对象中的属性
          选择变量表达式 *{...} 是另一种类似于变量表达式 ${...} 表示变量的方法
          选择变量表达式在执行时是在选择的对象上求解，而${...}是在上下文的变量Model上求解；
          通过 th:object 属性指明选择变量表达式的求解对象
    -->
    
    <div th:object="${user}">
        <p>
            nick:<span th:text="*{nick}">xxx</span>
        </p>
        <p>
            phone:<span th:text="*{phone}">123456</span>
        </p>
        <p>
            email:<span th:text="*{email}">11111@111.com</span>
        </p>
        <p>
            address:<span th:text="*{address}">xxxx</span>
        </p>
    </div>
    <!--标准变量表达式和选择变量表达式可以混合一起使用，比如-->
    <div th:object="${user}">
        <p>
            nick:<span th:text="*{nick}">xxx</span>
        </p>
        <p>
            phone:<span th:text="${user.phone}">123456</span>
        </p>
        <p>
            email:<span th:text="${user.email}">11111@111.com</span>
        </p>
        <p>
            address:<span th:text="*{address}">xxxx</span>
        </p>
    </div>
    
    <!--也可以不使用 th:object 进行对象的选择，而直接使用 *{...} 获取数据，比如-->
    <div>
        <p>
            nick:<span th:text="*{user.nick}">xxx</span>
        </p>
        <p>
            phone:<span th:text="*{user.phone}">123456</span>
        </p>
        <p>
            email:<span th:text="*{user.email}">11111@111.com</span>
        </p>
        <p>
            address:<span th:text="*{user.address}">xxxx</span>
        </p>
    </div>
     </head>
     <body>

```
一样的controller

运行：

![在这里插入图片描述](https://img-blog.csdnimg.cn/201812302140480.png)

## 3.URL表达式

```html
    <!--3.url表达式-->
    <!--绝对url-->
    <a href="info.html" th:href="@{'http://localhost:8080/boot/user/info?phone='+${user.phone}}">查看</a>
    <a href="info.html" th:href="@{|http://localhost:8080/boot/user/info?phone=${user.phone}&email=${user.email}|}">查看</a>
    
    <!--相对url，第一种和第二种的区别是有没有加/，有 / 就自动把项目上下文加进去url中-->
    <a href="info.html" th:href="@{'user/info?phone='+${user.phone}}">查看</a>
    <a href="info.html" th:href="@{'/user/info?phone='+${user.phone}}">查看</a>
    </body>
   
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230214303396.png)

# 3.Thymeleaf 的常见属性

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230214513134.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

需要用到的时候去查文档就可以了，这里举例一两个

   ```html    
   list循环遍历

    <p th:each="u : ${user}">
        <!--
            interStat是循环体的信息，通过该变量可以获取如下信息：
                index: 当前迭代对象的index（从0开始计算）
                count: 当前迭代对象的个数（从1开始计算）
                size: 被迭代对象的大小
                current: 当前迭代变量
                even/odd: 布尔值，当前循环是否是偶数/奇数（从0开始计算）
                first: 布尔值，当前循环是否是第一个
                last: 布尔值，当前循环是否是最后一个
        -->
        <span th:text="${uStat.count}"></span>
        <span th:text="${u.nick}">xxx</span>
        <span th:text="${u.phone}">123</span>
        <span th:text="${u.email}">xxx.com</span>
        <span th:text="${u.address}">xxx</span>
    </p>
    
    map循环遍历
    <p th:each="m : ${map}">
        <span th:text="${m.key}"></span>
        <span th:text="${m.value.nick}"></span>
        <span th:text="${m.value.phone}"></span>
        <span th:text="${m.value.email}"></span>
        <span th:text="${m.value.address}"></span>
    </p>
    
    if标签
    <p>
        <span th:if="${sex} eq 1">
            男
        </span>
        <span th:if="${sex} eq 2">
            女
        </span>
    
        第二种写法
        <span th:if="${sex==1}">
            男
        </span>
    -->
    </p>
    
    </br>
    unless
    <p>
        <span th:unless="${sex} == 2">
            男
        </span>
        <span th:unless="${sex} == 1">
            女
        </span>
    </p>
    </br>
    switch
    <div th:switch="${sex}">
            <p th:case="1">男</p>
            <p th:case="2">女</p>
    </div>
```
controller:
```java
    @RequestMapping("/boot/demo2")
        public String demo2(Model model, HttpServletRequest request){
    
            request.setAttribute("name","kevin");
    
            List<User> list = new ArrayList<>();
            Map map = new HashMap();
            for(int i=0;i<10;i++) {
                User user = new User();
                user.setNick("kevin"+i);
                user.setPhone("123456"+i);
                user.setEmail(i+"123456789@123.com");
                user.setAddress("北京"+i);
                list.add(user);
                map.put(String.valueOf(i),user);
            }
            model.addAttribute("user",list);
            model.addAttribute("map", map);
            model.addAttribute("sex",1);
            return "index";
        }
```
运行：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230214953140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230215022293.png)

# 4.Thymeleaf 字面量和字符串拼接

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230215145394.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 5.Thymeleaf 三元运算判断和关系判断

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018123021530890.png)


# 6.Thymaleaf 表达式基本对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230215357842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 7.Thymaleaf 表达式功能对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181230215446509.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

```html
    <span th:text="${#request.getAttribute('name')}">
    name
    </span>
```
Thymeleaf 在 SpringBoot 中的配置大全

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018123021561790.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
