# Spring中的ioc和DI
ioc：控制反转，DI：依赖注入，其实，这两个是一个相同的概念，ioc偏重于是一种思想，di则是偏重于过程。
## 一，ioc
#### 何为ioc？

**ioc：字面意思就是控制反转，那么是谁控制谁呢，为什么是反转不是正转呢？**

   原来，不是spring框架的时候，我们实例化对象，都是直接在程序内部通过new来创建对象，是程序主动去创建对象，主动权在程序或者在我们手上，而Spring IoC容器，负责实例化，配置和组装上述bean，即由Ioc容器来控制对象的创建,传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象，也就是正转；而反转则是由容器来帮忙创建及注入依赖对象；为何是反转？因为由容器帮我们查找及注入依赖对象，对象只是被动的接受依赖对象，所以是反转；哪些方面反转了？依赖对象的获取被反转了。所以现在控制权不在程序，而是ioc容器，控制权反转。

**ioc能做什么？**

IoC 不是一种技术，只是一种思想，一个重要的面向对象编程的法则，它能指导我们如何设计出松耦合、更优良的程序。传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了IoC容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是 松散耦合，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。

　　其实IoC对编程带来的最大改变不是从代码上，而是从思想上，发生了“主从换位”的变化。应用程序原本是老大，要获取什么资源都是主动出击，但是在IoC/DI思想中，应用程序就变成被动的了，被动的等待IoC容器来创建并注入它所需要的资源了。

　　IoC很好的体现了面向对象设计法则之一—— 好莱坞法则：“别找我们，我们找你”；即由IoC容器帮对象找相应的依赖对象并注入，而不是由对象主动去找。
## 二，DI
### 依赖
#### 1.何为依赖注入
　DI—Dependency Injection，即“依赖注入”：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。通过依赖注入机制，我们只需要通过简单的配置，而无需任何代码就可指定目标需要的资源，完成自身的业务逻辑，而不需要关心具体的资源来自何处，由谁实现。

　　**理解DI的关键是：“谁依赖谁，为什么需要依赖，谁注入谁，注入了什么”，那我们来深入分析一下：**

　　●**谁依赖于谁**：当然是应用程序依赖于IoC容器；

　　●**为什么需要依赖**：应用程序需要IoC容器来提供对象需要的外部资源；

　　●**谁注入谁**：很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；

　　●**注入了什么**：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。
　　</br>
#### 2.基于构造方法的依赖注入
实体类User
```java
    public class User {
        private String userName;
        private int age;
        private Student student;
    
        public User() {
            System.out.println("User构造函数");
        }
            
        public User(String userName,int age,Student student){
            this.userName=userName;
            this.age=age;
            this.student=student;
            System.out.println("User有参构造函数");
        }
    
        public Student getStudent() {
            return student;
        }
    
        public void setStudent(Student student) {
            this.student = student;
        }
    
        public String getUserName() {
            return userName;
        }
    
        public void setUserName(String userName) {
            this.userName = userName;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    }
```
实体类Student

```java


    public class Student {
    
        private String studentName;
        private int age;
    
        public Student(String studentName, int age) {
            System.out.println("student有参构造方法");
            this.studentName = studentName;
            this.age = age;
        }
    
        public Student() {
            System.out.println("student无参构造方法");
        }
    
        public String getStudentName() {
            return studentName;
        }
    
        public void setStudentName(String studentName) {
            this.studentName = studentName;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    }
```
##### a.基于参数索引（默认是0开始）使用index属性显式指定构造函数参数的索引
  ```xml
    <!--基于构造函数参数索引 除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义-->
        <bean id="user" class="main.java.pojo.User">
            <constructor-arg index="0" value="kevin"></constructor-arg>
            <constructor-arg index="1" value="100"></constructor-arg>
            <constructor-arg ref="student"></constructor-arg>
        </bean>
 ```
 
   ##### b.基于参数类型 
   ```xml
        <!--基于构造函数参数类型 如果使用type显式指定构造函数参数的类型，则容器可以使用与简单类型匹配的类型type Spring无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配 -->
        <bean id="user" class="main.java.pojo.User">
            <constructor-arg type="java.lang.String" value="kevin"></constructor-arg>
            <constructor-arg type="int" value="100"></constructor-arg>
            <constructor-arg ref="student"></constructor-arg>
        </bean>
   ```
