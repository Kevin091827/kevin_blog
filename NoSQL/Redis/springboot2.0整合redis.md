# 一

# 二，springboot2.0整合redis手动配置

## 新建redis配置类RedisConfig

### 1.继承 CachingConfigurerSupport类完成对redis的基本配置
```java
public class CachingConfigurerSupport implements CachingConfigurer {
    public CachingConfigurerSupport() {
    }

    //自定义缓存管理器
    @Nullable
    public CacheManager cacheManager() {
        return null;
    }

    //通过自定义CacheResolver实现动态选择CacheManager
    @Nullable
    public CacheResolver cacheResolver() {
        return null;
    }

    //自定义key生成策略
    @Nullable
    public KeyGenerator keyGenerator() {
        return null;
    }

    //自定义缓存读写异常
    @Nullable
    public CacheErrorHandler errorHandler() {
        return null;
    }
}
```

#### CacheErrorHandler

CacheErrorHandler是一个缓存异常处理接口，定义了缓存读写异常的方法
```java
public interface CacheErrorHandler {
    void handleCacheGetError(RuntimeException var1, Cache var2, Object var3);

    void handleCachePutError(RuntimeException var1, Cache var2, Object var3, @Nullable Object var4);

    void handleCacheEvictError(RuntimeException var1, Cache var2, Object var3);

    void handleCacheClearError(RuntimeException var1, Cache var2);
}
```
缓存仅仅是为了业务更快地查询而存在的，如果因为缓存操作失败导致正常的业务流程失败，有点得不偿失了。因此需要开发者自定义CacheErrorHandler处理缓存读写的异常。

redis缓存读写异常的默认实现是SimpleCacheErrorHandler

通过查看源码可以知道，SimpleCacheErrorHandler对每种错误都是简单的抛出一个Exception
```java
public class SimpleCacheErrorHandler implements CacheErrorHandler {
    public SimpleCacheErrorHandler() {
    }

    public void handleCacheGetError(RuntimeException exception, Cache cache, Object key) {
        throw exception;
    }

    public void handleCachePutError(RuntimeException exception, Cache cache, Object key, @Nullable Object value) {
        throw exception;
    }

    public void handleCacheEvictError(RuntimeException exception, Cache cache, Object key) {
        throw exception;
    }

    public void handleCacheClearError(RuntimeException exception, Cache cache) {
        throw exception;
    }
}
```

言归正传，自定义缓存异常只需要重写这四个方法即可
```java
    /**
     * 处理缓存读写异常（缓存自定义异常）
     * @return
     */
    @Override
    public CacheErrorHandler errorHandler() {

       CacheErrorHandler cacheErrorHandler = new CacheErrorHandler() {

           @Override
           public void handleCacheGetError(RuntimeException e, Cache cache, Object key) {
                log.error("缓存 查找 失败，失败原因："+e.getMessage()+"缓存key："+key);
           }

           @Override
           public void handleCachePutError(RuntimeException e, Cache cache, Object key, Object o1) {
               log.error("缓存 更新 失败，失败原因："+e.getMessage()+"缓存key："+key);
           }

           @Override
           public void handleCacheEvictError(RuntimeException e, Cache cache, Object key) {
               log.error("缓存 删除 失败，失败原因："+e.getMessage()+"缓存key："+key);
           }

           @Override
           public void handleCacheClearError(RuntimeException e, Cache cache) {
               log.error("缓存 清除 失败，失败原因："+e.getMessage());
           }
       };
       return cacheErrorHandler;
    }
```


