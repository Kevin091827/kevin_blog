# 业务场景

在前后端分离的场景下，越来越多的项目使用token作为接口的安全机制，APP或者web端使用token与后端接口交互，以达到安全的目的

## 拦截器

* 拦截器
    
    * 获取前端请求的token
    * 判断token是否为空
    * token不为空，则查询redis缓存是否有相关token
    * 有相关token则通过拦截，请求资源

```java
@Configuration
@Slf4j
public class JwtInterceptor implements HandlerInterceptor {

    @Autowired
    private TbGuestMapper tbGuestMapper;

    @Autowired
    private LoginMapper loginMapper;

    @Autowired
    private RedisUtils redisUtils;

    /**
     * 预处理方法
     *
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        // 允许跨域
        response.setHeader("Access-Control-Allow-Origin", "*");
        String token = request.getHeader("authorization");
        //服务器内置的token，用于测试接口
        if ("testToken".equals(token)) {
            return true;
        }

        // 1.----------- 解析jwt信息

        //判断token是否为null，使用Optional防止出现空指针异常（optional可以接受null值）
        Optional.ofNullable(token)
                .map(n -> {
                    //判断token是否过期，过期则抛出异常
                    try {
                        log.info("----->" + n);
                        log.info("----->" + JwtUtils.parseJWT(n));
                        return JwtUtils.parseJWT(n);
                    } catch (Exception e) {
                        log.warn(">>>>>>>>>>>>>>>>>>token不存在!，重新授权登录！>>>>>>>>>>>>>>>>>>");
                        throw new RuntimeException("token不存在!，重新授权登录！");
                    }
                });

//        // 2.------------- 数据库交互
//        //解密出该用户的unionid，并去数据库教验是否存在，不存在则让其重新授权登录
//        Claims claims = JwtUtils.parseJWT(token);
//        String base64UnionId = (String) claims.get("key");
//        if("admin".equals(claims.getSubject())){
//            //后台登录情况
//        }
//        byte[] decode = Base64.getDecoder().decode(base64UnionId);
//        String unionId = new String(decode);
//        TbGuestExample tbGuestExample = new TbGuestExample();
//        tbGuestExample.createCriteria().andUnionidEqualTo(unionId);
//        long i = tbGuestMapper.countByExample(tbGuestExample);
//        if(i < 1){
//            //不存在该用户
//            log.warn("用户unionid:" + unionId + "不存在于数据库中，重新授权登陆");
//            return false;
//        }

        //解析token
        Claims claims = JwtUtils.parseJWT(token);
        //获取登录token标识
        String base64UnionId = (String)claims.get("key");
        //分情况登录
        //后台登录
        if("admin".equals(claims.getSubject())){

        }
        // 小程序登录
        // 取出token中的unionid
        byte[] decode = Base64.getDecoder().decode(base64UnionId);
        String unionId = new String(decode);
        //查redis中有无该token
        String tokenKey = "token:"+unionId;
        log.info("redis中查到相关token");
        if (redisUtils.get(tokenKey) == null){
            //不存在该key
            log.warn("用户unionid:" + unionId + "不存在于数据库中，重新授权登陆");
            return false;
        }
        return true;
    }
}
```

## 1.登录授权获取token

第一种业务场景就是最普遍的登录授权获取到token后，前端带上token去请求资源接口，token具有一定有效期限

**流程图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916213535901.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**流程解释**

* 流程：
     * 1.前端传unionid给后端
     * 2.后端生成jwt后，以token:unionid为key将token存入redis
     * 3.调用拦截路径外的接口需要提供token。
     * 4.后端拿到token后查询redis是否有相关token，有则放行