##### c.基于参数名字
  ```xml   
       <!--基于参数名字注入 还可以使用构造函数参数名称进行值消歧    -->
        <bean id="user" class="main.java.pojo.User">
            <constructor-arg name="userName" value="kevin"></constructor-arg>
            <constructor-arg name="age" value="100"></constructor-arg>
            <constructor-arg ref="student"></constructor-arg>
        </bean>
  ```
student注入
  ```xml 

    <bean id="student" class="main.java.pojo.Student">
            <constructor-arg index="0" value="student"></constructor-arg>
            <constructor-arg index="1" value="12"></constructor-arg>
    </bean>
```
#### 3.基于setter的依赖注入
```xml
		<bean id="student" class="main.java.pojo.Student">
           <property name="age" value="12"></property>
            <property name="studentName" value="asdawd"></property>
        </bean>
        <bean id="user" class="main.java.pojo.User">
            <property name="userName" value="kevin"></property>
            <property name="age" value="18"></property>
            <property name="student" ref="student"></property>
        </bean>
```
#### 1.循环依赖问题
**什么是循环依赖问题？**

当我们的bean A依赖于bean B，bean B也依赖于bean A 时

**类似如下：**
```java
    public Student(String studentName, int age,User user) {
            System.out.println("student有参构造方法");
            this.studentName = studentName;
            this.age = age;
            this.user=user;
        }
    public User(String userName,int age,Student student){
            this.userName=userName;
            this.age=age;
            this.student=student;
            System.out.println("User有参构造函数");
        }

```
如果我采用构造注入的方式实例化bean此时：
##### a.构造器参数循环依赖
```xml
			<bean id="student" class="main.java.pojo.Student">
                <constructor-arg type="java.lang.String" value="kevinawdaw"></constructor-arg>
                <constructor-arg type="int" value="1002213"></constructor-arg>
                <constructor-arg ref="user"></constructor-arg>
            </bean>
          
            <bean id="user" class="main.java.pojo.User">
                <constructor-arg name="userName" value="kevin"></constructor-arg>
                <constructor-arg name="age" value="100"></constructor-arg>
                <constructor-arg ref="student"></constructor-arg>
            </bean>

```
<p>报错信息</p>
   
```shell
     Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException:   
            Error creating bean with name 'a': Requested bean is currently in creation: Is there an unresolvable circular reference? 
```
###### 为什么使用构造方法注入会出现循环依赖问题？

Spring容器会将每一个正在创建的Bean 标识符放在一个“当前创建Bean池”中，Bean标识符在创建过程中将一直保持在这个池中，因此如果在创建Bean过程中发现自己已经在“当前创建Bean池”里时将抛出BeanCurrentlyInCreationException异常表示循环依赖；而对于创建完毕的Bean将从“当前创建Bean池”中清除掉。
也就是说，当我们的spring容器先创建出单例的student的时候也创建了user，并且将student，user放入了当前正在创建bean池中，那么当容器接着去创建user实例时发现池中已经存在了该bean，便抛出异常。

##### b.setter循环依赖
那么我们使用setter注入看看会如何
```xml
   		   <bean id="student" class="main.java.pojo.Student">
               <property name="age" value="12"></property>
                <property name="studentName" value="asdawd"></property>
                <property name="user" ref="user"></property>
            </bean>
            <bean id="user" class="main.java.pojo.User">
                <property name="userName" value="kevin"></property>
                <property name="age" value="18"></property>
                <property name="student" ref="student"></property>
            </bean>
```
   结果没有任何异常

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190225222008931.png)

###### 为什么使用setter注入可以解决循环依赖问题？

Spring是先将Bean对象实例化之后再设置对象属性的,所以不存在上述类似构造函数注入的问题。

#### 4.详细配置
##### a.普通值注入value
	  
```xml
    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <!-- results in a setDriverClassName(String) call -->
        <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
        <property name="username" value="root"/>
        <property name="password" value="masterkaoli"/>
    </bean>
```
也可以使用p命名空间简化配置

    引入约束： xmlns:p="http://www.springframework.org/schema/p"
