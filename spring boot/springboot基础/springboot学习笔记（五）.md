今天主要内容：

 - spring boot下使用拦截器
 - spring boot下使用过滤器
 - spring boot下使用servlet

# 一，spring boot 下使用拦截器
## 1、按照Spring mvc的方式编写一个拦截器类

```java
     /**
     *自定义拦截器： 1.实现HandlerInterceptor接口
     *              2.继承HandlerInterceptorAdapter抽象类（该抽象类实现了HandlerInterceptor接口）
     */
    @Configuration  //spring boot 2.0这个要加，不然报错
    public class LoginInterceptor implements HandlerInterceptor {
    
        /**
         * preHandle:预处理回调方法
         * 作用：实现处理器（controller）的预处理（如：权限检查）
         * 返回值为true则继续流程，继续controller
         * 返回值为false则终止流程，使用response来产生响应
         *
         */
    
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    
            System.out.print("已经进入拦截器");
            //拦截器逻辑
            return true;
        }
    
        /**
         *当preHandle执行且返回true后执行
         *postHandle:后处理回调方法
         *作用：实现对处理器的后处理（但在视图渲染之前）
         *此时可以通过modelAndView对模型数据或对视图进行处理
         */
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    
        }
    
        /*
        *afterCompletion：整个请求处理完毕回调方法
        *性能监控中我们可以在此记录结束时间并输出消耗时间，
        *还可以进行一些资源清理，类似于try-catch-finally中的finally，
        *但仅调用处理器执行链中preHandle返回true的拦截器的afterCompletion。
        **/
    
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    
        }
    }
```
## 2.写一个拦截器配置类，实现WebMvcConfigurer接口,覆盖其中的方法并添加已经编写好的拦截器：

```java
      //WebMvcConfigurerAdapter 在spring boot 2.0过期，解决方法：实现WebMvcConfigurer接口
    //拦截器配置类，让spring boot可以扫描到
    @Configuration
    public class ConfigInterceptor implements WebMvcConfigurer {
        @Autowired
        private LoginInterceptor loginInterceptor;
    
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
    
            //需要拦截的路径
            String[] addpathpatterns ={
                    "/boot/*"
            };
    
            //不需要拦截的路径
            String[] excludepathpatterns = {
    
                  //  "/index.jsp",
                      "/boot/dologin",
                    //"/success.jsp"
    
            };
    
            //注册拦截器
            InterceptorRegistration interceptorRegistry = registry.addInterceptor( loginInterceptor).
                    addPathPatterns(addpathpatterns).excludePathPatterns(excludepathpatterns);
        }
    }
```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227143718791.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227143826400.png)

# 二，spring boot下使用过滤器

spring boot 下使用过滤器有两种方式：


## 第一种实现方式

### 1.先编写一个普通的Filter
```java
    /**
     * 所有的过滤器都必须实现Filter接口
     */
    //
    @WebFilter("/*")//Filter要过滤的路径
    public class MyFilter implements Filter {
    
        //过滤器是在Web服务器启动加载当前项目完毕以后自动实例化
        //在初始化过滤器的时候**执行1次**
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
        }
    
        // 执行过滤的功能，**每次请求都会执行这个方法**
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                throws IOException, ServletException {
            System.out.println("您已进入filter过滤器，您的请求正常，请继续遵规则...");
            chain.doFilter(request, response);
        }
    
        //在服务器关闭的时候销毁，执行1次
        @Override
        public void destroy() {
        }
    }
     
```
### 2、在main方法的主类上添加注解：`@ServletComponentScan(basePackages{"com.bjpowernode.springboot.servlet", "com.bjpowernode.springboot.filter"})`

```java
    //spring Boot的入口main方法
    @EnableTransactionManagement//开始spring boot事务支持
    @MapperScan("com.springboot.demo.dao")
    @ServletComponentScan( basePackages = {"com.springboot.demo.servlet","com.springboot.demo.Filter"},basePackageClasses =org.springframework.web.filter.CharacterEncodingFilter.class)
    @SpringBootApplication//spring boot的核心注解，作用：开启spring的自动配置
    public class Application {
    
        public static void main(String[] args) {
    
            //启动了spring Boot程序，启动spring容器，内嵌的tomcat
            SpringApplication.run(Application.class, args);
        }
    
    }

```

## 第二种方式：通过Spring boot的配置类实现

### 1.先写一个普通Filter

```java
    @Configuration//也要加
    public class MyFilter2 implements Filter {
    
        //过滤器是在Web服务器启动加载当前项目完毕以后自动实例化
        //在初始化过滤器的时候**执行1次**
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {
        }
    
        // 执行过滤的功能，**每次请求都会执行这个方法**
        @Override
        public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                throws IOException, ServletException {
            System.out.println("您已进入filter过滤器");
            chain.doFilter(request, response);
        }
    
        //在服务器关闭的时候销毁，执行1次
        @Override
        public void destroy() {
        }
    }
```

### 2.写一个filter配置类

```java
      @Configuration
    public class FilterConfig {
    
        @Autowired
        private MyFilter2 myFilter2;
        /*
        * @Bean <bean id=""  class=""></bean>
        **/
    
        //@Bean是一个方法级别上的注解，主要用在@Configuration注解的类里，也可以用在@Component注解的类里。添加的bean的id为方法名
        @Bean
        public FilterRegistrationBean filterRegistrationBean(){
            //注册过滤器
            FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean(myFilter2);
            //添加过滤路径
            filterRegistrationBean.addUrlPatterns("/*");
            return filterRegistrationBean;
        }
    }
 ```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227143718791.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227143826400.png)


# 三，在spring boot下使用servlet

## 1.第一种方式：

### 1.写一个servlet
  ```java
    @WebServlet("/myservlet")//访问的url
    public class MyServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.getWriter().print("hello,myservlet");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            this.doGet(req,resp);
        }
    }
 ```
### 2.在main方法的主类上添加注解：
```java
@ServletComponentScan(basePackages="com.bjpowernode.servlet")
```
## 2.第二种方式

### 1.写一个普通servlet

```java
    public class MyServlet2 extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            resp.getWriter().print("hello,myservlet");
        }
    
        @Override
        protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            this.doGet(req,resp);
        }
    }

```

2.写一个配置类

```java
    @Configuration
    public class ServletConfig{
    
        @Bean
        public ServletRegistrationBean servletRegistrationBean(){
    
            ServletRegistrationBean registrationBean = new ServletRegistrationBean(new MyServlet2(),"/myservlet2");
            return registrationBean;
        }
    
        /**
         *等价于原来配置web.xml中的characterFilter
         **/
        @Bean
        public FilterRegistrationBean filterRegistrationBean() {
            FilterRegistrationBean registrationBean = new FilterRegistrationBean();
            CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
            characterEncodingFilter.setForceEncoding(true);
            characterEncodingFilter.setEncoding("UTF-8");
            registrationBean.setFilter(characterEncodingFilter);
            registrationBean.addUrlPatterns("/*");
            return registrationBean;
        }
    }

```


