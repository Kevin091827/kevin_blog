### 简介

http是超文本安全传输协议，由于http在网络上是明文传输，可能会被拦截，而https是加密传输，加密和解密工作交由ssl完成

区别：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164954575.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801165219965.png)

HTTPS和HTTP的区别主要为以下四点：

- 一、https协议需要到ca申请证书，一般免费证书很少，需要交费。
- 二、http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
- 三、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
- 四、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

### ssl加密解密

加密方法：

**对称加密：**

对称加密原理：

- 对称加密靠秘钥来进行加密，也靠同一个秘钥来进行解密

常用的对称加密算法：

- DES
- 3DES
- AES

```java
    /**
     * AES加密
     * @param data
     * @return
     */
    public static String encrypt(String data){
        String result = null;
        try {
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.ENCRYPT_MODE,getSecretKey());
            result = new String(cipher.doFinal(data.getBytes("utf-8")));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * AES解密
     * @param data
     * @return
     */
    public static String decrypt(String data){
        String result = null;
        try{
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.DECRYPT_MODE,getSecretKey());
            result = new String(cipher.doFinal(data.getBytes("utf-8")));
        }catch (Exception e){
            e.getMessage();
        }
        return result;
    }

    /**
     * 获得加密和解密所需的密钥
     * @return
     */
    private static SecretKey getSecretKey(){
        SecretKey secretKey = null;
        try {
            KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
            SecureRandom secureRandom = new SecureRandom();
            keyGenerator.init(secureRandom);
            secretKey = keyGenerator.generateKey();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return secretKey;
    }
```
对称加密算法的优缺点：

- 优点：计算量相对较小，可以处理数据量较大的数据的加密工作
- 缺点：密钥容易泄露，且服务器管理密钥比较麻烦

**非对称加密**

非对称加密原理：

- 非对称加密靠公钥来进行加密，靠私钥来进行解密

常见的非对称加密算法：

- rsa加密算法

```java
public class RSA {

    /**
     * 公钥加密
     * @param data
     * @param publicKey
     * @return
     */
    public static byte[] publicEncrytype(String data,PublicKey publicKey){
        byte[] result = null;
        try {
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.ENCRYPT_MODE,publicKey);
            result = cipher.doFinal(data.getBytes("UTF-8"));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 私钥解密
     * @param data
     * @param privateKey
     * @return
     */
    public static String privateDecrytypr(byte[] data,PrivateKey privateKey){
        String result = null;
        try {
            Cipher cipher = Cipher.getInstance("RSA");
            cipher.init(Cipher.DECRYPT_MODE,privateKey);
            result = new String(cipher.doFinal(data),"utf-8");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }


    /**
     * 获得公钥
     * @param keyPair
     * @return
     */
    private static PublicKey getPublicKey(KeyPair keyPair){
        return keyPair.getPublic();
    }

    /**
     * 获得私钥
     * @param keyPair
     * @return
     */
    private static PrivateKey getPrivateKey(KeyPair keyPair){
        return keyPair.getPrivate();
    }

    /**
     * 获得密钥对
     * @return
     */
    private static KeyPair getKeyPair(){
        KeyPair keyPair = null;
        try {
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            SecureRandom secureRandom = new SecureRandom();
            keyPairGenerator.initialize(64,secureRandom);
            keyPair = keyPairGenerator.generateKeyPair();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return keyPair;
    }
}
```

非对称加密优缺点：

- 优点：数据传输安全
- 缺点：计算量大，加密和解密速度较慢

### https工作流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080118410042.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

[https原理简单介绍](https://blog.csdn.net/liupeifeng3514/article/details/79840274)