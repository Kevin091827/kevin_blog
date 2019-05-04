**bean的作用域**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019022721302947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**1.singleton**

在bean定义中把bean的范围设置成单例的时候，Spring Ioc容器会根据bean的定义只创建一个实例。此单个实例会被存在一个单例bean的缓存中，后面的所有请求和对这个bean的指向，都会返回缓存中的bean实例。Spring依赖注入Bean实例默认是单例的。
```xml
    <bean id="student" class="main.java.pojo.Student" scope="singleton" init-method="init" destroy-method="destory">
                <property name="age" value="12"></property>
                <property name="studentName" value="asdawd"></property>
                <property name="user">
                    <bean class="main.java.pojo.User"
                          p:age="15"
                          p:userName="12315">
                    </bean>
                </property>
            </bean>
```
测试

```java

       public static void main(String[] args) {
        
                /**
                 * spring源码分析
                 * 测试ioc容器bean的实例化
                 */
                ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("main/resource/beans.xml");
                Student student = (Student) classPathXmlApplicationContext.getBean("student");
                Student student1 = (Student) classPathXmlApplicationContext.getBean("student");
                Student student2 = (Student) classPathXmlApplicationContext.getBean("student");
                System.out.println(student.getStudentName());
                System.out.println(student.getAge());
                System.out.println(student.getUser().getAge());
                System.out.println(student.getUser().getUserName());
                System.out.println(student == student1);
            }
```
结果：
```shell
    student无参构造方法
    User构造函数
    初始化
    asdawd
    12
    15
    12315
    true
```
**那么spring底层是如何实现单例的呢？**

Spring框架对单例的支持是采用单例注册表的方式进行实现的

源码如下：
```java
    public abstract class AbstractBeanFactory implements ConfigurableBeanFactory{  
           /** 
            *  singletonCache 充当了Bean实例的缓存，实现方式和单例注册表相同 
            */  
           private final Map singletonCache=new HashMap();  
           public Object getBean(String name)throws BeansException{  
               return getBean(name,null,null);  
           }  
        ...  
           public Object getBean(String name,Class requiredType,Object[] args)throws BeansException{  
              //对传入的Bean name稍做处理，防止传入的Bean name名有非法字符(或则做转码)  
              String beanName=transformedBeanName(name);  
              Object bean=null;  
              //手工检测单例注册表  
              Object sharedInstance=null;  
              //使用了代码锁定同步块，原理和同步方法相似，但是这种写法效率更高  
              synchronized(this.singletonCache){  
                 sharedInstance=this.singletonCache.get(beanName);  
               }  
              if(sharedInstance!=null){  
                 ...  
                 //返回合适的缓存Bean实例  
                 bean=getObjectForSharedInstance(name,sharedInstance);  
              }else{  
                ...  
                //取得Bean的定义  
                RootBeanDefinition mergedBeanDefinition=getMergedBeanDefinition(beanName,false);  
                 ...  
                //根据Bean定义判断，此判断依据通常来自于组件配置文件的单例属性开关  
                //<bean id="date" class="java.util.Date" scope="singleton"/>  
                //如果是单例，做如下处理  
                if(mergedBeanDefinition.isSingleton()){  
                   synchronized(this.singletonCache){  
                    //再次检测单例注册表  
                     sharedInstance=this.singletonCache.get(beanName);  
                     if(sharedInstance==null){  
                        ...  
                       try {  
                          //真正创建Bean实例  
                          sharedInstance=createBean(beanName,mergedBeanDefinition,args);  
                          //向单例注册表注册Bean实例  
                           addSingleton(beanName,sharedInstance);  
                       }catch (Exception ex) {  
                          ...  
                       }finally{  
                          ...  
                      }  
                     }  
                   }  
                  bean=getObjectForSharedInstance(name,sharedInstance);  
                }  
               //如果是非单例，即prototpye，每次都要新创建一个Bean实例  
               //<bean id="date" class="java.util.Date" scope="prototype"/>  
               else{  
                  bean=createBean(beanName,mergedBeanDefinition,args);  
               }  
        }  
        ...  
           return bean;  
        }  
        }
```
**还有就是，spring的单例bean和普通的单例模式有什么区别吗？**

spring的单例模式和普通的单例模式的区别在于：他们关联的环境不同
1.spring的单例模式是指在一个spring容器中只有一个实例
2.普通单例模式是在一个jvm进程中只有一个实例

如果我创建多个spring容器，如果有多个Spring容器，即使是单例bean，也一定会创建多个实例

代码：
```java
  		    ClassPathXmlApplicationContext classPathXmlApplicationContext1 = new ClassPathXmlApplicationContext("main/resource/beans.xml");
            User user1 =(User) classPathXmlApplicationContext1.getBean("user");
            ClassPathXmlApplicationContext classPathXmlApplicationContext2 = new ClassPathXmlApplicationContext("main/resource/beans.xml");
            User user2 =(User) classPathXmlApplicationContext2.getBean("user");
            System.out.println(user1 == user2);
```
beans.xml显示声明为单例
```xml
  		    <bean class="main.java.pojo.User" id="user" scope="singleton"
                p:age="15"
                 p:userName="12315">
             </bean>
```
        
