# mybatis连接池和事务管理

> 最近在复习框架的基础知识，利用国庆假期，想复习下mybatis，jpa，hibernate等orm框架，如果觉得写得还不错，有帮助到你，可以点波关注，我会继续更，一起努力！
## 一,连接池

为什么要使用连接池？

创建一个java.sql.Connection对象的代价是如此巨大，是因为创建一个Connection对象的过程，在底层就相当于和数据库建立的通信连接，在建立通信连接的过程，消耗了这么多的时间，而往往我们建立连接后（即创建Connection对象后），就执行一个简单的SQL语句，然后就要抛弃掉，这是一个非常大的资源浪费！

所以对于需要频繁地跟数据库交互的应用程序，可以在创建了Connection对象，并操作完数据库后，可以不释放掉资源，而是将它放到内存中，当下次需要操作数据库时，可以直接从内存中取出Connection对象，不需要再创建了，这样就极大地节省了创建Connection对象的资源消耗
 
手写一个简单的数据库连接池：

```java
public class ConnectionPool {
    
    private static String DRIVER = "com.mysql.jdbc.Driver";
    private static String URL="";
    private static String USERNAME="";
    private static String PASSWORD="";
    
    static {
        try {
            Class.forName(DRIVER);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }


    private LinkedList<Connection> poollist = new LinkedList<Connection>();

    //向连接池中放入10个连接对象，封装到一个集合中
    public ConnectionPool() throws Exception {
        for(int i=0;i<10;i++) {
            poollist.addLast(this.creatConnection());
        }
    }

    //创造连接对象
    public Connection creatConnection() throws Exception {
        return DriverManager.getConnection(URL, USERNAME, PASSWORD);
    }

    //从连接池中取出一个连接对象
    public Connection getConnection() {
        return poollist.removeFirst();
    }

    //关闭连接，释放资源，将连接对象放回连接池中
    public void close(Connection conn) {
        poollist.addLast(conn);
    }
}
```





mybatis的数据源分为：

- 不使用连接池的UnpooledDataSource
- 使用连接池的PooledDataSource
- 使用JNDI实现的数据源的JndiDataSource



### 1.不使用连接池的UnpooledDataSource

将数据库驱动，用户名密码等属性封装到properties中，底层还是使用熟悉的jdbc获取数据库连接

```java
  /**
   * 底层还是熟悉的jdbc的数据库连接
   * @param properties
   * @return
   * @throws SQLException
   */
  private Connection doGetConnection(Properties properties) throws SQLException {
    initializeDriver();
    //属性的前缀是以“driver.”开 头的,它 是 通 过 DriverManager.getConnection(url,driverProperties)方法传递给数据库驱动
    Connection connection = DriverManager.getConnection(url, properties);
    configureConnection(connection);
    return connection;
  }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191002120431391.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 2.使用连接池的PooledDataSource

**mybatis中PooledDataSource连接池的设计**

PooledDataSource将connection包装成PooledConnection对象放到PoolState连接容器中，这个连接容器中有两种连接

```java
  protected PooledDataSource dataSource;

  //空闲的连接
  protected final List<PooledConnection> idleConnections = new ArrayList<PooledConnection>();
  //活动的连接
  protected final List<PooledConnection> activeConnections = new ArrayList<PooledConnection>();
```



PooledDataSource获取连接会优先从空闲连接list中取，当前正在被使用的连接会放入活动连接的list中

![img](https://img-blog.csdn.net/20140710231403937?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHVhbmxvdWlz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

获取连接的过程：

```java
 public Connection getConnection() throws SQLException {
    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
  }
 
  public Connection getConnection(String username, String password) throws SQLException {
    return popConnection(username, password).getProxyConnection();
  }
