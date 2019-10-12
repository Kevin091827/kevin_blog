# springboot专题复习spring data MongoDB



## 一，MongoDB复习

### 1.MongoDB概述

MongoDB是一款分布式，高性能的文档数据库。 



### 2.集合操作

> 显示所有集合

```shell
show collections
```

> 创建集合

```shell
db.createCollection("collectionName")
```

> 删除集合

```shell
db.collectionName.drop()
```

综上示例：

```shell
show collections

db.createCollection("userInfo")

show collections

db.userInfo.drop()

show collections
```



### 3.文档操作

文档的数据结构和 JSON 基本一样。

所有存储在集合中的数据都是 BSON 格式。

BSON 是一种类似 JSON 的二进制形式的存储格式，是 Binary JSON 的简称。

> 插入文档

基本格式：

```shell
db.COLLECTION_NAME.insert(document)
db.COLLECTION_NAME.save(document)
```

如果不指定 _id 字段 save() 方法类似于 insert() 方法。如果指定 _id 字段，则会更新该 _id 的数据。

示例：

```shell
//使用save插入
db.userInfo.save({
    title : 'mongodb',
    author : 'kevin',
    date : 'today'
});

//使用insert插入
db.userInfo.insert({
    title : 'mongodb',
    author : 'kevin',
    date : 'today1'
});

//也可以将文档封装成一个变量插入
document = ({
    title : 'mongodb',
    author : 'kevin',
    date : 'today2'
});
db.userInfo.insert(document)
```

> 文档查找

查询全部：

```shell
db.COLLECTION_NAME.find(query,projection)
//query:指定查询条件（相当于sql中的where后边跟的属性），可选
//projection：指定查询匹配值（相当于sql中where后边跟的属性后边=的内容），可选
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012132451666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

格式化显示所有文档：

```shell
db.userInfo.find().pretty()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012133003600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

条件查询

> AND条件

基本格式：直接,拼接就相当于and

```shell
db.col.find({key1:value1, key2:value2}).pretty()
```

根据id和author查找

