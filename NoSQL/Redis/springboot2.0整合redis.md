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

### KeyGenerator

缓存key生成器，定义了缓存key的生成方法
```java
@FunctionalInterface
public interface KeyGenerator {
    Object generate(Object var1, Method var2, Object... var3);
}
```
spring cache缓存的默认key生成策略是SimpleKeyGenerator

```java
public class SimpleKeyGenerator implements KeyGenerator {
    public SimpleKeyGenerator() {
    }

    public Object generate(Object target, Method method, Object... params) {
        return generateKey(params);
    }

    //生成key
    public static Object generateKey(Object... params) {
        //没有方法参数：key = 0
        if (params.length == 0) {
            return SimpleKey.EMPTY;
        } else {
            //参数个数为1：key = 第一个参数
            if (params.length == 1) {
                Object param = params[0];
                if (param != null && !param.getClass().isArray()) {
                    return param;
                }
            }
            //如果参数多于一个的话则使用所有参数的hashCode作为key。
            return new SimpleKey(params);
        }
    }
}
```
场景：当我们先调用了getModel1(1)，ehcache就会将方法的返回结果以"1"为key放入缓存中，当我们再调用getModel2(1)时，ehcache就会从缓存中找key为"1"的数据(即 Model1 )并试图将它转换为Model2 ，这就出现了异常:  Model1 can not be cast to Model2.....所以我们需要自定义key策略来解决这个问题，将类名和方法名和参数列表一起来生成key，下面是自定义的Key生成代码：
```java
    /**
     * 自定义key生成器
     *
     * key ---> hashCode( 类名（class）+ 方法名（method)+ 所有参数（params）)
     * @return
     */
    @Bean
    @Override
    public KeyGenerator keyGenerator() {
        return (target,method,params)->{
            // 采取拼接的方式生成key
            StringBuilder stringBuilder = new StringBuilder();
            // 目标类的类名
            stringBuilder.append(target.getClass().getName());
            stringBuilder.append(":");
            // 目标方法名
            stringBuilder.append(method.getName());
            // 参数
            for(Object object : params){
                stringBuilder.append(":"+String.valueOf(object));
            }
            String result = String.valueOf(stringBuilder.hashCode());
            log.info("自定义的key为："+result);
            return result;
        };
    }
```