```

接着从连接池中获取

popConnection()主要获取连接的逻辑：

- while循环不断尝试获取
- 先判断空闲连接池中是否还有空闲连接

  - 有空闲连接：返回连接池第一个连接
  - 没有空闲连接：查看活动连接池信息
- 判断活动连接池中是否达到了最大连接数

  - 没有达到：直接new 一个新连接
  - 达到：不能再new新连接，获取连接池中最老的了、第一个连接，删除该连接后new 一个新连接


```java
    //最外面是while死循环，如果一直拿不到connection，则不断尝试
    while (conn == null) {
      synchronized (state) {
        //先看看是否还有空闲连接
        if (!state.idleConnections.isEmpty()) {
          //如果有空闲的连接的话，删除空闲列表里第一个，返回，拿到这个连接
          conn = state.idleConnections.remove(0);
          if (log.isDebugEnabled()) {
            log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
          }
        } else {
        	//如果没有空闲的连接，则判断活动连接list还有没有空间
          if (state.activeConnections.size() < poolMaximumActiveConnections) {
        	  //如果activeConnections太少,那就new一个PooledConnection
            conn = new PooledConnection(dataSource.getConnection(), this);
            if (log.isDebugEnabled()) {
              log.debug("Created connection " + conn.getRealHashCode() + ".");
            }
          } else {
        	  //如果activeConnections已经很多了，那不能再new了，此时取得activeConnections列表的第一个（最老的）
            PooledConnection oldestActiveConnection = state.activeConnections.get(0);
            //获取该连接的checkout时间
            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
            if (longestCheckoutTime > poolMaximumCheckoutTime) {
            	//如果checkout时间过长，则这个connection标记为overdue（过期）
              state.claimedOverdueConnectionCount++;
              state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
              state.accumulatedCheckoutTime += longestCheckoutTime;
              state.activeConnections.remove(oldestActiveConnection);
              if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                oldestActiveConnection.getRealConnection().rollback();
              }
              //删掉最老的连接，然后再new一个新连接
              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
              oldestActiveConnection.invalidate();
```

归还连接pushConnection

```java
    synchronized (state) {
      //先从activeConnections中删除此connection
      state.activeConnections.remove(conn);
      //判断连接是否失效
      if (conn.isValid()) {
        //如果空闲连接池还有空间
        if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {
      	  //空闲的连接太少，
          state.accumulatedCheckoutTime += conn.getCheckoutTime();
          if (!conn.getRealConnection().getAutoCommit()) {
            conn.getRealConnection().rollback();
          }
          //new一个新的Connection，加入到idle列表
          PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
          state.idleConnections.add(newConn);
          newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
          newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());
          conn.invalidate();
          if (log.isDebugEnabled()) {
            log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");
          }
          //通知其他线程可以来抢connection了
          state.notifyAll();
        } else {
        	//否则，即空闲的连接已经足够了
          state.accumulatedCheckoutTime += conn.getCheckoutTime();
          if (!conn.getRealConnection().getAutoCommit()) {
            conn.getRealConnection().rollback();
          }
          //那就将connection关闭就可以了
          conn.getRealConnection().close();
          if (log.isDebugEnabled()) {
            log.debug("Closed connection " + conn.getRealHashCode() + ".");
          }
          conn.invalidate();
        }
```



关于JNDI连接池就不说了，我们一般使用阿里巴巴的druid或者HikariCP 



## 二,mybatis事务管理

### 1，数据库事务复习



#### 1.ACID基本性质

- 原子性
	
	一个事务中的所有操作，要么全部完成，要么全部不完成
	
- 一致性

	事务必须使数据库从一个一致性状态变换到另一个一致性状态，也就是说一个事务执行之前和执行之后都必须处于一致性状态。
	
- 隔离性

	是当多个用户并发访问数据库时，比如操作同一张表时，数据库为每一个用户开启的事务，不能被其他事务的操作所干扰，多个并发事务之间要相互隔离。

	- 读未提交
	- 读已提交
	- 可重复读（innoDB默认隔离级别）
	- 不可重复读

	![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100214451485.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

- 持久性

	是指一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便是在数据库系统遇到故障的情况下也不会丢失提交事务的操作。



#### 2.java中的基本事务模型代码

jdbc中事务基本编写模板，还可以设定savepoint保存点，回滚事务时回滚到保存点

```java
try {  
    conn.setAutoCommit(false);  //将自动提交设置为false           
    ps.executeUpdate("修改SQL"); //执行修改操作  
    ps.executeQuery("查询SQL");  //执行查询操作                 
    conn.commit();      //当两个操作成功后手动提交            
} catch (Exception e) {  
    conn.rollback();    //一旦其中一个操作出错都将回滚，使两个操作都不成功  
    e.printStackTrace();  
} 
```

#### 2，mybatis事务管理

mybatis事务模块分层：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100215012290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

可以看到Transaction定义了事务的创建，提交，回滚等操作

```java
public interface Transaction {
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
}
```

也可以看到MyBatis的事务管理分为两种形式：

- 一、使用JDBC的事务管理机制：即利用java.sql.Connection对象完成对事务的提交（commit()）、回滚（rollback()）、关闭（close()）等

- 二、使用MANAGED的事务管理机制：这种机制MyBatis自身不会去实现事务管理，而是让程序的容器如（JBOSS，Weblogic）来实现对事务的管理

重点分析下JDBC的事务管理机制在mybatis中的实现

    JdbcTransaction直接使用JDBC的提交和回滚事务管理机制 。它依赖与从dataSource中取得的连接connection 来管理transaction 的作用域，connection对象的获取被延迟到调用getConnection()方法。如果autocommit设置为on，开启状态的话，它会忽略commit和rollback。
    
    直观地讲，就是JdbcTransaction是使用的java.sql.Connection 上的commit和rollback功能
我们在spring整合mybatis时，mybatis配置文件中的一般这样使用事务

> 转发自https://blog.csdn.net/luanlouis/article/details/37992171

![img](https://img-blog.csdn.net/20140720150704081)

那么，mybatis事务管理的流程是：

1. 在初始化mybatis配置时，加载相关配置到conguration对象

2. 根据配置创建事务工厂

   ```java
   TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"))
   ```
   
3. 从事务工厂中获取事务

   ```java
     /**
      * 
      * @param ds  数据源
      * @param level Desired isolation level事务隔离级别
      * @param autoCommit Desired autocommit是否自动提交
      * @return
      */
     @Override
     public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
       return new JdbcTransaction(ds, level, autoCommit);
     }
   ```

4. 将事务封装在执行器中

    ```java
      protected BaseExecutor(Configuration configuration, Transaction transaction) {
        this.transaction = transaction;
        this.deferredLoads = new ConcurrentLinkedQueue<DeferredLoad>();
        this.localCache = new PerpetualCache("LocalCache");
        this.localOutputParameterCache = new PerpetualCache("LocalOutputParameterCache");
        this.closed = false;
        this.configuration = configuration;
        this.wrapper = this;
      }
    ```
    
5. 在具体的jdbcTrancation中底层还是jdbc的事务操作

    ```java
      @Override
      public void commit() throws SQLException {
        if (connection != null && !connection.getAutoCommit()) {
          if (log.isDebugEnabled()) {
            log.debug("Committing JDBC Connection [" + connection + "]");
          }
          //熟悉的commit  
          connection.commit();
        }
      }
    
      @Override
      public void rollback() throws SQLException {
        if (connection != null && !connection.getAutoCommit()) {
          if (log.isDebugEnabled()) {
            log.debug("Rolling back JDBC Connection [" + connection + "]");
          }
          //熟悉的rollback  
          connection.rollback();
        }
      }
    ```



6. 上面仅仅提到对jdbc的事务封装，那如何实现真正的事务管理呢

之前我们有说过mybatis的执行过程

![[å¤é¾å¾çè½¬å­å¤±è´¥,æºç«å¯è½æé²çé¾æºå¶,å»ºè®®å°å¾çä¿å­ä¸æ¥ç´æ¥ä¸ä¼ (img-RW5zE0vK-1569948080819)(C:\Users\12642\AppData\Roaming\Typora\typora-user-images\1569939999990.png)]](https://img-blog.csdnimg.cn/20191002113818181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

sqlSession是与数据库交互的工具，可以理解为一组数据库增删改查的一组操作，事务也是数据库的一组操作，那么mybatis对事务的管理就发生在一个一个的session中

- 从sqlSessionFactory中获取sqlSession	

  ```java
  public SqlSession openSession() {
    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
  ```

  在进入openSessionFromDataSource

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

  