```shell
db.userInfo.find({'author':'kevin','_id': ObjectId("5da161f3055a042559aef9f0")}).pretty()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012133600990.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

> OR条件

基本格式：使用到$or关键字

```shell
db.col.find(
   {
      $or: [
         {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

示例：

```shell
db.userInfo.find({$or:[{"date":"today"},{"author": "kevin"}]}).pretty()
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012134504507.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

> 其他条件符

```shell
$gt -------- greater than  >

$gte --------- gt equal  >=

$lt -------- less than  <

$lte --------- lt equal  <=

$ne ----------- not equal  !=

$eq  --------  equal  =
```

> 文档更新

基本格式：

```shell
db.collection.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

```shell
db.userInfo.update({"date":"today2"},{$set:{"author" : "james"}})
```



> 删除文档

基本格式：

```shell
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```

**参数说明：**

- **query** :（可选）删除的文档的条件。
- **justOne** : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- **writeConcern** :（可选）抛出异常的级别。

```shell
db.userInfo.remove({"date":"today2"})
```



> MongoDB增删改查示例

```shell
show collections

db.createCollection("userInfo")

db.userInfo.save({
    title : 'mongodb',
    author : 'kevin',
    date : 'today'
});

db.userInfo.insert({
    title : 'mongodb',
    author : 'kevin',
    date : 'today1'
});

document = ({
    title : 'mongodb',
    author : 'kevin',
    date : 'today2'
});

db.userInfo.insert(document)

db.userInfo.find()

db.userInfo.find().pretty()

db.userInfo.find({'author':'kevin','_id': ObjectId("5da161f3055a042559aef9f0")}).pretty()

db.userInfo.find({$or:[{"date":"today"},{"author": "kevin"}]}).pretty()

db.userInfo.update({"date":"today2"},{$set:{"author" : "james"}})

db.userInfo.remove({"date":"today2"})
```



## 二，springboot2.x整合MongoDB

### 1.引入依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-mongodb</artifactId>
        </dependency>
```

### 2.CRUD

MongoTemplate给我们提供了很多操作MongoDB的api

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012140357600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012140433972.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

对于基本的增删改查操作来说，这些api已经足够应对我们的开发了

> 插入数据

```java
    /**
     * 插入
     * @param logInfo
     */
    @Override
    public void insert(LogInfo logInfo) {
        //插入
        mongoTemplate.save(logInfo);
        //可以使用save或者insert完成插入
        //mongoTemplate.insert(logInfo,"collection_name");
    }
```

> 更新

Query对象就想相当查询的条件，使用Update对象来封装要set的属性，和jpa中的操作类似

```java
    @Override
    public void update(String requestIp,LogInfo logInfo) {
        //通过where来设置update依据的属性，然后在来选择update依据属性的依据值（类似jpa）
        //  类似：gt：大于，gte：大于等于
        //      Query query1 = new Query(Criteria.where("requestIp").gt(requestIp));
        Query query = new Query(Criteria.where("requestIp").is(requestIp));
        //更新采用update对象
        //  类似sql中的set，（属性，值）
        Update update = new Update();
        update.set("gmtCreate",logInfo.getGmtCreate());
        //更新一条数据
        mongoTemplate.updateFirst(query,update,LogInfo.class);
        //更新多条数据
        mongoTemplate.updateMulti(query,update,LogInfo.class);
        //默认返回旧对象，然后更新新对象
        mongoTemplate.findAndModify(query,update,LogInfo.class);
    }
```

> 删除

```java
    @Override
    public void delete(String requestIp) {
        //删除和更新类似
        //通过where来设置update依据的属性，然后在来选择update依据属性的依据值（类似jpa）
        //  类似：gt：大于，gte：大于等于
        //      Query query1 = new Query(Criteria.where("requestIp").gt(requestIp));
        Query query = new Query(Criteria.where("requestIp").is(requestIp));
        //调用remove删除
        mongoTemplate.remove(query,LogInfo.class);
        //带返回值的删除
        List<LogInfo> list = mongoTemplate.findAllAndRemove(query,LogInfo.class);
    }
```

对于springboot整合MongoDB这块，我基于spring aop和MongoDB做了一个日志储存模块，利用MongoDB来存储我们的日志，方便日后进行日志分析	

### 3.基于spring aop和MongoDB的日志储存管理

封装的日志实体：

```java
@Data
public class LogInfo {

    private String requestMethod;
    private String requestIp;
    private String requestUrl;
    private Map<String,String[]> parameters;
    private Object[] args;
    private Date gmtCreate;
    private String requestTime;
    private Object returnValue;
    private String errorMsg;
}
```

切面类

```java
@Component
@Aspect
@Slf4j
public class AopUtils {

    @Autowired
    private LogInfoService logInfoService;

    /**
     * 切入点表达式
     *      1.格式：方法修饰符 + 返回值类型 + 包名 + 类名 + 方法名 +方法参数
     */
    @Pointcut("execution(public * com.springboot.learning.controller.*.*(..))")
    public void log(){

    }

    //环绕通知
    @Around("log()")
    public Object arround(ProceedingJoinPoint joinPoint) {
        LogInfo logInfo = new LogInfo();
        try {
            log.info("前置通知");
            // 接收到请求，记录请求内容
            ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = attributes.getRequest();
            //请求方式
            String requestMethod = request.getMethod();
            //请求的ip
            String requestIp = request.getRemoteAddr();
            //请求的url
            String requestUrl = request.getRequestURI().toString();
            //请求的参数
            Map<String,String[]> parameters = request.getParameterMap();
            Object[] args = joinPoint.getArgs();
            logInfo.setArgs(args);
            //请求时间
            Date requestDate = new Date();
            //请求精确时间
            logInfo.setGmtCreate(requestDate);
            //请求时间：yyyy-mm-dd
            DateFormat dateFormat = new SimpleDateFormat("yyyy年MM月dd日 HH时mm分ss秒");
            //格式转换
            String requestTime = dateFormat.format(requestDate);
            logInfo.setRequestTime(requestTime);
            logInfo.setParameters(parameters);
            logInfo.setRequestIp(requestIp);
            logInfo.setRequestMethod(requestMethod);
            logInfo.setRequestUrl(requestUrl);

            //调用目标方法
            /**
             * Spring AOP的环绕通知和前置、后置通知有着很大的区别，主要有两个重要的区别：
             *      1）目标方法的调用由环绕通知决定，即你可以决定是否调用目标方法，
             *         而前置和后置通知是不能决定的，它们只是在方法的调用前后执行通知而已，
             *         即目标方法肯定是要执行的。joinPoint.proceed()就是执行目标方法的代码。
             *      2）环绕通知可以控制返回对象，即可以返回一个与目标对象完全不同的返回值。虽然这很危险，但是却可以做到。
             */
            Object result = joinPoint.proceed();
            log.info("后置通知");
            //目标方法返回值
            logInfo.setReturnValue(result);
            return result;
        } catch (Throwable e) {
            //异常信息
            String errorMsg = e.getMessage();
            logInfo.setErrorMsg(errorMsg);
            return null;
        }finally {
            //插入MongoDB
            logInfoService.insert(logInfo);
        }
    }




//
//    /**
//     * 前置通知
//     *      方法执行前通知
//     */
//    @Before("log()")
//    public void before(JoinPoint joinPoint) throws Exception {
//
//    }
//
//
//    /**
//     * 后置通知
//     *      方法执行后通知
//     */
//    @After("log()")
//    public void after(JoinPoint joinPoint){
//        log.info("后置通知");
//
//    }
//
//    /**
//     * 方法返回通知
//     *      接收方法返回值
//     * @param result
//     */
//    @AfterReturning(returning = "result",pointcut = "log()")
//    public void afterReturning(Object result){
//        //获取方法返回值
//        LogInfo logInfo  = new LogInfo();
//        logInfo.setReturnValue(result);
//    }
//
//    /**
//     * 后置异常通知
//     *      当方法抛出异常时调用
//     */
//    @AfterThrowing("log()")
//    public void afterThrowing(JoinPoint joinPoint){
//
//    }
}
```

从controller层开始进行请求的监控，记录请求日志

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191012141148236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

[github](https://github.com/Kevin091827/springboot_learning)