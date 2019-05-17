今日内容：
 - shiro简介
 - spring boot 整合 shiro 实现登录验证功能 
 
# 1.shiro简介
1. Apache Shiro 是一个强大而灵活的开源安全框架，它干净利落地处理身份认证，授权，企业会话管理和加密。 

2. 以下是你可以用 Apache Shiro 所做的事情： 
   1.验证用户来核实他们的身份 
   2.对用户执行访问控制，如：判断用户是否被分配了一个确定的安全角色 ，判断用户是否被允许做某事 
   3. 在任何环境下使用 Session API，即使没有 Web 或 EJB 容器。 
   4.  在身份验证，访问控制期间或在会话的生命周期，对事件作出反应。  聚集一个或多个用户安全数据的数据源，并作为一个单一的复合用户“视图”。 
   5.  启用单点登录（SSO）功能。 
   6.  为没有关联到登录的用户启用"Remember Me"服务


3.特性：
 1. Web Support：Shiro 的 web 支持的 API 能够轻松地帮助保护 Web 应用程序。 
 2. Caching：缓存是 Apache Shiro 中的第一层公民，来确保安全操作快速而又高效。
 3. Concurrency：Apache Shiro 利用它的并发特性来支持多线程应用程序。
 4. Testing：测试支持的存在来帮助你编写单元测试和集成测试，并确保你的能够如预期的一样安全。
 5. "Run As"：一个允许用户假设为另一个用户身份（如果允许）的功能，有时候在管理脚本很有用。
 6.  "Remember Me"：在会话中记住用户的身份，所以他们只需要在强制时候登录。
 7.  Concurrency：Apache Shiro 利用它的并发特性来支持多线程应用程序。
 8.  Testing：测试支持的存在来帮助你编写单元测试和集成测试，并确保你的能够如预期的一样安全。
 
Shiro 的架构有 3 个主要的概念：Subject，SecurityManager 和 Realms

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190113223343592.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 二，登录验证
## 1.在spring boot环境下搭建好shiro环境
### 1.maven项目 pom.xml导入shiro环境
  ```xml
    		 <dependency>
                <groupId>org.apache.shiro</groupId>
                <artifactId>shiro-spring</artifactId>
                <version>1.4.0</version>
            </dependency>
```
### 2.配置shiro配置类
```java


    //shiro配置类
    @Configuration
    public class ShiroConfig {
    
        /**
         * @密码加密处理
         * 配置自定义密码加密器
         * 凭证匹配器 （由于我们的密码校验交给Shiro的SimpleAuthenticationInfo进行处理了 )
         * @return md5加密
         */
        @Bean(name = "CredentialsMatcher")
        public CredentialsMatcher credentialsMatcher() {
            return new CredentialsMatcher();
        }
    
        //创建realm实体bean,交给spring容器
        @Bean(name="loginRealm")
        public LoginRealm getLoginRealm(@Qualifier("CredentialsMatcher") CredentialsMatcher credentialsMatcher){
            LoginRealm loginRealm = new LoginRealm();
            loginRealm.setCredentialsMatcher(credentialsMatcher);
            System.out.print("realm已经加载");
            return loginRealm;
        }
    
        //创建securityManager 关联realm
        @Bean(name="securityManager")
        public DefaultWebSecurityManager getSecurityManager(@Qualifier("CredentialsMatcher") CredentialsMatcher credentialsMatcher){
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            securityManager.setRealm(getLoginRealm(credentialsMatcher));
            System.out.print("安全管理器已经加载");
            return securityManager;
        }
    
        //创建shiroFilter关联securityManager
        @Bean
        public ShiroFilterFactoryBean getShiroFilterFactoryBean( @Qualifier("securityManager") DefaultWebSecurityManager securityManager){
            ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
            shiroFilterFactoryBean.setSecurityManager(securityManager);
            //拦截器，拦截路径有顺序，所以采用LinkedHashMap
            Map<String,String> filterMap = new LinkedHashMap<>();
            //配置拦截路径，注意顺序，先配置无需权限的路径,过滤链定义，从上向下顺序执行，一般将/**放在最为下边
            filterMap.put("/all/login2","anon");
            filterMap.put("/all/toLogin2","anon");
            filterMap.put("/all/**","authc");
            //设置登录url，如果不设置的，shiro会自动跳转到login.jsp
            shiroFilterFactoryBean.setLoginUrl("/all/toLogin2");
            shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
            System.out.print("shiro拦截器已经加载");
            return shiroFilterFactoryBean;
        }
        
    }
```
### 3.自定义密码加密器
我们知道线上系统的数据库中存储的密码不应该是明文，而是密码加密后的字符串，并且要求加密算法是不可逆的。著名的加密算法有MD5、SHA1等。其中MD5是目前比较可靠的不可逆的加密方式。在shiro中，密码加密是由CredentialsMatcher接口实现的，里边提供很多密码加密的实现类

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114003634744.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

举个例子，其实现类SimpleCredentialsMatcher的子类HashedCredentialsMatcher ，就是采用经典md5加密。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019011400385811.png)

