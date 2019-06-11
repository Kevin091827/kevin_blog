>在上文中我们知道了redisTemplate是springboot中操作redis的核心，今天来进一步了解了解其内部实现


**继承关系**
```java

public class RedisTemplate<K, V> extends RedisAccessor implements RedisOperations<K, V>, BeanClassLoaderAware {

    //````````````
}
```
分析的顺序会按照从父到子，从接口到实现的顺序

## 一，RedisOperations

RedisOperations接口定义了redisTemplate操作redis的一些api方法，主要提供了一些对Redis键，事务，运行脚本等命令的支持，不负责数据的读写,具体实现类有两个,
```java
StringRedisTemplate

RedisTemplate
```
根据泛型类型的参数有不同的实现

## 二，RedisAccessor
RedisAccessor主要就是对连接工厂的配置，实现了InitializingBean接口，在初始化bean时会断言是否已经配置连接工厂

```java
public class RedisAccessor implements InitializingBean {
    protected final Log logger = LogFactory.getLog(this.getClass());
    @Nullable
    private RedisConnectionFactory connectionFactory;

    public RedisAccessor() {
    }

    public void afterPropertiesSet() {
        Assert.state(this.getConnectionFactory() != null, "RedisConnectionFactory is required");
    }

    @Nullable
    public RedisConnectionFactory getConnectionFactory() {
        return this.connectionFactory;
    }

    public RedisConnectionFactory getRequiredConnectionFactory() {
        RedisConnectionFactory connectionFactory = this.getConnectionFactory();
        if (connectionFactory == null) {
            throw new IllegalStateException("RedisConnectionFactory is required");
        } else {
            return connectionFactory;
        }
    }

    public void setConnectionFactory(RedisConnectionFactory connectionFactory) {
        this.connectionFactory = connectionFactory;
    }
}
```

## 三，RedisTemplate

### 1.序列化配置
RedisTemplate主要关注初始化bean时做了什么

```java
    public void afterPropertiesSet() {
        super.afterPropertiesSet();
        boolean defaultUsed = false;
        if (this.defaultSerializer == null) {
            this.defaultSerializer = new JdkSerializationRedisSerializer(this.classLoader != null ? this.classLoader : this.getClass().getClassLoader());
        }

        if (this.enableDefaultSerializer) {
            if (this.keySerializer == null) {
                this.keySerializer = this.defaultSerializer;
                defaultUsed = true;
            }

            if (this.valueSerializer == null) {
                this.valueSerializer = this.defaultSerializer;
                defaultUsed = true;
            }

            if (this.hashKeySerializer == null) {
                this.hashKeySerializer = this.defaultSerializer;
                defaultUsed = true;
            }

            if (this.hashValueSerializer == null) {
                this.hashValueSerializer = this.defaultSerializer;
                defaultUsed = true;
            }
        }

        if (this.enableDefaultSerializer && defaultUsed) {
            Assert.notNull(this.defaultSerializer, "default serializer null and not all serializers initialized");
        }

        if (this.scriptExecutor == null) {
            this.scriptExecutor = new DefaultScriptExecutor(this);
        }

        this.initialized = true;
    }
```
可见初始化bean时，如果我们没有配置序列化，默认是jdk序列化


### 2.连接获取和关闭

```java
    @Nullable
    public <T> T execute(RedisCallback<T> action, boolean exposeConnection, boolean pipeline) {
        Assert.isTrue(this.initialized, "template not initialized; call afterPropertiesSet() before using it");
        Assert.notNull(action, "Callback object must not be null");
        RedisConnectionFactory factory = this.getRequiredConnectionFactory();
        RedisConnection conn = null;

        //获取连接从连接工厂中
        Object var11;
        try {
            if (this.enableTransactionSupport) {
                conn = RedisConnectionUtils.bindConnection(factory, this.enableTransactionSupport);
            } else {
                conn = RedisConnectionUtils.getConnection(factory);
            }

            boolean existingConnection = TransactionSynchronizationManager.hasResource(factory);
            RedisConnection connToUse = this.preProcessConnection(conn, existingConnection);
            boolean pipelineStatus = connToUse.isPipelined();
            if (pipeline && !pipelineStatus) {
                connToUse.openPipeline();
            }

            RedisConnection connToExpose = exposeConnection ? connToUse : this.createRedisConnectionProxy(connToUse);
            //封装连接，传入回调函数执行真正的redis操作
            T result = action.doInRedis(connToExpose);
            if (pipeline && !pipelineStatus) {
                connToUse.closePipeline();
            }

            var11 = this.postProcessResult(result, connToUse, existingConnection);
        } finally {
            //释放连接
            RedisConnectionUtils.releaseConnection(conn, factory);
        }

        return var11;
    }
```

以delete 为例子，执行execute传入连接执行真正的删除操作
```java
    public Boolean delete(K key) {
        byte[] rawKey = this.rawKey(key);
        Long result = (Long)this.execute((connection) -> {
            return connection.del(new byte[][]{rawKey});
        }, true);
        return result != null && result.intValue() == 1;
    }
```

