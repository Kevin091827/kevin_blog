# 前言

排行榜作为互联网应用中几乎必不可少的一个元素，能勾起人类自身对比的欲望，某宝中的商品销量排行，店铺信誉排行等，实现排行榜的方式也有很多种，可以使用快速排序算法 + 实现Comparator接口实现按某项权重排序，现在很多公司都在使用redis这个nosql数据库实现排行榜的功能


# 基于redis实现排行榜

现在要做的是对公司进行排行，排行的标准是用户对公司的搜索次数，做一个前十公司的排行榜

## 1.相关的redis知识

与排行榜功能实现相关的redis数据结构是sort set（有序集合）

### 关于sort set

我们知道set是一种集合，集合有一个特点就是无重复元素，sort set除了无重复元素外，还有一个特点就是有序性。

数据结构组成：
- key：sort set 的唯一标识
- 权重：也叫分数（score）redis通过权重为集合中的元素进行升序排序（默认），权重可以重复
- value：集合元素，元素不可重复

```redis
String(set key),double(权重),String(value)
```

sort set是通过哈希表实现的，所以添加，函数，查找的时间复杂度都是O(1),每个集合可以存储40多亿个元素


### 基本命令

**向集合中添加一个或多个元素**
```shell
ZADD "KEY" SCORE "VALUE" [ SCORE "VALUE"]
```
效果：
```shell
MyRedis:0>ZADD test 1 "one"
"1"
MyRedis:0>zadd test 4 "four" 5 "five"
"2"
```
**获取集合的元素数量**
```shell
ZCARD "key"
```
效果
```shell
MyRedis:0>ZCARD test
"5"
```
**获取指定元素分数（权重）**

```shell
ZSCORE "KEY" "VALUE"
```
效果
```shell
MyRedis:0>ZSCORE "test" "one"
"2"
```
**指定集合的指定元素增加指定分数**
```shell
ZINCRBY "key" score "value"
```
效果：
```shell
MyRedis:0>ZSCORE "test" "one"
"2"
MyRedis:0>ZINCRBY "test" 1 "one"
"3"
MyRedis:0>ZSCORE "test" "one" 
"3"
```
**获取指定范围的元素(默认按照分数|权重的升序排列)**

```shell
ZRANGE "key" 开始下标 结束下标
```
效果
```shell
MyRedis:0>ZRANGE "test" 0 1
 1)  "two"
 2)  "one"
```

完成这个需求大概需要这么多命令，接下来开始实现我们的这个需求

## 2.springboot + redis实现

导入redis依赖
```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

编写工具类

```java
    //=============================== sort set =================================

    /**
     * 添加指定元素到有序集合中
     * @param key
     * @param score
     * @param value
     * @return
     */
    public boolean sortSetAdd(String key,double score,String value){
        try{
            return redisTemplate.opsForZSet().add(key,value,score);
        }catch (Exception e){
            e.printStackTrace();
            return false;
        }
    }

    /**
     * 有序集合中对指定成员的分数加上增量 increment
     * @param key
     * @param value
     * @param i
     * @return
     */
    public double sortSetZincrby(String key,String value,double i){
        try {
            //返回新增元素后的分数
            return redisTemplate.opsForZSet().incrementScore(key, value, i);
        }catch(Exception e){
            e.printStackTrace();
            return -1;
        }
    }

    /**
     * 获得有序集合指定范围元素 (从大到小)
     * @param key
     * @param start
     * @param end
     * @return
     */
    public Set sortSetRange(String key,int start,int end){
        try {
            return redisTemplate.opsForZSet().reverseRange(key, start, end);
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
```

业务实现：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912132449715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

因为排行榜对实时性要求比较高，个人认为没必要进行持久化到数据库

```java
    /**
     * 根据公司名找到指定公司
     * @param companyName
     * @return
     */
    @Override
    public AjaxResult selectCompanyName(String companyName) {
        Set<Object> set =  redisUtils.sGet("company");
        for(Object i : set){
            String json = JSONObject.toJSONString(i);
            JSONObject jsonObject = JSONObject.parseObject(json);
            if(jsonObject.getString("companyName").equals(companyName)){
                //搜索次数 + 1
                redisUtils.sortSetZincrby("companyRank",companyName,1);
                log.info("直接缓存中返回");
                return new AjaxResult().ok(jsonObject);
            }
        }
        log.error("缓存中没有,查数据库");
        TbCommpanyExample tbCommpanyExample = new TbCommpanyExample();
        tbCommpanyExample.createCriteria().andCompanyNameEqualTo(companyName);
        List<TbCommpany> list = tbCommpanyMapper.selectByExample(tbCommpanyExample);
        if(list.size() != 0){
            //放入缓存中
            redisUtils.sSet("company",list.get(0));
            //数据库中存在
            //搜索次数 + 1
            redisUtils.sortSetZincrby("companyRank",companyName,1);
            log.info("sql");
            return new AjaxResult().ok(list.get(0));
        }else{
            return new AjaxResult().error("没有找到该公司："+companyName);
        }
    }
```
获取排名
```java
    /**
     * 获得公司排行榜(前十)
     * @return
     */
    @Override
    public AjaxResult getCompanyRank() {
        Set set = redisUtils.sortSetRange("companyRank",0,9);
        if(set.size() == 0){
            return new AjaxResult().error("公司排行榜为空");
        }
        return new AjaxResult().ok(set);
    }
```

## 3.测试与总结

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912141309208.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

postman测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190912141334573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

还有一个问题就是相同分数的排行问题

如果我希望A是先到的排在相同分数但是后到的B前边，这个问题该如何解决呢？

要解决这个问题，我们可以考虑在分数中加入时间戳，计算公式为：
```shell
带时间戳的分数 = 实际分数*10000000000 + (9999999999 – timestamp)
```
这个带时间的公司可以自己编写，尽量缩减误差
