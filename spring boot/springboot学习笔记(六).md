今日内容：

 1. 复习拦截器，过滤器
 2. 写一个实现拦截器的登录功能
 3. 区分拦截器，过滤器
 
 # 一，复习拦截器，过滤器

## 1.拦截器

**1.拦截器概念**

是指通过统一拦截从浏览器发出的请求来完成功能的增强，可以是请求前增强，也可以请求后增强。说白了就是在一个流程正在进行的时候，你希望干预它的进展，甚至终止它进行，这是拦截器做的事情，在实现上基于Java的反射机制，属于面向切面编程（AOP）的一种运用。同时一个拦截器实例在一个controller生命周期之内可以多次调用。但是缺点是只能对controller请求进行拦截，对其他的一些比如直接访问静态资源的请求则没办法进行拦截处理

**2.使用场景：**

* 1、日志记录：记录请求信息的日志，以便进行信息监控、信息统计、计算PV（Page View）等。

* 2、权限检查：如登录检测，进入处理器检测检测是否登录，如果没有直接返回到登录页面；(待会实现这样的一个小功能)

* 3、性能监控：有时候系统在某段时间莫名其妙的慢，可以通过拦截器在进入处理器之前记录开始时间，在处理完后记录结束时间，从而得到该请求的处理时间（如果有反向代理，如apache可以自动记录）；

* 4、通用行为：读取cookie得到用户信息并将用户对象放入请求，从而方便后续流程使	用，还有如提取Locale、Theme信息等，只要是多个处理器都需要的即可使用拦截器实现。

* 5、OpenSessionInView：如hibernate，在进入处理器打开Session，在完成后关闭Session

## 2.过滤器

**1.过滤器概念**

依赖于servlet容器。在实现上基于函数回调，可以对几乎所有请求进行过滤，但是缺点是一个过滤器实例只能在容器初始化时调用一次。使用过滤器的目的是用来做一些过滤操作，获取我们想要获取的数据，比如：在过滤器中修改字符编码；在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等。

**2.使用场景**

* 1.统一项目字符编码
* 2.过滤非法url请求
* 3.在过滤器中修改HttpServletRequest的一些参数，包括：过滤低俗文字、危险字符等


# 二，拦截器实现登录权限管理

1.先写一个登录拦截器

```java
    @Configuration
    public class LoginInterceptor2 implements HandlerInterceptor {
    
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    
            Logger logger = LoggerFactory.getLogger(getClass());
            HttpSession session = request.getSession();
            logger.info("进入登录拦截器");
    
            if(session.getAttribute("username")==null){
                logger.info("还没有登录,返回登录界面");
                response.sendRedirect(request.getContextPath()+"/index.jsp");
                return false;
            }
            logger.info("已经登录");
            return true;
        }
   
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    
        }
    
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    
        }
    }
```

2.写一个拦截器配置类，注册拦截器

```java
    //WebMvcConfigurerAdapter 在spring boot 2.0过期，解决方法：实现WebMvcConfigurer接口
    //拦截器配置类，让spring boot可以扫描到
    @Configuration
    public class ConfigInterceptor implements WebMvcConfigurer {
    
        @Autowired
        private LoginInterceptor2 loginInterceptor2;
    
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
    
            //需要拦截的路径
            String[] addpathpatterns ={
                    "/boot/*"
            };
    
            //不需要拦截的路径
            String[] excludepathpatterns = {
                      "/boot/dologin",
    
            };
    
            //注册拦截器
            InterceptorRegistration interceptorRegistry = registry.addInterceptor( loginInterceptor2).
                    addPathPatterns(addpathpatterns).excludePathPatterns(excludepathpatterns);
        }
    }

```

3.编写controller层代码

```java

    @Controller
    public class JspController {
    
    
        @RequestMapping("/boot/dologin")
        public String dologin(HttpServletRequest request,Model model, @RequestParam("userName") int userName, @RequestParam("pwd") int pwd){
    
            Logger logger = LoggerFactory.getLogger(getClass());
            logger.info("dologin");
            HttpSession session = request.getSession();
    
            if(userName==123 && pwd==123){
                logger.info("if");
                session.setAttribute("username",userName);
                logger.info("msg");
                model.addAttribute("msg","登录成功");
                return "redirect:/boot/hello";
            }
            else{
                logger.info("else");
                return "redirect:/boot/configInfo";
            }
        }
    
    }
```
第二个
```java
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
4.测试一下

先成功登录一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122715143514.png)

登录成功，跳转页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151443527.png)

登录成功的基础上，访问其他页面，同样可以跳转

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151525345.png)

控制台日志输出：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151554381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

我们重新打开浏览器，重新登录，但是在登录之前，我们试着去请求另一个页面

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151652128.png)

但是结果是失败的，自动跳转到登录界面来，必须登录才有操作权限

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151659859.png)

控制台输出：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151805544.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 三，区分拦截器和过滤器

**1.执行流程**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181227151921806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

多个的时候

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122715275985.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**2，区别**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2018122715285565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)