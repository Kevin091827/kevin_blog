**@Bean指定初始化和销毁方法**

原来我们可以通过xml的方式配置bean，并且配置其初始化和销毁的方法
```xml
    	   <bean id="student" class="main.java.pojo.Student" scope="singleton" init-method="init" destroy-method="destory">
                <property name="age" value="12"></property>
                <property name="studentName" value="asdawd"></property>
                <property name="user">
                    <ref bean="user"></ref>
                </property>
            </bean>
```
我们也可以通过@bean注解进行配置
```java
    public class Teacher {
    
        public Teacher() {
            System.out.println("构造teacher对象");
        }
    
        public void init(){
            System.out.println("初始化");
        }
    
        public void destory(){
            System.out.println("销毁");
        }
    }
```
配置注解
```java
       /**
         * 单实例情况下，spring容器会管理bean的生命周期，会在容器创建的时候构造和初始化对象，在容器关闭的时候，销毁对象
         * 多实例情况下，spring容器只负责对象的创建，创建完成之后的生命周期，spring容器不再管理
         * @return
         */
        @Bean(initMethod = "init",destroyMethod = "destory")
        public Teacher teacher(){
            return new Teacher();
        }
```
测试：
```java
    	    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfig.class);
            System.out.println("ioc容器启动");         
            Teacher teacher = (Teacher) applicationContext.getBean("teacher");
            System.out.println(teacher);
            //关闭容器
            ((AnnotationConfigApplicationContext) applicationContext).close();
```
结果：
```shell
    构造teacher对象
    初始化
    三月 03, 2019 7:07:29 下午 org.springframework.context.annotation.AnnotationConfigApplicationContext doClose
    信息: Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@6267c3bb: startup date [Sun Mar 03 19:07:29 GMT+08:00 2019]; root of context hierarchy
    ioc容器启动
    main.java.pojo.Teacher@453da22c
    销毁
```
从结果可以看到：
单实例的bean在容器一启动时就会构造并初始化对象，当容器关闭时销毁对象

看看多实例情况下
	
```java
        @Scope("prototype")
        @Bean(initMethod = "init",destroyMethod = "destory")
        public Teacher teacher(){
            return new Teacher();
        }
```
结果：
```shell
    ioc容器启动
    构造teacher对象
    初始化
    main.java.pojo.Teacher@49e202ad
```
结论：多实例情况下，1.当我们不去获取这个bean的时候，spring容器再启动后不会去构造和初始化对象，只有我们主动去获取，才会初始化对象；2.spring容器在帮我们初始化对象后，并不会帮我们销毁对象。

**InitializingBean和DisposableBean**

```java
    /**
     * 实现InitializingBean和DisposableBean接口完成bean的初始化和销毁
     */
    public class Teacher implements InitializingBean, DisposableBean {
    
        public Teacher() {
            System.out.println("构造teacher对象");
        }
    
        /**
         * spring容器关闭时销毁bean
         * @throws Exception
         */
        @Override
        public void destroy() throws Exception {
            System.out.println("销毁");
        }
    
        /**
         * 在属相设置完成后初始化对象
         * @throws Exception
         */
        @Override
        public void afterPropertiesSet() throws Exception {
            System.out.println("在属相设置完成后初始化对象");
        }
    }
  ```
注解配置bean
 ```java

       @Bean
        public Teacher teacher(){
            return new Teacher();
        }
 ```
结果：
```shell
    构造teacher对象
    在属相设置完成后初始化对象
    ioc容器启动
    main.java.pojo.Teacher@442675e1
    销毁
```
**@PostConstruct&@PreDestroy**

 也可以使用jsr-250的两个注解完成两个回调
  
```java
      public class Teacher {
        
            public Teacher() {
                System.out.println("构造teacher对象");
            }
            /**
             *  也可以使用jsr-250的两个注解完成两个回调
             */
            @PostConstruct  //在构建好bean，设置好相关属性后调用
            public void init(){
                System.out.println("初始化");
            }
        
            @PreDestroy //在容器关闭，销毁bean前调用
            public void destory(){
                System.out.println("销毁");
            }
        
        }
 ```
结果：
```shell
    构造teacher对象
    初始化
    ioc容器启动
    main.java.pojo.Teacher@3d3fcdb0
    销毁
```
  **BeanPostProcessor-后置处理器**
```java
    /**
     * 实现BeanPostProcessor 接口来处理bean初始化前后的操作
     */
    @Component
    public class MyBeanPostProcessor implements BeanPostProcessor {
    
        /**
         * bean初始化前后置处理器
         * @param o
         * @param s
         * @return
         * @throws BeansException
         */
        @Override
        public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
            System.out.println(" bean初始化前后置处理器"+s);
            return o;
        }
    
        /**
         * bean初始化完成后 后置处理器
         * @param o
         * @param s
         * @return
         * @throws BeansException
         */
        @Override
        public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
            System.out.println(" bean初始化完成后 后置处理器"+s);
            return o;
        }
    }
```
**BeanPostProcessor原理**

```java
    BeanPostProcessor原理
     * populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
     * initializeBean
     * {
     * applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);初始化前方法
     * invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
     * applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);初始化后方法
     *}
```

所以综上所述：就可以大致看到bean的一个完整的生命周期：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190303200327551.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)