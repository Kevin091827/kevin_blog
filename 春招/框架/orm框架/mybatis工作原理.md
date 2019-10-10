# mybatis工作原理

> 最近在复习框架的基础知识，利用国庆假期，想复习下mybatis，jpa，hibernate等orm框架，如果觉得写得还不错，有帮助到你，可以点波关注，我会继续更，一起努力！

## 1.工作流程

### 核心配置文件读取

配置文件的读取时mybatis工作的第一步，也可以当初是mybatis的初始化

在这一步中，mybatis做了些什么？下边将会解析这几部分

- mybatis基于xml配置文件创建Configuration对象的过程
- 基于configuration对象创建sqlSessionFactory的过程
- SQLSessionFactory创建sqlSession的过程

#### 1，mybatis基于xml配置文件创建Configuration对象的过程

**mybatis-config.xml**


```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<!-- 属性 -->
	<properties resource="db.properties"></properties>
	<!-- 别名 -->
	<typeAliases>
		<typeAlias type="com.example.mybatis.entity.User" value="u"></typeAlias>
		<package name="com.example.mybatis.enity" />
	</typeAliases>
 	<!-- 设置 -->
	<settings>
		<!-- 全局开启或关闭配置文件中的所有映射器已经配置的任何缓存,默认是开启 -->
		<setting name="cacheEnabled" value="true" />
		<!-- 延迟加载，默认关闭 -->
		<setting name="lazyLoadingEnabled" value="false" />
		<!-- 是否允许单一语句返回多结果集，默认开启 -->
		<setting name="multipleResultSetsEnabled" value="true" />
		<!-- 使用列标签代替列名，默认开启 -->
		<setting name="useColumnLabel" value="true" />
		<!-- 允许 JDBC 支持自动生成主键，默认关闭 -->
		<setting name="useGeneratedKeys" value="false" />
		<!-- 指定 MyBatis 应如何自动映射列到字段或属性，默认只会自动映射没有定义嵌套结果集映射的结果集 -->
		<setting name="autoMappingBehavior" value="PARTIAL" />
		<setting name="autoMappingUnknownColumnBehavior"
			value="WARNING" />
		<!-- 默认执行器simple -->	
		<setting name="defaultExecutorType" value="SIMPLE" />
		<!-- 默认超时时间 -->
		<setting name="defaultStatementTimeout" value="25" />
		<setting name="defaultFetchSize" value="100" />
		<setting name="safeRowBoundsEnabled" value="false" />
		<!-- 是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。默认不开启-->
		<setting name="mapUnderscoreToCamelCase" value="false" />
		<!-- 默认开启一级缓存，不在sqlsession间共享 -->
		<setting name="localCacheScope" value="SESSION" />
		<setting name="jdbcTypeForNull" value="OTHER" />
		<setting name="lazyLoadTriggerMethods"
			value="equals,clone,hashCode,toString" />
	</settings>
	<!-- 环境配置 -->
	<environments default="development">
		<!-- id为环境的唯一标识 -->
		<environment id="development">
			<!-- 事物管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据源 -->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
	</environments>
 
	<!-- 多数据库支持 -->
	<databaseIdProvider type="DB_VENDOR">
		<property name="MySQL" value="mysql"></property>
		<property name="Oracle" value="oracle"></property>
		<property name="SQL Server" value="sql server"></property>
	</databaseIdProvider>
 
	<!-- sql映射文件注册 -->
	<mappers>
		<mapper resource="com/example/mybatis/dao/UserMapper.xml"></mapper>
		<mapper class="com.example.mybatis.dao.UserMapperAnnotaion"></mapper>
	</mappers>
</configuration>
```
Configuration可以把其看成一个entity实体，主要包括下列属性：

- properties（属性）
- settings（设置）
- typeAliases（类型别名）
- typeHandlers（类型处理器）
- objectFactory（对象工厂）
- plugins（插件）
- environments（环境配置）
	- environment（环境变量）
	- transactionManager（事务管理器）
	- dataSource（数据源）

- databaseIdProvider（数据库厂商标识）
- mappers（映射器）

所以说，mybatis初始化就是相当于conguration对象的初始化过程

#### 2，基于configuration对象创建sqlSessionFactory的过程

那么，这个conguration的初始化过程是怎样的呢？

先看下边这段java代码

```java

//配置文件
String resource = "mybatis-config.xml";
//获取配置文件输入流
InputStream inputStream = Resources.getResourceAsStream(resource);
//获取sqlSessionFactory
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

```

大体流程：

![1569939999990](C:\Users\12642\AppData\Roaming\Typora\typora-user-images\1569939999990.png)

sqlSessionFactory构建大体流程：

1. 将mybatis.xml写入输入流inputStream

2. 基于输入流构建sqlSessionFactory
```java
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```
3. 委托XMLConfigBuilder来解析xml文件     
```java
XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);*       
return build(parser.parse());
```
4. 装配到conguration对象
5. 构建sqlSessionFactory
```java
public SqlSessionFactory build(Configuration config) {
return new DefaultSqlSessionFactory(config);
}
```

**具体看看关于mybatis配置文件解析过程**

