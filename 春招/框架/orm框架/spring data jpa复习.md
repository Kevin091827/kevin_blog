# spring data jpa复习



> 本人最近一直在为秋招和明年的春招做准备，目前在复习一些框架的基础知识，ORM框架有很多种，本人比较常用的mybatis，所以在mybatis上会加深到对一些源码的认识，但是对于jpa，我不是很经常用，只是对其使用做一个基本的整理，本文的目的就是如此，今后有时间会对jpa的内部原理进行剖析，如果有在一起为明年春招做准备的小伙伴，欢迎互粉呀，共同进退

## 一，spring data jpa是什么？



> 聊到spring data jpa 就不得不从jpa聊起来了

### 1.什么是jpa？

jpa就是实现了ORM思想的一种规范。

> 何为orm思想？

orm思想简单来说就是，操作实体类对象就相当于操作数据库表，通过对实体类对象的操作完成对数据库表的操作。将实体类和数据库表，实体类属性和数据库表字段建立起了映射关系。

> 有了orm思想自然会有实现orm思想的框架

实现了orm思想的框架就有hibernate，Toplink等等，那么，jpa规范也就相应出现了，SUN公司推出的一套基于ORM的规范，内部是由一系列的接口和抽象类构成，框架提供实现

### 2.jpa规范



