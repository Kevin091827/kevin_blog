

# mybatis复习（三）缓存机制

> 最近在复习框架的基础知识，利用国庆假期，想复习下mybatis，jpa，hibernate等orm框架，如果觉得写得还不错，有帮助到你，可以点波关注，我会继续更，一起努力！ 

## 一，一级缓存

> mybatis提供了一级缓存和二级缓存的缓存机制，能够很好的处理和维护缓存，以提高系统的性能，不用频繁的进行数据库交互

### 1.什么是一级缓存？

mybatis每一次与数据库的交互实际上是sqlSession对象和数据库的交互，具体工作原理分析可见：[工作原理](https://blog.csdn.net/weixin_41922289/article/details/101877972)，如果每一次的查询都需要sqlSession对象和数据库发起一次select，那么当访问量达到一定程度时，就会造成数据库承受的压力越来越大

针对这一性能问题，mybatis会在每一个sqlSession中维护一个简单的缓存，将每次查询的结构缓存起来，当下次查询的时候，会先查缓存，缓存中有，则直接返回出去，这样就减少了对数据库的访问次数

那么，**对于会话（Session）级别的数据缓存，我们称之为一级数据缓存，简称一级缓存。**

### 2.工作示意图

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100515503857.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)



### 3.深入源码



#### sqlSession组织

> 先看看sqlSession是如何组织一级缓存的？

转发自：https://img-blog.csdn.net/20141120100824184

![](https://img-blog.csdn.net/20141120100824184)

可以看到，缓存其实是由每一个Executor进行组织的，当mybatis和数据库进行交互时，会创建一个sqlSession对象，同时也会为这个sqlSession对象创建一个新的Executor执行器，对于缓存的操作就交由Executor执行

#### Executor缓存组织

> Executor中的对于缓存的操作在Executor接口也有相关定义

```java
  //创建CacheKey
  CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql);

  //判断是否缓存了
  boolean isCached(MappedStatement ms, CacheKey key);

  //清理Session缓存
  void clearLocalCache();
```

> 在执行器基类BaseExecutor中，是这样定义缓存的

```java
  //本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询(一级缓存)
  //本地缓存
  protected PerpetualCache localCache;
  //本地输出结果缓存
  protected PerpetualCache localOutputParameterCache;
```

>**Session**级别的一级缓存实际上就是使用**PerpetualCache**维护的，那么**PerpetualCache**是怎样实现的呢？PerpetualCache**实现原理其实很简单，其内部就是通过一个简单的**HashMap<k,v>** 来实现的，没有其他的任何限制

```java
public class PerpetualCache implements Cache {

  //每个永久缓存有一个ID来识别，相当于缓存key
  private String id;

  //内部就是一个HashMap,所有方法基本就是直接调用HashMap的方法,不支持多线程？
  private Map<Object, Object> cache = new HashMap<Object, Object>();
    
  //.....
}    
```

**在这里我还有个问题没搞懂，就是对于这个缓存的实现是hashMap的不解，hashmap是线程不安全的，那么在多线程的情况下，会出现问题吗？为什么要这样设计？**



 #### 操作缓存

> 查询方法

```java
  @Override
  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    //得到绑定sql
    BoundSql boundSql = ms.getBoundSql(parameter);
    //创建缓存Key
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
    //查询
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
 }
```

> 根据缓存key查缓存

1. 对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果；

2. 判断从Cache中根据特定的key值取的数据数据是否为空，即是否命中；
3. 如果命中，则直接将缓存结果返回；
4. 如果没命中：
1. 去数据库中查询数据，得到查询结果；
    2. 将key和查询到的结果分别作为key,value对存储到Cache中；
3. 将查询结果返回；
5. 结束

```java
      //先根据cachekey从localCache去查
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        //若查到localCache缓存，处理localOutputParameterCache
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        //从数据库查
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
```

> 针对这个工作流程，延伸几个问题

##### 怎样判断某两次查询是完全相同的查询？

1. statementId：对于**MyBatis**而言，你要使用它，必须需要一个***statementId***，它代表着你将执行什么样的**Sql**；
2. ***查询时要求的结果集中的结果范围*** 
3. sql语句

只有满足以上三点全部一致，才能确定是完成相同的查询



> 知道了怎么确定一次查询，就相当于确定这类查询的缓存key

##### CacheKey的确定

> statementId  + rowBounds  + 传递给JDBC的SQL  + 传递给JDBC的参数值





> 关于下一步具体如何操作缓存就不深究了

**SqlSession**中执行了任何一个**update**操作(**update()、delete()、insert()**) ，都会清空**PerpetualCache对象的数据，但是该对象可以继续使用**

```java
  public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    //先清局部缓存
    clearLocalCache();
    //在执行数据库的更新sql
    return doUpdate(ms, parameter);
  }
```

#### 一级缓存的生命周期

> 前文说了sqlsession创建后就会初始化一个PerpetualCache对象用于维护缓存，当执行查询操作时就会先查缓存，将查询结果放于缓存，当执行update就会先清除缓存，在执行sql，那么完整的生命周期是怎样的呢？

1. MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象，Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。

2. 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用；

3. 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用；

4. SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191005172752275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)



#### 总结