配置：
```xml
    <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
            destroy-method="close"
            p:driverClassName="com.mysql.jdbc.Driver"
            p:url="jdbc:mysql://localhost:3306/mydb"
            p:username="root"
            p:password="masterkaoli"/>
```
##### b.引用注入

**注意：idref和ref的区别**

ref是将bean的实例注入，即将student类实例注入user
```xml
   			  <bean id="user" class="main.java.pojo.User">
                    <property name="userName" value="kevin"></property>
                    <property name="age" value="18"></property>
                    <property name="student" ref="student"></property>
                </bean>
```
而idref是将目标bean的id注入而不是bean实例
```xml
    <bean id="theTargetBean" class="..."/>
    
    <bean id="theClientBean" class="...">
        <property name="targetName">
            <idref bean="theTargetBean"/>
        </property>
    </bean>
    
    上面的bean定义片段与以下片段完全等效（在运行时）：
    <bean id="theTargetBean" class="..." />
    
    <bean id="client" class="...">
        <property name="targetName" value="theTargetBean"/>
    </bean>
```
###### 内部bean
 ```xml

       <bean id="student" class="main.java.pojo.Student">
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
##### c.集合注入

在`<list/>，<set/>，<map/>，和<props/>`元素，你将Java的性能和参数Collection类型List，
   ```xml
    Set，Map，和Properties分别。

    <bean id="moreComplexObject" class="example.ComplexObject">
        <!-- results in a setAdminEmails(java.util.Properties) call -->
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.org</prop>
                <prop key="support">support@example.org</prop>
                <prop key="development">development@example.org</prop>
            </props>
        </property>
        <!-- results in a setSomeList(java.util.List) call -->
        <property name="someList">
            <list>
                <value>a list element followed by a reference</value>
                <ref bean="myDataSource" />
            </list>
        </property>
        <!-- results in a setSomeMap(java.util.Map) call -->
        <property name="someMap">
            <map>
                <entry key="an entry" value="just some string"/>
                <entry key ="a ref" value-ref="myDataSource"/>
            </map>
        </property>
        <!-- results in a setSomeSet(java.util.Set) call -->
        <property name="someSet">
            <set>
                <value>just some string</value>
                <ref bean="myDataSource" />
            </set>
        </property>
    </bean>
```
##### d.空值注入
Spring将属性等的空参数视为空Strings。以下基于XML的配置元数据片段将email属性设置为空 String值（“”）。
```xml
    <bean class="ExampleBean">
        <property name="email" value=""/>
    </bean>
```
上面的示例等效于以下Java代码：
```xml
    exampleBean.setEmail("");
```
该<null/>元素处理null值。例如：
```xml
    <bean class="ExampleBean">
        <property name="email">
            <null/>
        </property>
    </bean>
```
以上配置等同于以下Java代码：
```java
    exampleBean.setEmail(null);
```
##### e.懒加载
默认情况下，ApplicationContext实现会急切地创建和配置所有 单例 bean，作为初始化过程的一部分。通常，这种预先实例化是可取的，因为配置或周围环境中的错误是立即发现的，而不是几小时甚至几天后。如果不希望出现这种情况，可以通过将bean定义标记为延迟初始化来阻止单例bean的预实例化。延迟初始化的bean告诉IoC容器在第一次请求时创建bean实例，而不是在启动时。

在XML中，此行为由 元素lazy-init上的属性控制<bean/>; 例如：
```xml
    <bean id="lazy" class="com.foo.ExpensiveToCreateBean" lazy-init="true"/>
    <bean name="not.lazy" class="com.foo.AnotherBean"/>
```
当前面的配置被a使用时ApplicationContext，命名的bean lazy在ApplicationContext启动时不会急切地预先实例化，而not.lazybean被急切地预先实例化。

但是，当延迟初始化的bean是未进行延迟初始化的单例bean的依赖项时 ，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。惰性初始化的bean被注入到其他地方的单独的bean中，而这个bean并不是惰性初始化的。

您还可以通过使用元素default-lazy-init上的属性来控制容器级别的延迟初始化 <beans/>; 例如：
```xml
    <beans default-lazy-init="true">
        <!-- no beans will be pre-instantiated... -->
    </beans>

```