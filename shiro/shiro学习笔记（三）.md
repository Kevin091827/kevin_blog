今日内容：

 - shiro实现rememberMe功能
 - shiro权限的两种方式总结


# 1.shiro实现rememberMe

### 1.什么是rememberMe
  Shiro 对记 住我的 Subject 和通过验证的 Subject 作了精确的区分:
  1. Remembered(记住我)：一个记住我的 Subject 不是匿名的，而且有一个已知的身份 ID（也就是 subject.getPrincipals()是非空的）。但是这个被记住的身份 ID 是在之前的 session 中被认证的。如果 subject.isRemembered()返回 true，则 Subject 被认为是被记住的。 
  2.  Authenticated(已认证)：一个已认证的 Subject 是指在当前 Session 中被成功地验证过了（也就是说，login 方法被调用并且没有抛出异常）。如果 subject.isAuthenticated()返回 true 则认为 Subject 已通过验证。
### 2.记住我和认证我的区别（ isAuthenticated()和isRemembered()）
两者为互斥关系
 Remembered 和 Authenticated 是互斥的——若其中一个为真则另一个为假，反之亦然
 当用户只记得之前与应用的交互时，认证将不复存在：被记住的身份 ID 使系统明白这个用户可能是谁，但在 现实中没有办法绝对保证被记住的 Subject 代表期望的用户。一旦 Subject 通过验证，它们将不再仅仅被认为 是被记住的，由于它们的身份已经在当前 session 中被证实。 
### 3.具体实现
具体实现很简单
#### 1.在shiro配置类配置好，可以通过记住我直接免登陆访问路径，采用"user" 

 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115155558180.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

```java
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
    
            //配置rememberme可以直接访问的路径
            filterMap.put("/all/selectByPages","user");
    
            //配置资源授权过滤器
            filterMap.put("/all/toUpdateUser","authc,perms[user:update]");
            filterMap.put("/all/deleteUser","authc,perms[user:delete]");
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
#### 2.在controller层登录方法， token.setRememberMe(true);
```java
     //登录认证页面
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
                token.setRememberMe(true);
                try {
                    subject.login(token);
                    model.addAttribute("msg", "登录成功");
                    return "redirect:/all/selectByPages";
                } catch (UnknownAccountException e) {
                    model.addAttribute("msg", "用户名错误");
                    return "login2";
                } catch (IncorrectCredentialsException e) {
                    model.addAttribute("msg", "密码错误");
                    return "login2";
                }
            }
            return "redirect:/all/selectByPages";
        }
```
#### 3.测试
##### 第一次登录：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115155716974.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115155758273.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 关掉再开一个，直接访问selectByPages（之前没登录会直接拦截，跳转到登录界面）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115155953716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

发现可以直接访问，rememberMe生效了

但是，当尝试修改功能时，又会直接跳转到登录界面

### 4.应用与感想
下面是一个相当普遍的情况，有助于说明 Remembered 和 Authenticated 之间区别的重要性。 比方说，你正在访问 Amazon.com。你已经登录成功并添加了几本书到你的购物车。但你心烦意乱地跑出去开 会，却忘了注销。会议结束后，已经到了回家的时候，于是你离开了办公室。   第二天你工作的时候，你意识到你没有完成购买，于是你返回到 amazon.com。这一次，Amazon“记得”你是 谁，给出了你的欢迎页面，并仍然为你提供一些个性化的建议书籍。对 Amazon 而言，subject.isRemembered() 将返回 true。   但是，当你尝试访问你的帐户来更新你的信用卡信息为你书付账时会发生什么呢？尽管 Amazon“记住”你 (isRemembered() = = true)，它不能保证你就是实际上的你（例如，也许一个同事正在使用你的计算机）。   
所以，在你能够执行像更新信用卡信息等敏感行为之前，Amazon 将强制让你登录，使它们能够保证你的身份。 在登录后，你的身份已经被核实，同时对 Amazon 而言，isAuthenticated()现在返回是 true。   这种情况在许多类型的应用中发生的是如此的频繁，所以这些功能被内置在 Shiro 中，这样你就能利用它来为 你的应用服务了。现在，无论你使用的是 isRemembered()还是 isAuthenticated()来定制你的视图和工作流都由 你来决定，但 Shiro 将维持这一基本情况以防你需要它。   

# 二，shiro权限实现的两种方式：
## 1.基于授权字符串
##### 1.shiro配置类配置授权字符串，必须和数据库中的授权字符串一样
```java
   		    //配置资源授权过滤器---1.授权字符串
            //filterMap.put("/all/toUpdateUser","authc,perms[user:user]");
            //filterMap.put("/all/deleteUser","authc,perms[user:boss]");
            //filterMap.put("/all/insertUser","authc,perms[user:boss]");
```
##### 2.自定义realm
```java
            Subject subject = SecurityUtils.getSubject();
            String username = (String) subject.getPrincipal();
            /*
            @第一种方式：权限字符串
            UserInfo userInfo = userService.getPerm(username);
            if (userInfo!=null){
               simpleAuthorizationInfo.addStringPermission(userInfo.getPerm());
               return simpleAuthorizationInfo;
            }else {
                return null;
            }
            */

```
##### 3.前端html页面使用shiro标签控制
```xml
     <!--thymeleaf整合shiro权限标签，实现权限管理--><!--shiro:hasPermission="user:boss"-->            
```
## 2.基于角色授权
步骤和上边一样

##### 1.自定义realm
```java
            Subject subject = SecurityUtils.getSubject();
            String username = (String) subject.getPrincipal();
            //@第二种方式：角色授权
            UserInfo userInfo = userService.getRole(username);
            if(userInfo!=null){
                simpleAuthorizationInfo.addRole(userInfo.getRole());
                return simpleAuthorizationInfo;
            }else{
                return null;
            }
```
##### 2.shiro配置类
```java
	
		//配置资源授权过滤器---2.角色授权
        filterMap.put("/all/toUpdateUser","roles[boss]");
        filterMap.put("/all/deleteUser","roles[boss]");
        filterMap.put("/all/insertUser","roles[boss]");
```
##### 3.前台html
```html
    <div shiro:hasRole="boss">
```
### 数据库

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115171920749.png)

### 效果展示

第一个用户，没有任何操作权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115171743571.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

第二个用户： 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190115171839652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

这个比较简单，后期会基于这个深入
