今日内容：

 - spring boot整合shiro实现多realm认证
 
 # spring boot整合shiro实现多realm认证
 
在一些情况下我们数据库有多张表，登录的时候需要查不同的表，判断是否有该用户，这时候，一个realm就显得有心无力，所以多realm认证也是shiro中一项很强大的功能。

通过走源码。

```java
    protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
            this.assertRealmsConfigured();
            Collection<Realm> realms = this.getRealms();
            return realms.size() == 1 ? this.doSingleRealmAuthentication((Realm)realms.iterator().next(), authenticationToken) : this.doMultiRealmAuthentication(realms, authenticationToken);
        }
```

我们发现 ModularRealmAuthenticator 类中，这个方法就是判断realm的数量，然后分别执行相应的方法。单个realm执行的相应方法就不说了，看多realm时执行的doMultiRealmAuthentication(realms, authenticationToken)方法
```java
    protected AuthenticationInfo doMultiRealmAuthentication(Collection<Realm> realms, AuthenticationToken token) {
             //认证策略
            AuthenticationStrategy strategy = this.getAuthenticationStrategy();
            AuthenticationInfo aggregate = strategy.beforeAllAttempts(realms, token);
            if (log.isTraceEnabled()) {
                log.trace("Iterating through {} realms for PAM authentication", realms.size());
            }
    
            Iterator var5 = realms.iterator();
    		
    		//取出每一个realm
            while(var5.hasNext()) {
                Realm realm = (Realm)var5.next();
                aggregate = strategy.beforeAttempt(realm, token, aggregate);
                if (realm.supports(token)) {
                    log.trace("Attempting to authenticate token [{}] using realm [{}]", token, realm);
                    AuthenticationInfo info = null;
                    Throwable t = null;
    
                    try {
                    	//执行该方法，他是一个接口方法，直接到下边的实现方法
                        info = realm.getAuthenticationInfo(token);
                    } catch (Throwable var11) {
                        t = var11;
                        if (log.isDebugEnabled()) {
                            String msg = "Realm [" + realm + "] threw an exception during a multi-realm authentication attempt:";
                            log.debug(msg, var11);
                        }
                    }
    
                    aggregate = strategy.afterAttempt(realm, token, info, aggregate, t);
                } else {
                    log.debug("Realm [{}] does not support token {}.  Skipping realm.", realm, token);
                }
            }
    
            aggregate = strategy.afterAllAttempts(token, aggregate);
            return aggregate;
        }

```
接着
```java
    public final AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
            AuthenticationInfo info = this.getCachedAuthenticationInfo(token);
            if (info == null) {
            //重点在这，就会回来自定义的realm
                info = this.doGetAuthenticationInfo(token);
                log.debug("Looked up AuthenticationInfo [{}] from doGetAuthenticationInfo", info);
                if (token != null && info != null) {
                    this.cacheAuthenticationInfoIfPossible(token, info);
                }
            } else {
                log.debug("Using cached authentication info [{}] to perform credentials matching.", info);
            }
    
            if (info != null) {
                this.assertCredentialsMatch(token, info);
            } else {
                log.debug("No AuthenticationInfo found for submitted AuthenticationToken [{}].  Returning null.", token);
            }
    
            return info;
        }
```
所以，多realm认证开始：
## 1.自定义一个ModularRealmAuthenticator类
```java
    public class NewAuthenticator extends ModularRealmAuthenticator {
    
        /**
         * 自定义 ModularRealmAuthenticator
         * @param authenticationToken
         * @return
         * @throws AuthenticationException
         */
        @Override
        protected AuthenticationInfo doAuthenticate(AuthenticationToken authenticationToken) throws AuthenticationException {
            // 判断getRealms()是否返回为空
            assertRealmsConfigured();
            // 强制转换回自定义的Token
            NewToken token = (NewToken) authenticationToken;
            // 所有Realm
            Collection<Realm> realms = getRealms();
            //登录类型
            String loginType = token.getLoginType();
            //判断realm为哪种登录类型
            List<Realm> typeRealms = new ArrayList<>();
            for (Realm realm : realms) {
                if (realm.getName().contains(loginType)) {
                    typeRealms.add(realm);
                }
            }
            // 判断是单Realm还是多Realm
            if (realms.size() == 1)
                return doSingleRealmAuthentication(typeRealms.iterator().next(), token);
            else
                return doMultiRealmAuthentication(typeRealms, token);
        }
    
    }
```
## 2.自定义UsernamePasswordToken类
```java
    public class NewToken extends UsernamePasswordToken {
    
        /**
         * 自定义UsernamePasswordToken
         */
        private String loginType;
    
        public String getLoginType() {
            return loginType;
        }
    
        public void setLoginType(String loginType) {
            this.loginType = loginType;
        }
    
        public NewToken(String username, String password, String loginType) {
            super(username, password);
            this.loginType = loginType;
        }
    }
```
## 3.在shiroConfig配置类中配置
```java
     //自定义ModularRealmAuthenticator
        @Bean
        public ModularRealmAuthenticator modularRealmAuthenticator(){
            NewAuthenticator newAuthenticator = new NewAuthenticator();
            //认证策略
            newAuthenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
            return newAuthenticator;
        }
    
        //创建realm实体bean,交给spring容器
        @Bean(name="loginRealm")
        public StuLoginRealm getLoginRealm(@Qualifier("CredentialsMatcher") CredentialsMatcher credentialsMatcher){
            StuLoginRealm stuLoginRealm = new StuLoginRealm();
            stuLoginRealm.setCredentialsMatcher(credentialsMatcher);
            System.out.print("realm已经加载");
            return stuLoginRealm;
        }
    
        @Bean(name="manLoginRealm")
        public ManLoginRealm getManLoginRealm(@Qualifier("CredentialsMatcher") CredentialsMatcher credentialsMatcher){
            ManLoginRealm manLoginRealm = new ManLoginRealm();
            manLoginRealm.setCredentialsMatcher(credentialsMatcher);
            System.out.print("Manrealm已经加载");
            return manLoginRealm;
        }
    
        //创建securityManager 关联realm
        @Bean(name="securityManager")
        public DefaultWebSecurityManager getSecurityManager(@Qualifier("CredentialsMatcher") CredentialsMatcher credentialsMatcher){
            DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
            //配置自定义的modularRealmAuthenticator
            securityManager.setAuthenticator(modularRealmAuthenticator());
            //配置realms
            List<Realm> realms = new ArrayList<>();
            realms.add(getLoginRealm(credentialsMatcher()));
            realms.add(getManLoginRealm(credentialsMatcher()));
            securityManager.setRealms(realms);
            System.out.print("安全管理器已经加载");
            return securityManager;
        }
    
        //shiro AOP 支持
        @Bean
        public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(){
            AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor = new AuthorizationAttributeSourceAdvisor();
            authorizationAttributeSourceAdvisor.setSecurityManager(getSecurityManager(credentialsMatcher()));
            return authorizationAttributeSourceAdvisor;
        }
```