- mybatis的一级缓存是粗粒度的缓存，每次更新缓存都是先删除旧缓存，在插入新缓存，没有根据key去更新缓存内容
- 对于数据变化频率很大，并且需要高时效准确性的数据要求，我们使用SqlSession查询的时候，要控制好SqlSession的生存时间，SqlSession的生存时间越长，它其中缓存的数据有可能就越旧，从而造成和真实数据库的误差；同时对于这种情况，用户也可以手动地适时清空SqlSession中的缓存；
- 一级缓存默认开启
- 不可跨session
- 在未开启事务的情况之下，每次查询，spring都会关闭旧的sqlSession而创建新的sqlSession,因此此时的一级缓存是没有启作用的;
  在开启事务的情况之下，spring使用threadLocal获取当前资源绑定同一个sqlSession，因此此时一级缓存是有效的。

## 二，二级缓存

> 上文中说到一级缓存是sqlSession级别的缓存，那么接下来的二级缓存就是应用级别的缓存

#### 工作原理

当开一个会话时，一个SqlSession对象会使用一个Executor对象来完成会话操作，MyBatis的二级缓存机制的关键就是对这个Executor对象做文章。如果用户配置了"cacheEnabled=true"，那么MyBatis在为SqlSession对象创建Executor对象时，会对Executor对象加上一个装饰者：CachingExecutor，这时SqlSession使用CachingExecutor对象来完成操作请求。CachingExecutor对于查询请求，会先判断该查询请求在Application级别的二级缓存中是否有缓存结果，如果有查询结果，则直接返回缓存结果；如果缓存中没有，再交给真正的Executor对象来完成查询操作，之后CachingExecutor会将真正Executor返回的查询结果放置到缓存中，然后在返回给用户。

转发自https://img-blog.csdn.net/20141123125616381

![](https://img-blog.csdn.net/20141123125616381)

#### 缓存操作

> cacheExecutor组织

```java
/**
 * 二级缓存执行器
 */
public class CachingExecutor implements Executor {

  //baseExecutor
  private Executor delegate;
  //事务缓存，其实就是一个基于应用的
  private TransactionalCacheManager tcm = new TransactionalCacheManager();
```

> 事务缓存

```java
/**
 * 事务缓存管理器，被CachingExecutor使用
 *
 */
public class TransactionalCacheManager {

  //管理了许多TransactionalCache
  private Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();

  public void clear(Cache cache) {
    getTransactionalCache(cache).clear();
  }

  //得到某个TransactionalCache的值
  public Object getObject(Cache cache, CacheKey key) {
    return getTransactionalCache(cache).getObject(key);
  }
  //插入缓存
  public void putObject(Cache cache, CacheKey key, Object value) {
    getTransactionalCache(cache).putObject(key, value);
  }
  //....
    
}    
```

CachingExecutor中有个TransactionalCacheManager类来操作缓存，这个就是代码中的TCM，它的类中实例化了一个HashMap，存储Cache与TransactionalCache的映射关系。TransactionalCache实现了Cache接口，CachingExecutor会默认使用它包装初始生成的Cache，作用是如果事务提交，对缓存的操作才会生效，如果事务回滚或者不提交事务，则不对缓存产生影响。我们可以看到先通过TCM查询缓存，一直查询到PerpetualCache，如果查询不到，再通过Delegate查询，Delegate就是前文描述的一级缓存查询过程，这里不再赘述，然后再把查询到的结果写入二级缓存中，调用TCM.putObject方法。



> select操作

```java
  public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
      throws SQLException {
    Cache cache = ms.getCache();
    //默认情况下是没有开启缓存的(二级缓存).要开启二级缓存,你需要在你的 SQL 映射文件中添加一行: <cache/>
    //简单的说，就是先查CacheKey，查不到再委托给实际的执行器去查
    if (cache != null) {
      flushCacheIfRequired(ms);
      //判断是否开启二级缓存isUseCache
      if (ms.isUseCache() && resultHandler == null) {
        ensureNoOutParams(ms, parameterObject, boundSql);
        @SuppressWarnings("unchecked")//查询二级缓存
        //二级缓存使用的一个事务缓存，只要事务正常提交，缓存就会刷新        
        List<E> list = (List<E>) tcm.getObject(cache, key);
        if (list == null) {
          //查不到二级缓存在使用真正的executor，继续一级缓存的过程
          list = delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
          //查完放入缓存
          tcm.putObject(cache, key, list); // issue #578 and #116
        }
        return list;
      }
    }
    return delegate.<E> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
  }
```

> update



```java
  public int update(MappedStatement ms, Object parameterObject) throws SQLException {
	//先清除缓存完再update
    flushCacheIfRequired(ms);
    //真正执行器执行update  
    return delegate.update(ms, parameterObject);
  }  

  private void flushCacheIfRequired(MappedStatement ms) {
    Cache cache = ms.getCache();
    if (cache != null && ms.isFlushCacheRequired()) {      
      tcm.clear(cache);
    }
  }
```



#### 缓存淘汰策略



mybatis主要提供了一下几种策略用来应对当缓存满时的情况

- LRU缓存淘汰算法

  最近最少使用算法，即如果缓存中容量已经满了，会将缓存中最近做少被使用的缓存记录清除掉，然后添加新的记录；

- FIFO先进先出算法

  如果缓存中的容量已经满了，那么会将最先进入缓存中的数据清除掉；

- Scheduled指定时间间隔清空算法

  该算法会以指定的某一个时间间隔将**Cache**缓存中的数据清空；



 #### 总结

- 二级缓存默认不开启，需配置
- 二级缓存可以使用mybatis的默认实现，默认对于缓存的操作都是基于cache接口实现类
- 二级缓存也可以使用redis等相关第三方缓存服务器，但是不推荐，因为mybatis针对缓存更新都是先删除缓存在插入新缓存，而不能针对指定key，进行指定更新