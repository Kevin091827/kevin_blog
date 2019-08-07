
在进行前后端项目开发时，身份认证是一个很重要的问题。

在身份认证这一块上，我们可以使用JWT来进行验证

# 一，JWT简述

JWT(Json Web Token)是一种为了在网络应用环境之间传递声明而基于json的开放标准，JWT声明一般被采用在身份提供者和服务器提供者间传递被认证的身份信息，以便从资源服务器获取资源

JWT一般用于用户登录上，身份认证这种场景下，一旦用户登录完成，在接下来的每个涉及用户权限的请求中都包含JWT，可以对用户身份，路由，服务和资源的访问权限进行验证

举一个例子，假如一个电商网站，在用户登录以后，需要验证用户的地方其实有很多，比如购物车，订单页，个人中心等等，访问这些页面正常的逻辑是先验证用户权限和登录状态，如果验证通过，则进入访问的页面，否则重定向到登录页。

# 二，JWT对比session或cookie

在JWT之前，这样的身份认证大多数都是通过session和cookie去实现的

## 1.用户身份认证cookie session实现

![](https://img-blog.csdn.net/2018091121224432?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NTQ4OTU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

浏览器第一次请求服务器时，服务器会为浏览器的这次会话（session）生成一个索引为session_id的会话数据存入服务器的后端数据库，然后在cookie中存入session_id封装在响应信息中，返回给浏览器保存在本地主机的cookie文件（追加写入），第二次请求时带上cookie信息去请求服务器，服务器解析出cookie中的session_id去查询后端数据库返回相应信息

**缺陷**

1. 服务器需要保存session数据，数据量增大，服务器性能会受到很大的影响，内存开销大

2. 服务器session不能共享，严重的限制了服务器扩展能力， 比如说我用两个机器组成了一个集群， 小F通过机器A登录了系统，  那session id会保存在机器A上，  假设小F的下一次请求被转发到机器B怎么办？  机器B可没有小F的 session id啊。只能进行session复制转移到另一台机器

    ![](https://images2018.cnblogs.com/blog/1350514/201805/1350514-20180504122814029-1201707523.png)

3. 基于上述解决session不能共享的问题，还可以使用redis或者Memcached中作为session共享服务器，但是，如果共享服务器挂掉，就会影响到所有服务器的会话管理

    ![](https://images2018.cnblogs.com/blog/1350514/201805/1350514-20180504123036062-1920411426.png)

4. 使用本机cookie保存也会出现数据泄露或者遭到踹改的问题


## 2.用户身份认证JWT实现


### 1.JWT结构

JWT生成的token由三部分组成，用 . 隔开

- **header**

    header主要包括：
    - token类型
    - 所使用的加密算法

    ```json
    {
        typ: "jwt", 
        alg: "HS256"
    }
    ```

- **payload**

    payload主要是用来存放有效信息，有效信息中被分为标准中注册的声明，公共的声明和私有声明

    **下面是标准中注册的声明，建议但不强制使用。**

    - iss：jwt 签发者；
    - sub：jwt 所面向的用户；
    - aud：接收 jwt 的一方；
    - exp：jwt 的过期时间，这个过期时间必须要大于签发时间，这是一个秒数；
    - nbf：定义在什么时间之前，该 jwt 都是不可用的；
    - iat：jwt 的签发时间。

    上面的标准中注册的声明中常用的有 exp 和 nbf。

    **公共声明**

    公共声明可以添加任何信息，一般添加用户的相关信息或其他业务需要的必要信息，但不建议添加敏感信息，该部分在客户端可以解密获得

    **私有声明**

    私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为 base64 是对称解密的，意味着该部分信息可以归类为明文信息。

- **signature 签名**

    签名这一部分是指将header或payload通过秘钥进行加密后生成，秘钥存储在服务端，不会发送给任何人，所以jwt的运输方式很安全

    最后将三部分使用 . 连接成字符串，就是要返回给浏览器的 token 浏览器一般会将这个 token 存储在 localStorge 以备其他需要验证用户的请求使用。


### 2.实现身份认证

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807193552928.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

实现：

controller
```java
    @Autowired
    private LoginService loginService;

    /**
     * 登录
     * @return
     */
    @RequestMapping("/login")
    public AjaxResult login(@RequestParam("userName")String userName,
                            @RequestParam("password")String password){
        return loginService.login(userName, password);
    }
```
service:

```java
@Slf4j
@Service
@Transactional
public class LoginServiceImpl implements LoginService {

    @Autowired
    private LoginMapper loginMapper;

    @Override
    public AjaxResult login(String userName, String password) {
        Guest guest =  loginMapper.selectFromTbGuest(userName, password);
        //校验
        if(guest != null){
            //校验成功
            //信息封装返回给前端
            Map resultMap = new HashMap<String,String>();
            //登录成功
            String token = null;
            //生成token
            try{
                //加密用户名和密码(仅为参考，一般不存储用户的隐私敏感信息)
                String key = Base64.getEncoder().encodeToString((guest.getUserName()+","+guest.getPassword()).getBytes());
                Map claims = new HashMap();
                claims.put("key",key);
                token = JwtUtils.createJwt(claims,"admin",720000);
                resultMap.put("token",token);
            }catch (Exception e){
                log.error("生成token失败"+e.getMessage());
            }
            resultMap.put("msg","登录成功");
            //封装返回信息
            return new AjaxResult().ok(resultMap);
        }else {

            return new AjaxResult().error("登录失败");
        }
    }
}
```
jwt工具类
```java
@Component
public class JwtUtils {

    private static String JWT_SECRET = "123456789abcdefg";

    /**
     * 签发jwt
     * @param claims
     * @param subject   主题
     * @param ttlMillis 存活时间
     * @return
     */
    public static String createJwt(Map claims, String subject, long ttlMillis){
        //jwt头部中的签名算法，默认是Hsha256
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        //jwt签发时间
        long now = System.currentTimeMillis();
        Date nowTime = new Date(now);
        //生成签名时使用的秘钥
        SecretKey secretKey = generalKey();
        //构造payload
        JwtBuilder jwtBuilder = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(nowTime)
                .setSubject(subject)
                .signWith(signatureAlgorithm,secretKey);
        if(ttlMillis >= 0){
            //设置过期时间
            Date expireTime = new Date(now + ttlMillis);
            jwtBuilder.setExpiration(expireTime);
        }
        //
        return jwtBuilder.compact();
    }

    /**
     * 签名秘钥
     * @return
     */
    private static SecretKey generalKey() {
        //使用base64解码
        byte[] data = Base64.decodeBase64(JWT_SECRET);
        //AES构造指定的秘钥
        SecretKey secretKey = new SecretKeySpec(data,0,data.length,"AES");
        return secretKey;
    }

    /**
     * 解析jwt获取claims数据对象
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {

        SecretKey secretKey = generalKey();
        Claims claims = Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(jwt)
                .getBody();
        return claims;
    }
}
```
拦截器设置：

```java
@Configuration
@Slf4j
public class JwtInterceptor implements HandlerInterceptor {

    @Autowired
    private LoginMapper loginMapper;

    /**
     * 预处理方法
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
                        log.info("----->"+n);
                        return JwtUtils.parseJWT(n);
                    } catch (Exception e) {
                        log.warn(">>>>>>>>>>>>>>>>>>token不存在!，重新授权登录！>>>>>>>>>>>>>>>>>>");
                        throw new RuntimeException("token不存在!，重新授权登录！");
                    }
                });

        // 2.------------- 数据库交互

        String subject = JwtUtils.parseJWT(token).getSubject();
        if("admin".equals(subject)){
            //管理员
            //取出claims中的用户标识
            byte[] bytes = Base64.getDecoder().decode(String.valueOf(JwtUtils.parseJWT(token).get("key")));
            //byte[]转String
            String userInfo = new String(bytes);
            //分割userInfo，获取userName和password
            String[] strs = userInfo.split(",");
            //校验用户是否存在数据库
            Guest guest = loginMapper.selectFromTbGuest(strs[0],strs[1]);
            return guest != null;
        }
        return false;
    }
}
```

测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807195656662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

使用token请求资源api

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190807195806689.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

总结：

JWT本身没啥难度，但是安全整体是一件比较复杂的事情，JWT只不过负责提供了一种基于token的身份验证机制，但是对于我们的用户权限，对于相应用户权限的api划分，资源的权限划分等，都不是JWT负责的，也就是说，请求验证完成，是否有权限请求对应的内容还是由用户权限决定