![](https://oscimg.oschina.net/oscnet/fd5e9c1f88bcdb6c41f6685998b44e0f068.jpg))



### 3.何为spring data jpa？

Spring Data Jpa则是在JPA之上添加另一层抽象（Repository层的实现），极大地简化持久层开发及ORM框架切换的成本。相当于在jpa规范上的一层封装



## 二，spring data jpa使用

> 下边的示例是基于springboot2.x ，spring data jpa2.1.5实现，是一个maven工程

maven依赖

```xml
    <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
    </dependencies>
```

application.properties

```properties
##########################################
spring.datasource.url=jdbc:mysql://localhost:3306/employee?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=123
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
##########################################
###spring JPA配置信息
###spring.jpa.database指定目标数据库.
spring.jpa.database = MYSQL
###spring.jps.show-sq：是否显示sql语句
spring.jpa.show-sql = true
###spring.jpa.hibernate.ddl-auto指定DDL mode (none, validate, update, create, create-drop). 当使用内嵌数据库时，默认是create-drop，否则为none.
spring.jpa.hibernate.ddl-auto = update
###spring.jpa.hibernate.naming-strategy指定命名策略
spring.jpa.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy
###指定方言
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect
```

> 关于spring.jpa.hibernate.ddl-auto

- update：原来数据库表存在则不删除，每次项目启动运行执行数据库表更新(一般推荐update)

- create：不管原来数据库表存在与否，每次项目启动运行都会重新建表，有表的则删除后重新建表
- none：什么都不干
- create-drop：每次运行程序时会先创建表结构，然后待程序结束时清空表

### 1.建立实体和表，实体属性和表字段的映射关系

```java
//标识数据库实体
@Entity
@Table(name = "tb_jpatest")//name:数据库表名
@Data
public class JpaTest {

    //指定主键和主键生成策略
    /**
     * jpa自带主键生成策略
     *      - TABLE : 使用一个特定的数据库表格来保存主键值
     *      - SEQUENCE : 根据底层数据库的序列来生成主键，条件是数据库支持序列。
     *                   这个值要与generator一起使用，generator 指定生成主键使用的生成器（可能是orcale中自己编写的序列）
     *      - IDENTITY :  主键由数据库自动生成（主要是支持自动增长的数据库，如mysql）
     *      - AUTO： 主键由程序控制，也是GenerationType的默认值
     */
    @Id
    //@GenericGenerator(name = "id",strategy = "uuid")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    //标识列属性
    @Column(name = "test_id")
    private int testId;

    @Column(name = "info",nullable = false,length = 255)
    private String info;

    @Column(name = "gmt_create")
    private Date gmtCreate;

    //标识不是数据库的列，默认是 @Basic
    @Transient
    @Override
    public String toString() {
        return "JpaTest{" +
                "id=" + id +
                ", testId=" + testId +
                ", info='" + info + '\'' +
                ", gmtCreate=" + gmtCreate +
                '}';
    }
}
```



### 2.CRUD操作

> 执行crud操作，我们需要先建立一个自己的dao层，但是，这个dao层不同于原来mybatis需要自己编写代码，和mapper.xml，我们可以在spring data jpa实现0sql，0代码

```java
public interface JpaTestDao extends JpaRepository<JpaTest,Long>, JpaSpecificationExecutor<JpaTest> {

    /**
     *  JpaRepository<T,D>     --> 封装了基本的CRUD操作
     *        需要提供两个泛型   - T:操作的实体类类型
     *                         -  D：实体类主键的类型
     *
     *  JpaSpecificationExecutor<T>   ---> 封装了复杂查询（分页，排序等等）
     *         只需要提供一个泛型    - T:操作的实体类类型
     *
     */
}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010142347643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

所以我们只需要继承JpaRepository，JpaSpecificationExecutor便可以实现大多数的增删改查操作了



#### 1.查询

##### 基于内部api的查询

> 实现了这两个接口，基本上jpa已经为我们提供了很多内部对数据库表crud的api了

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191010142601503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

save()保存或更新：

- save()方法默认jpa提供得save方法当不提供id时就是执行insert操作
- save()默认jpa提供的save方法当提供id时就是执行update操作

```java
    @Autowired
    private JpaTestDao jpaTestDao;

    /**
     * save()保存
     * @param jpaTest
     */
    @Override
    public void insert(JpaTest jpaTest) {
        jpaTest.setGmtCreate(new Date());
        jpaTest.setInfo("test1");
        //save()方法默认jpa提供得save方法当不提供id时就是执行insert操作
        jpaTestDao.save(jpaTest);
    }

    /**
     * save()更新
     * @param jpaTest
     */
    @Override
    public void update(JpaTest jpaTest) {
        jpaTest.setId((long)1);
        jpaTest.setInfo("update");
        //save()默认jpa提供的save方法当提供id时就是执行update操作
        jpaTestDao.save(jpaTest);
    }
```

根据主键查询

```java
        //findById()根据主键查询
        Optional<JpaTest> jpaTest = jpaTestDao.findById((long)2);
        System.out.println(jpaTest.toString());
        System.out.println("查询所有");
        //查询所有
        List<JpaTest> list = jpaTestDao.findAll();
        for(JpaTest j : list){
            System.out.println(j);
        }
```



但是这些内部api大多都是基于主键的操作，很多时候在开发中,并不能满足需求，所以我们需要一些复杂查询。

##### 复杂查询

> 基于Specification接口的复杂查询

```java
        jpaTestDao.findAll(new Specification<JpaTest>() {
            //实现该接口
            @Override
            public Predicate toPredicate(Root<JpaTest> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
                //构造查询条件和查询方式
                return null;
            }
        })
```

参数讲解：

- root：查询需要比较的对象属性

  ```java
                  //获取查询需要比较的属性
                  Path<Object> testId = root.get("testId");//实体类属性
  ```

  相当于sql语句中的where后边跟的字段

  ```sql
  select * from tb where testId = ''
  ```

- criteriaQuery：一般很少用到

- criteriaBuilder：查询构造器，构造查询方式

  ```java
                  //构造查询方式
                      // 1.第一个参数：需要比较的属性（path对象）
                      // 2.第二个参数：该比较的属性的取值，这里可以实现动态的传参
                  Predicate p1 = criteriaBuilder.equal(testId,"1");
  ```

  返回的Predicate就是最终的返回值

  模糊查询需要注意,不同于equals，like在构造查询方式时需要将path对象利用path.as（属性对应类型的字节码）转化才行

  ```java
                  //模糊查询需要注意
                      //模糊查询不同于equals，
                          // - equals在构造查询方式时直接传递path对象即可
                          // - like在构造查询方式时需要将path对象利用path.as（属性对应类型的字节码）转化才行
                  Predicate p3 = criteriaBuilder.like(info.as(String.class),"1%");
  ```

  也可以拼接多个条件，有and，or等等

  ```java
                  //拼接查询条件
                      //and() -- 与，是可变参数
                      //or() -- 或，也是可变参数
                  Predicate p4 = criteriaBuilder.and(p1,p3);
  ```

示例：

```java
        System.out.println("复杂查询");
        //复杂查询
        List<JpaTest> list1 = jpaTestDao.findAll(new Specification<JpaTest>() {
            /**
             * 相当于mybatis中逆向工程出来的example的条件查询,构造查询条件
             * @param root  【查询需要比较的对象属性】
             * @param criteriaQuery    自定义查询方式
             * @param criteriaBuilder  【查询条件】 查询的构造器，封装了很多查询条件
             * @return
             */
            @Override
            public Predicate toPredicate(Root<JpaTest> root, CriteriaQuery<?> criteriaQuery,
                                         CriteriaBuilder criteriaBuilder) {
                //获取查询需要比较的属性
                Path<Object> testId = root.get("testId");
                //构造查询方式
                    // 1.第一个参数：需要比较的属性（path对象）
                    // 2.第二个参数：该比较的属性的取值，这里可以实现动态的传参
                Predicate p1 = criteriaBuilder.equal(testId,"1");

                //多条件拼接
                Path<Object> info = root.get("info");
                //Predicate p2 = criteriaBuilder.equal(info,"1");

                //模糊查询需要注意
                    //模糊查询不同于equals，
                        // - equals在构造查询方式时直接传递path对象即可
                        // - like在构造查询方式时需要将path对象利用path.as（属性对应类型的字节码）转化才行
                Predicate p3 = criteriaBuilder.like(info.as(String.class),"1%");
                //拼接查询条件
                    //and() -- 与，是可变参数
                    //or() -- 或，也是可变参数
                Predicate p4 = criteriaBuilder.and(p1,p3);
                return p4;
            }
        });
        for(JpaTest j : list1){
            System.out.println(j);
        }
```

> 排序

排序主要用到Sort对象，实例化sort对象时需要传递两个参数

- 确定倒序还是正序：

  ```java
  Sort.Direction.ASC：正序
  Sort.Direction.ASC：正序
  ```

- 第二参数就是确定排序的根据属性，就是根据哪个字段排序的意思，是一个可变参数，可传递多个

  ```java
  Sort sort = new Sort(Sort.Direction.ASC,"id");
  ```

  示例：

```java
        System.out.println("复杂查询 + 排序");
        //复杂查询 + 排序
        //排序对象
            //需要传递两个参数才能实例化sort对象
            // - 第一个参数：正序还是倒序
                    // Sort.Direction.ASC：正序
                    // Sort.Direction.ASC：正序
            // - 第二个参数：排序的属性
                    //该参数也是一个可变参数，可以有多个排序字段
        Sort sort = new Sort(Sort.Direction.ASC,"id");
        List<JpaTest> list2 = jpaTestDao.findAll(sort);
        for(JpaTest j : list2){
            System.out.println(j);
        }
```

> 分页

分页主要用到Pageable对象，实例化该对象时需要指定两个参数：

- page：当前查询的页数，从0开始(即，你要查的是哪一页)
- size：每页的结果数

```java
Pageable pageable = new PageRequest(0,2);
```

示例：

```java
        //复杂查询 + 分页
        System.out.println("复杂查询 + 分页");
        //分页接口pageable实现类PageRequest（page，size）
            // 第一个参数：page：当前查询的页数，从0开始(即，你要查的是哪一页)
            // 第二个参数：size：每页的结果数
        Pageable pageable = new PageRequest(0,2);
        Page<JpaTest> page = jpaTestDao.findAll(pageable);
        //得到总条数
        System.out.println(page.getTotalElements());
        //得到数据集合的列表
        System.out.println(page.getContent());
        //得到总页数
        System.out.println(page.getTotalPages());
        //以上的分页和排序都可以用在findAll()方法中传递
```

##### 基于jpql的查询

**jpql与sql的区别**

> jpql与SQL的区别就是SQL是面向对象关系数据库，他操作的是数据表和数据列，而jpql操作的对象是实体对象和实体属性

除了select * 不能直接写从from开始之外，其他都可以直接写，类似sql

关于sql参数：

- 默认按照顺序拼接

- 也可以 ?0：表示第一个参数。?1：表示第二个参数，依次类推

```java
public interface JpaTestDao extends JpaRepository<JpaTest,Long>, JpaSpecificationExecutor<JpaTest> {

    /**
     *  JpaRepository<T,D>     --> 封装了基本的CRUD操作
     *        需要提供两个泛型   - T:操作的实体类类型
     *                         -  D：实体类主键的类型
     *
     *  JpaSpecificationExecutor<T>   ---> 封装了复杂查询（分页，排序等等）
     *         只需要提供一个泛型    - T:操作的实体类类型
     *
     */

    // jpql查询 --->可以用来进行复杂查询

    //支持jpql查询语句。
        //  基于注解的面向对象查询查询语句
        //  除了select * 不能直接写从from开始之外，其他都可以直接写，类似sql
        //  关于sql参数：
            // 默认按照顺序拼接
            // 也可以 ?0：表示第一个参数。?1：表示第二个参数，依次类推
    @Query(value = "from JpaTest where info = ?0")
    List<JpaTest> selectJpaTestByInfo(String info);

    @Query(value = "from JpaTest  where info = ?0 and id = ?1")
    List<JpaTest> selectJpaTestByInfoAndId(String info,int id);

    //更新（增删改）语句需要@Modifying注解配合，其他规则一致
    @Query(value = "update JpaTest set info = ?0 where id = ?1")
    @Modifying
    int updateJpaTest(String info,int id);

    //也可以编写原生sql。注解属性native = true即可
}
```



##### 基于方法名称规则查询

格式

```java
findBy + 查询比较属性 + and/or（查询方式）+查询比较属性 + and/or等等（查询方式）+查询比较属性 .....
```

示例：

```java
    //基于方法名称规则的查询
    //类似这种，以findBy 开头：
            // findBy + 查询比较属性 + and/or（查询方式）+查询比较属性 + and/or等等（查询方式）+查询比较属性 .....
    List<JpaTest> findByTestIdAndInfo(String testId,String info);
```



#### 2.更新（增删改）

默认提供的api的删除都是根据主键来执行的，先查询，在删除

```java
User user = new User();
user.setId(21);
user.setName("王二");
/**
 * 删除都是根据主键删除
 */
//删除单条,根据主键值
userRepository.delete(20);
//删除全部,先findALL查找出来,再一条一条删除,最后提交事务
userRepository.deleteAll();
//删除全部,一条sql
userRepository.deleteAllInBatch();
List<User> users = new ArrayList<>();
users.add(user);
//删除集合,一条一条删除
userRepository.delete(users);
//删除集合,一条sql,拼接or语句 如 id=1 or id=2
userRepository.deleteInBatch(users);
```

除了默认的api。我们还可以使用上面提到的jpql，这样我们就可以在dao层做到类似mybatis基于注解开发那样的实现了

```java
    //更新（增删改）语句需要@Modifying注解配合，其他规则一致
    @Query(value = "update JpaTest set info = ?0 where id = ?1")
    @Modifying
    int updateJpaTest(String info,int id);
```



> 刚打完球，比较累，就不说接下来的关系映射了，也就是几个注解的事情，最后一句，国足加油！

[github，jpa模块](https://github.com/Kevin091827/springboot_learning)