结果：
```shell
    三月 02, 2019 3:30:14 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
    信息: Loading XML bean definitions from class path resource [main/resource/beans.xml]
    User构造函数
    student无参构造方法
    初始化
    三月 02, 2019 3:30:14 下午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
    信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@56ac3a89: startup date [Sat Mar 02 15:30:14 GMT+08:00 2019]; root of context hierarchy
    三月 02, 2019 3:30:14 下午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
    信息: Loading XML bean definitions from class path resource [main/resource/beans.xml]
    User构造函数
    student无参构造方法
    初始化
    false
```
所以：Spring的单例bean与Spring bean管理容器密切相关，每个容器都会创建自己独有的实例，所以与普通单例模式相差极大，但在实际应用中，如果将对象的生命周期完全交给Spring管理(不在其他地方通过new、反射等方式创建)，其实也能达到单例模式的效果

**prototype作用域**

非单例的，prototype范围的bean，在每次对bean实例的请求都会创建一个新的bean的实例
```xml
    <bean id="student" class="main.java.pojo.Student" scope="prototype" init-method="init" destroy-method="destory">
                <property name="age" value="12"></property>
                <property name="studentName" value="asdawd"></property>
                <property name="user">
                    <bean class="main.java.pojo.User"
                          p:age="15"
                          p:userName="12315">
                    </bean>
                </property>
            </bean>
```
结果：

    student无参构造方法
    User构造函数
    初始化
    student无参构造方法
    User构造函数
    初始化
    student无参构造方法
    User构造函数
    初始化
    asdawd
    12
    15
    12315
    false
构造函数执行多次


问题：

**1.一个多例的bean需要注入一个一个单例的bean，结果如何？**

```xml
	 
   		   <bean class="main.java.pojo.User" id="user"
                p:age="15"
                 p:userName="12315">
             </bean>
            <bean id="student" class="main.java.pojo.Student" scope="prototype" init-method="init" destroy-method="destory">
                <property name="age" value="12"></property>
                <property name="studentName" value="asdawd"></property>
                <property name="user">
                    <ref bean="user"></ref>
                </property>
            </bean>
```
上边user是单例的，而student是多例
测试代码；
 ```java

    public static void main(String[] args) {
            /**
             * spring源码分析
             * 测试ioc容器bean的实例化
             */
            ClassPathXmlApplicationContext classPathXmlApplicationContext = new ClassPathXmlApplicationContext("main/resource/beans.xml");
            Student student = (Student) classPathXmlApplicationContext.getBean("student");
            Student student1 = (Student) classPathXmlApplicationContext.getBean("student");
            Student student2 = (Student) classPathXmlApplicationContext.getBean("student");
            System.out.println(student.getStudentName());
            System.out.println(student.getAge());
            System.out.println(student.getUser());
            System.out.println(student1.getUser());
            System.out.println(student.getUser() == student1.getUser());
            System.out.println(student.getUser().getAge());
            System.out.println(student.getUser().getUserName());
            System.out.println(student == student1);
        }
```

结果：

    User构造函数
    student无参构造方法
    初始化
    student无参构造方法
    初始化
    student无参构造方法
    初始化
    asdawd
    12
    main.java.pojo.User@56ac3a89
    main.java.pojo.User@56ac3a89
    true
    15
    12315
    false

总结：单例的user还是没受影响，这种情况下是可以存在的

**2.单例bean需要注入一个多例bean**

单例Bean只有一次初始化的机会，它的依赖关系只有在初始化阶段被设置，而它所依赖的多例Bean会不断更新产生新的Bean实例，这将导致单例Bean所依赖的多例Bean得不到更新，每次都得到的是最开始时生成的Bean，这就违背了使用多例的初衷。 
解决该问题有两种解决思路： 
1. 放弃依赖注入：主动向容器获取多例，可以实现ApplicationContextAware接口来获取ApplicationContext实例,通过ApplicationContext获取多例对象。 
2. 利用方法注入：方法注入是让Spring容器重写Bean中的抽象方法，该方法返回多例，Spring通过CGLIb修改客户端的二进制代码来实现。 


**3.bean的生命周期**

1. spring容器可以管理单例下的bean,在此作用域下，Spring能够精确地知道Bean何时被创建，何时初始化完成，以及何时被销毁
2.  对于prototype作用域的Bean，Spring只负责创建，当容器创建了Bean的实例后，Bean的实例就交给了客户端的代码管理，Spring容器将不再跟踪其生命周期，并且不会管理那些被配置成prototype作用域的Bean的生命周期

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190227225249796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