在这里，我自定义了一个md5的密码加密器，并替换掉原来的加密器

```java
    //继承SimpleCredentialsMatcher 自定义密码加密器
    public class CredentialsMatcher extends SimpleCredentialsMatcher {
    
        @Override
        public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
    
            UsernamePasswordToken utoken=(UsernamePasswordToken) token;
            //获得用户输入的用户名
            String token_username = new String(utoken.getUsername());
            //获得用户输入的密码:
            String token_pwd = new String(utoken.getPassword());
                System.out.println("表单密码："+token_pwd);
            //将表单密码md5加密
            String result = md5(token_pwd,token_username);
            String inPassword = new String(result);
                System.out.println("加密密码为："+inPassword);
            //获得数据库中的密码
            String dbPassword=(String) info.getCredentials();
                System.out.println("数据库密码："+dbPassword);
            //进行密码的比对
            return this.equals(inPassword, dbPassword);
        }
    
        public String md5(String token_pwd,String username){
            String hashAlgorithmName = "MD5";//加密方式
            Object crdentials = token_pwd;//密码原值
            Object salt = username;//盐值
            int hashIterations = 1024;//加密1024次
            Object result = new SimpleHash(hashAlgorithmName,crdentials,salt,hashIterations);
            return result.toString();
        }
    
        //加密测试
        /*public static void main(String[] args )
        {
            String hashAlgorithmName = "MD5";//加密方式
            Object crdentials = "123456";//密码原值
            Object salt = "kevin";//盐值
            int hashIterations = 1024;//加密1024次
            Object result = new SimpleHash(hashAlgorithmName,crdentials,salt,hashIterations);
            System.out.println(result);
        }*/
    }
```
认证流程：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114010256286.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114010323640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 4.自定义realm
```java
    //继承AuthorizingRealm 自定义realm
    public class LoginRealm extends AuthorizingRealm {
    
        @Resource
        private UserService userService;
    
        private SimpleAuthenticationInfo info=null;
        /**
         * 授权逻辑
         */
        @Override
        protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
            System.out.print("执行授权逻辑");
            return null;
        }
        /**
         * 认证逻辑
         */
        @Override
        protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
            // 将token装换成UsernamePasswordToken
            UsernamePasswordToken upToken = (UsernamePasswordToken) authenticationToken;
            // 获取用户名即可
            String username = upToken.getUsername();
            // 查询数据库，是否查询到用户名和密码的用户
            UserInfo userInfo = userService.loginAndShiro(username);
    
            if(userInfo != null) {
                /*
                * @principal 数据库用户名或者可以是查询数据库封装的对象
                * @credentials 查询出来的密码，也就是正确密码
                * @salt 盐值
                * @realmName 直接调用父类的getName()
                **/
                Object principal =  userInfo.getUsername();
                Object credentials = userInfo.getPassword();
                // 获取盐值，即用户名
                ByteSource salt = ByteSource.Util.bytes(username);
                String realmName = this.getName();
                // 将账户名，密码，盐值，realmName实例化到SimpleAuthenticationInfo中交给Shiro来管理
                info = new SimpleAuthenticationInfo(principal, credentials, salt,realmName);
            }else {
                // 如果没有查询到，抛出一个异常
                throw new AuthenticationException();
            }
    
            return info;
        }
    }
```
就此shiro的环境就搭好了。

## 2.编写相应controller
```java
     @RequestMapping("/all/toLogin2")
        public String toLogin2() {
            return "login2";
        }
    
        @RequestMapping("/all/login2")
        public String login2(@RequestParam("nick")String nick, @RequestParam("pwd") String pwd,Model model) {
            /**
             * @认证流程
             * 1.获取当前用户：Subject = SecurityUtils.getSubject();
             * 2.判断是否已经获得认证： !subject.isAuthenticated()
             * 3.若没有获得认证，则封装token： UsernamePasswordToken token = new UsernamePasswordToken(nick,pwd);
             * 4.执行登录： subject.login(token);
             * 5.自定义realm，从数据库中获取对应的记录，返回给shiro
             * 6.最后由shiro完成密码比对
             */
    
            Subject subject = SecurityUtils.getSubject();
            if(!subject.isAuthenticated()) {
                UsernamePasswordToken token = new UsernamePasswordToken(nick,pwd);
                try {
                    subject.login(token);
                    model.addAttribute("msg", "登录成功");
                    return "redirect:/all/selectByPages";
                } catch (UnknownAccountException e) {
                    model.addAttribute("msg", "用户名错误");
                    return "login2";
                } catch (IncorrectCredentialsException e) {
                    model.addAttribute("msg", "邮箱错误");
                    return "login2";
                }
            }
            return "redirect:/all/selectByPages";
        }
```
## 3.数据库中存放密文密码

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114011218134.png)

## 4.测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114011228552.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

登录成功！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114011301939.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

看看控制台

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114011332483.png)

密码加密后和数据库的密文比较，更安全

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190114011409379.png)

shiroFilter比securityManager慢加载

