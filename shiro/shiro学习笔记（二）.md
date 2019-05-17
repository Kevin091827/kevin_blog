今日内容：

 - shiro资源授权
 - thymeleaf整合shiro权限标签

# 1.shiro资源授权
## 1.授权：
授权，又称作为访问控制，是对资源的访问管理的过程。换句话说，控制谁有权限在应用程序中做什么

## 2.授权方式
#### 在 Shiro 中执行授权可以有 3 种方式：
 1. 编写代码——你可以在你的 Java 代码中用像 if 和 else 块的结构执行授权检查。
 2. JDK 的注解——你可以添加授权注解给你的 Java 方法。 
 3. JSP/GSP 标签库——你可以控制基于角色和权限的 JSP 或者 GSP 页面输出。 
 
## 3.实现
### 1.编写代码（硬编码）
1.无论哪种形式都需要在shiro的配置类配置资源过滤器，配置资源授权路径和未授权页面路径
```java
     	@Bean
        public ShiroFilterFactoryBean getShiroFilterFactoryBean( @Qualifier("securityManager") DefaultWebSecurityManager securityManager){
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            shiroFilterFactoryBean.setSecurityManager(securityManager);
            //拦截器，拦截路径有顺序，所以采用LinkedHashMap
            Map<String,String> filterMap = new LinkedHashMap<>();
            //配置拦截路径，注意顺序，先配置无需权限的路径,过滤链定义，从上向下顺序执行，一般将/**放在最为下边
            filterMap.put("/all/login2","anon");
            filterMap.put("/all/toLogin2","anon");
            //配置资源授权过滤器
            filterMap.put("/all/toUpdateUser","perms[user:update]");
            filterMap.put("/all/deleteUser","perms[user:delete]");
            filterMap.put("/all/insertUser","perms[user:add]");
    
            filterMap.put("/all/**","authc");
            //设置登录url，如果不设置的，shiro会自动跳转到login.jsp
            shiroFilterFactoryBean.setLoginUrl("/all/toLogin2");
            //设置未授权url
            shiroFilterFactoryBean.setUnauthorizedUrl("/all/toUnanthorizedUrl");
    
            shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
            System.out.print("shiro拦截器已经加载");
            return shiroFilterFactoryBean;
        }
```
2.硬编码实现资源授权

回到自定义的realm，重写doGetAuthorizationInfo方法
```java
       /**
         * 授权逻辑
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            System.out.print("执行授权逻辑");
            //给资源进行授权
            SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
            //添加授权字符串
            //1.硬编码形式实现资源授权
            simpleAuthorizationInfo.addStringPermission("user:update");
            return simpleAuthorizationInfo;
        }
```
#### 2.连接数据库，从数据库中获取授权字符串，实现资源授权
1. 数据库表添加授权字符串字段

    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114164508147.png)

2. 编写好相应的dao层接口和mapper.xml

3. 重写doGetAuthorizationInfo方法
```java
       /**
         * 授权逻辑
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
    
            System.out.print("执行授权逻辑");
    
            //给资源进行授权
            SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
    
            //添加授权字符串
            //1.硬编码形式实现资源授权
            //simpleAuthorizationInfo.addStringPermission("user:update");
            //2.从数据库中获取授权字符串
            //获取当前已经认证的用户的用户名
            Subject subject = SecurityUtils.getSubject();
            String username = (String) subject.getPrincipal();
            UserInfo userInfo = userService.getPerm(username);
            if (userInfo!=null){
               simpleAuthorizationInfo.addStringPermission(userInfo.getPerm());
               return simpleAuthorizationInfo;
            }else {
                return null;
            }
        }
```
资源授权在这里就完成

#  二，thymeleaf整合shiro权限标签
目的：实现不同权限用户看到的页面也不同

#### 1.导入jar包，完善pom.xml
```xml
    <!--thymeleaf整合shiro权限的标签-->
            <dependency>
                <groupId>com.github.theborakompanioni</groupId>
                <artifactId>thymeleaf-extras-shiro</artifactId>
                <version>2.0.0</version>
            </dependency>
```
#### 2.在shiro配置类配置shiroDialect
```java
    //配置shiroDialect,用于thymeleaf和shiro标签使用
        @Bean
        public ShiroDialect getShiroDialect(){
            return new ShiroDialect();
        }
```
#### 3.在登录成功html页面引入权限标签
```html
    <!DOCTYPE html>
    <html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <!--通过url表达式引入-->
        <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
        <script th:src="@{/js/jquery.min.js}"></script>
        <script th:src="@{/js/popper.min.js}"></script>
        <script th:src="@{/js/bootstrap.min.js}"></script>
    </head>
    <body style="margin-left: 70px;margin-right: 70px ">
    <div class="container">
        <h2>AllUser</h2>
        <table class="table">
            <caption>
                <!--thymeleaf整合shiro权限标签，实现权限管理-->
                <div shiro:hasPermission="user:add">
                    <a th:href="@{|/all/insertUser|}">添加</a>
                </div>
                <div>
                    <a th:href="@{|/all/toLoginOut|}">退出登录</a>
                </div>
            </caption>
            <thead>
            <tr>
                <th>num</th>
                <th>id</th>
                <th>nick</th>
                <th>phone</th>
                <th>Email</th>
                <th>address</th>
                <td>操作</td>
            </tr>
            </thead>
            <tbody>
                <tr th:each="u : ${userlist}">
                    <td th:text="${uStat.count}"></td>
                    <td th:text="${u.id}"></td>
                    <td th:text="${u.nick}">xxx</td>
                    <td th:text="${u.phone}">123</td>
                    <td th:text="${u.email}">xxx.com</td>
                    <td th:text="${u.address}">xxx</td>
                    <td>
                        <div shiro:hasPermission="user:update">
                            <a th:href="@{|/all/toUpdateUser?id=${u.id}|}">修改</a>
                        </div>
                        <div shiro:hasPermission="user:delete">
                            <a href="">删除</a>
                        </div>
                    </td>
                </tr>
    
                <tr style="text-align: center">
                   <td colspan="5">
    
                       <span th:if="${curpage <= 1}">
                            首页
                            上一页
                       </span>
    
                       <span th:if="${curpage > 1}">
                            <a th:href="@{/all/selectByPages}">首页</a>
                            <a th:href="@{|/all/selectByPages?selectPage=${curpage-1}|}">上一页</a>
                       </span>
    
                       <span th:if="${curpage == totalpages}">
                            下一页
                            尾页
                       </span>
                       <span th:if="${curpage < totalpages}">
                           <a th:href="@{|/all/selectByPages?selectPage=${curpage+1}|}">下一页</a>
                           <a th:href="@{|/all/selectByPages?selectPage=${totalpages}|}">尾页</a>
                       </span>
    
                   </td>
                </tr>
            </tbody>
        </table>
    </div>
    </body>
    </html>
```
#### 4.测试

先测试用户一：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114165447770.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

用户二

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114165534325.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)