1. xml解析是交给XMLConfigBuilder来进行的
```java
      //将输入流交给XMLConfigBuilder
      XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
      //paser()进行xml解析
      return build(parser.parse());
```
2. xml解析，获取conguration配置
```java
  public Configuration parse() {
    //如果已经解析过了，报错
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
//  <?xml version="1.0" encoding="UTF-8" ?> 
//  <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
//  "http://mybatis.org/dtd/mybatis-3-config.dtd"> 
//  <configuration> 
//  <environments default="development"> 
//  <environment id="development"> 
//  <transactionManager type="JDBC"/> 
//  <dataSource type="POOLED"> 
//  <property name="driver" value="${driver}"/> 
//  <property name="url" value="${url}"/> 
//  <property name="username" value="${username}"/> 
//  <property name="password" value="${password}"/> 
//  </dataSource> 
//  </environment> 
//  </environments>
//  <mappers> 
//  <mapper resource="org/mybatis/example/BlogMapper.xml"/> 
//  </mappers> 
//  </configuration>
    
    //根节点是configuration
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```
3. 解析配置，并初始化conguration
```java
  //解析配置
  private void parseConfiguration(XNode root) {
    try {
      //分步骤解析
      //issue #117 read properties first
      //1.properties
      propertiesElement(root.evalNode("properties"));
      //2.类型别名
      typeAliasesElement(root.evalNode("typeAliases"));
      //3.插件
      pluginElement(root.evalNode("plugins"));
      //4.对象工厂
      objectFactoryElement(root.evalNode("objectFactory"));
      //5.对象包装工厂
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      //6.设置
      settingsElement(root.evalNode("settings"));
      // read it after objectFactory and objectWrapperFactory issue #631
      //7.环境
      environmentsElement(root.evalNode("environments"));
      //8.databaseIdProvider
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      //9.类型处理器
      typeHandlerElement(root.evalNode("typeHandlers"));
      //10.映射器
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```
初始化conguration过程，取其中一个说明
```java
  private void propertiesElement(XNode context) throws Exception {
	 //...省略一些代码
      configuration.setVariables(defaults);
    }
  }
```

#### 3，SQLSessionFactory创建sqlSession的过程

sqlSession是与数据库进行交互的，可以从sqlSessionFactory中调用openSession获取一个sqlSession
```java
  public SqlSession openSession() {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
```
配置事务和执行器，默认的执行器是simpleExecutor
```java
  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      //通过事务工厂来产生一个事务
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      //生成一个执行器(事务包含在执行器里)
      final Executor executor = configuration.newExecutor(tx, execType);
      //然后产生一个DefaultSqlSession
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      //如果打开事务出错，则关闭它
      closeTransaction(tx); // may have fetched a connection so lets call close()
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      //最后清空错误上下文
      ErrorContext.instance().reset();
    }
  }
```

###  Executor执行器执行sql

#### 1，jdbc复习 

连接工厂
```java
    private static String JDBC_URL = "";
    private static String USERNAME = "";
    private static String PASSWORD = "";
    private static String DRIVER = "com.mysql.jdbc.Driver";

    /**
     * 获取数据库连接
     * @return
     */
    public static Connection getConnection(){
        Connection connection = null;
        try {
            //将加载的驱动类注册给DriverManager
            Class.forName(DRIVER);
            //获取数据库连接
            connection = DriverManager.getConnection(JDBC_URL,USERNAME,PASSWORD);
            return connection;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 关闭数据库连接
     * @param connection
     */
    public static void close(Connection connection){
        try {
            if(connection != null) {
                connection.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
增删改查工具类
```java

    /**
     * 增删改
     * @param sql
     * @param params
     * @param connection
     * @return
     * @throws Exception
     */
    public static int update(String sql, Object[] params, Connection connection) throws Exception {
        //连接为null，则抛出异常
        if(connection == null){
            throw new Exception("connection is null");
        }
        //操作数据库
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //封装参数
        if(null != params){
            for (int i = 0; i < params.length; i++) {
                preparedStatement.setObject(i + 1, params[i]);
            }
        }
        //执行sql
        int i = preparedStatement.executeUpdate();
        return i;
    }

    /**
     * 查询
     * @param sql
     * @param params
     * @param connection
     * @return
     * @throws Exception
     */
    public static ResultSet select(String sql, Object[] params, Connection connection) throws Exception{
        //连接为null，则抛出异常
        if(connection == null){
            throw new Exception("connection is null");
        }
        //操作数据库
        PreparedStatement preparedStatement = connection.prepareStatement(sql);
        //封装参数
        if(null != params){
            for (int i = 0; i < params.length; i++) {
                preparedStatement.setObject(i + 1, params[i]);
            }
        }
        ResultSet resultSet = preparedStatement.executeQuery();
        return resultSet;
    }
```
##### jdbc知识点总结

**Statement和PreparedStatement**

1. Statement和PreparedStatement都是用来执SQL语句
2. Statement是使用字符串拼接的方式，容易发生sql注入
3. PreparedStatement使用占位符参数设置，不容易发生sql注入
4. PreparedStatement其具有预编译机制，性能比statement更快。


#### mybatis中的executor执行sql

流程图：

> 转发：https://blog.csdn.net/qq_22423635/article/details/88311996

![å¨è¿éæå¥å¾çæè¿°](https://static.oschina.net/uploads/space/2016/0505/181036_SgDu_2727738.jpg)

执行sql

```java
  //update
  @Override
  public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      //新建一个StatementHandler
      //这里看到ResultHandler传入的是null
      StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
      //准备语句
      stmt = prepareStatement(handler, ms.getStatementLog());
      //StatementHandler.update
      return handler.update(stmt);
    } finally {
      closeStatement(stmt);
    }
  }

  //select
  @Override
  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {
    Statement stmt = null;
    try {
      Configuration configuration = ms.getConfiguration();
      //新建一个StatementHandler
      //这里看到ResultHandler传入了
      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
      //准备语句
      stmt = prepareStatement(handler, ms.getStatementLog());
      //StatementHandler.query
      return handler.<E>query(stmt, resultHandler);
    } finally {
      closeStatement(stmt);
    }
  }
```

