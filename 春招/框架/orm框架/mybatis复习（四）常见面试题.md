# mybatis复习（四）常见面试题



### 1.mybatis是什么？

1. mybatis是一个半ORM框架，内部实现是基于JDBC，但是他很好的封装了JDBC，开发时只需要关注SQL本身，不需要花费精力去加载驱动，创建连接，创建statement等等
2. mybatis可以使用XML或者注解来完成配置和映射
3. 通过xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过java对象和 statement中sql的动态参数进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射为java对象并返回
4. 优点：
   	1. 基于SQL编程，灵活，把SQL写在XML配置文件中可以使sql和程序代码解耦，统一管理，还可以支持编写动态sql
    	2. 消除了JDBC的大量重复代码，不需要手动开关连接
    	3. 提供映射标签，支持对象与数据库的ORM字段关系映射
5. 缺点：
   1. SQL语句的编写工作量较大，尤其当字段多、关联表多时，对开发人员编写SQL语句的功底有一定要求。
   2. SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库。

### 2.mybatis和hibernate的区别？

1. Mybatis和hibernate不同，它不完全是一个ORM框架，因为MyBatis需要程序员自己编写Sql语句。
2. Mybatis直接编写原生态sql，可以严格控制sql执行性能，灵活度高，非常适合对关系数据模型要求不高的软件开发，因为这类软件需求变化频繁，一但需求变化要求迅速输出成果。但是灵活的前提是mybatis无法做到数据库无关性，如果需要实现支持多种数据库的软件，则需要自定义多套sql映射文件，工作量大。 
3. Hibernate对象/关系映射能力强，数据库无关性好，对于关系模型要求高的软件，如果用hibernate开发可以节省很多代码，提高效率

### 3.#{}和${}的区别是什么？

- #{}是预编译处理，${}是字符串替换
- mybatis在处理#{}时会将sql中的#{}替换为?，然后利用PreparedStatement的set方法来设值，有效的防止了sql注入，提高系统安全性
- Mybatis在处理${}时，就是把${}替换成变量的值

### 4.当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

1. 通过在查询的sql语句中定义字段名的别名，让字段名的别名和实体类的属性名一致。

   ```xml
        <select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”>
           select order_id id, order_no orderno ,order_price price form orders where order_id=#{id};
        </select>
   ```

2. 通过<resultMap>来映射字段名和实体类属性名的一一对应的关系。

   ```xml
   <select id="getOrder" parameterType="int" resultMap="orderresultmap">
           select * from orders where order_id=#{id}
       </select>
    
      <resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
           <!–用id属性来映射主键字段–>
           <id property=”id” column=”order_id”>
    
           <!–用result属性来映射非主键字段，property为实体类属性名，column为数据表中的属性–>
           <result property = “orderno” column =”order_no”/>
           <result property=”price” column=”order_price” />
       </reslutMap>
   ```

### 5.模糊查询like语句如何编写？

1. 在Java代码中添加sql通配符。

   ```java
   	string wildcardname = “%smi%”;
       list<name> names = mapper.selectlike(wildcardname);
   ```

   ```xml
   	<select id=”selectlike”>
        	select * from foo where bar like #{value}
       </select>
   ```

2. 在sql语句中拼接通配符，会引起sql注入

   ```java
   	string wildcardname = “smi”;
       list<name> names = mapper.selectlike(wildcardname);
   ```

   ```xml
   	<select id=”selectlike”>
            select * from foo where bar like "%"#{value}"%"
       </select>
   ```

### 6.通常一个Xml映射文件，都会写一个Dao接口与之对应，请问，这个Dao接口的工作原理是什么？Dao接口里的方法，参数不同时，方法能重载吗？

Dao接口就是Mapper接口，mapper.xml中的namespace就是mapper接口的全限定类名，接口的方法名就是<mapper>标签中statement的id,接口方法内的参数，就是传递给sql的参数。

Mapper接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为key值，可唯一定位一个MapperStatement。在Mybatis中，每一个<select>、<insert>、<update>、<delete>标签，都会被解析为一个MapperStatement对象。

> 举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到namespace为com.mybatis3.mappers.StudentDao下面 id 为 findStudentById 的 MapperStatement。

Mapper接口里的方法，是不能重载的，因为是使用 全限名+方法名 的保存和寻找策略。**Mapper 接口的工作原理是JDK动态代理，Mybatis运行时会使用JDK动态代理为Mapper接口生成代理对象proxy，代理对象会拦截接口方法，转而执行MapperStatement所代表的sql，然后将sql执行结果返回。**

​	

### 7.Mybatis动态sql有什么用？执行原理？有哪些动态sql？

Mybatis动态sql可以在Xml映射文件内，以标签的形式编写动态sql，执行原理是根据表达式的值 完成逻辑判断并动态拼接sql的功能。

Mybatis提供了9种动态sql标签：trim | where | set | foreach | if | choose | when | otherwise | bind。

 