```java
    @Autowired
    private RedisUtils redisUtils;

    /**
     * jwt + redis 实现登录授权
     * 流程：
     *      1.前端传unionid给后端
     *      2.后端生成jwt后，以token:unionid为key将token存入redis
     *      3.调用拦截路径外的接口需要提供token。
     *      4.后端拿到token后查询redis是否有相关token，有则放行
     * @param unionId
     * @return
     */
    @RequestMapping("/login")
    public AjaxResult login(@RequestParam("unionId") String unionId){
        // ----- 验证用户名密码 ------
        //JWT token
        String token = null;
        //生成token
        if(unionId != null || "".equals(unionId)) {
            try {
                Map claims = new HashMap<String, Integer>();
                String key = Base64.getEncoder().encodeToString(unionId.getBytes());
                claims.put("key", key);
                token = JwtUtils.createJwt(claims, "WxUser", 1000 * 60 * 60 * 72);
                log.info("生成的toen：" + token);
                //生成的token保存到redis中
                redisUtils.set("token:"+unionId,token,1000 * 60 * 60 * 72);
            }catch (Exception e){
                log.error("生成token失败: "+e.getMessage());
                throw new RuntimeException("生成token失败");
            }
        }
        return new AjaxResult().ok(token);
    }
```

## 2.登出使token失效

登出的时候需要删除相关token，使token失效

```java
    /**
     * 用户登出并且使jwt token失效
     * 流程：
     *      1.前端传入unionid
     *      2.后端删除redis中key为token:unionid的token
     * @Param unionId
     * @return
     */
    @RequestMapping("/loginOut")
    public AjaxResult loginOut(@RequestParam("unionId")String unionId){
        String key = "token:"+unionId;
        redisUtils.del(key);
        return new AjaxResult().ok("登出成功");
    }
```

**测试**

token为空情况下请求资源接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220005480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

请求结果:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220057918.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


使用已经登出的token去请求资源接口

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220149358.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

## 3.续期token

* 流程：

  * 1.前端传入unionid和续期时间

  * 2.后端重新设置token失效时间为：剩余时间 + 指定失效时间

```java
    /**
     * 续期token
     * 流程：
     *      1.前端传入unionid和续期时间
     *      2.后端重新设置token失效时间为：剩余时间 + 指定失效时间
     * @param unionId
     * @return
     */
    @RequestMapping("/renewalToken")
    public AjaxResult renewalToken(@RequestParam("unionId")String unionId,
                                   @RequestParam("time")long time){
        String key = "token:"+unionId;
        if(redisUtils.get(key) == null){
            return new AjaxResult().ok("token不存在");
        }
        long oldTime = redisUtils.getExpire(key);
        log.info("oldTime:"+oldTime);
        boolean b = redisUtils.expire(key,oldTime+time);
        return b == true ? new AjaxResult().ok("续期成功") : new AjaxResult().error("续期失败");
    }
```
**申请新的token**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220645274.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**剩余过期时间**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220927922.png)

**续期**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916220909815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**续期后的token过期时间**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916221047728.png)

## 4.获取新的token

当token失效，前端还可以请求这个restful接口获取有效token，流程和登录授权接口差不多

```java
    /**
     * 获取新的token
     * 流程：
     *      1.前端传入unionid后端刷新token
     *      2.后端刷新redis缓存
     * @param unionId
     * @return
     */
    @GetMapping(value = "/token/{unionId}")
    public AjaxResult getNewToken(@PathVariable(name = "unionId") String unionId) {
        String token = null;
        if (StringUtils.isBlank(unionId)) {
            return new AjaxResult(502, "unionId为空");
        }
        //获取token
        Map claims = new HashMap<String, Integer>();
        //在claims中添加通过base64加密过的unionId
        String encodeUnionid = Base64.getEncoder().encodeToString(unionId.getBytes());
        claims.put("key", encodeUnionid);
        //72h过期
        try {
            token = JwtUtils.createJwt(claims, "wxuser", 1000 * 60 * 60 * 72);
        } catch (Exception e) {
            e.printStackTrace();
        }
        if (!StringUtils.isBlank(token)) {
            redisUtils.set("token:"+unionId,token,1000 * 60 * 60 * 72);
            return new AjaxResult().ok(token);
        }
        return new AjaxResult().error("无法获取token");
    }
```
