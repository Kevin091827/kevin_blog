# 一，java中的加密算法

近期在写项目，很多时候都需要对签名进行加密处理然后在进行比对的操作，因此想总结总结java中比较常用到的一些加密算法

# 二，分类

看了一些博客，我认为，加密算法可以分为以下几种

* 可逆加密
* 不可逆加密
* 对称加密
* 非对称加密

接下来按照分类依次总结

## 1.可逆加密

### 什么是可逆加密？

可逆加密就是能将需要的明文进行加密，也能对原来加密后的密文进行解密，其中最经典也是比较常用到的就是Base64加密，可以依赖于jdk的Base64类的相关方法实现加密，解密

### 经典可逆加密 --- base64

#### base64简介

其实base64准确来说应该算是一种编码方式,Base64是网络上最常见的用于传输8Bit字节码的编码方式之一，Base64就是一种基于64个可打印字符来表示二进制数据的方法,在http协议中，传输二进制数据要将二进制数据转换成字符数据，然而直接转换是不行的。因为网络传输只能传输可打印字符，在ASCII码中规定0~31、127这33个字符属于控制字符，32~126这95个字符属于可打印字符，也就是说网络传输只能传输这95个字符，不在这个范围内的字符无法传输，想要传输这33个控制字符，就需要用到base64

#### base64加密思路

Base64的索引与对应字符的关系如下表所示：

![](https://img-blog.csdn.net/20180313122446386?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjA1NDUzNjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

每一个ASCII字符是8比特，而如果将索引转换为对应的二进制数据的话需要至多6个Bit,怎么使用6个比特表示8比特数据呢？--- 答案就是比特流分组（个人定义的名字）如下图：son 经过base64后变为U29u

![](https://img-blog.csdn.net/20180313131013494?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMjA1NDUzNjc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 如何实现？

```java
/**
 * @Description:    java加密模块---Base64实现加密解密(直接依赖于jdk，是可逆加密操作)
 * @Author:         Kevin
 * @CreateDate:     2019/5/2 1:44
 * @UpdateUser:     Kevin
 * @UpdateDate:     2019/5/2 1:44
 * @UpdateRemark:   修改内容
 * @Version: 1.0
 */
public class Base64Demo {

    /**
     * Base64加密
     * @param data
     * @return
     */
    public static String base64Encode(byte[] data){
        String result = null;
        try {
            Class clazz = Class.forName("com.sun.org.apache.xerces.internal.impl.dv.util.Base64");
            Method method = clazz.getMethod("encode",byte[].class);
            //Base64的加密方法是静态方法
            result = (String) method.invoke(null,data);

        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * Base64加密
     * @param data
     * @return
     */
    public static String EncodeBase64(byte[] data){
        return Base64.encode(data);
    }

    /**
     * Base64解密
     * @param data
     * @return
     */
    public static byte[] base64Decode(String data){
        byte[] result = null;
        try {
            Class clazz = Class.forName("com.sun.org.apache.xerces.internal.impl.dv.util.Base64");
            Method method = clazz.getMethod("decode",byte[].class);
            //Base64的加密方法是静态方法
            result = (byte[]) method.invoke(null,data);

        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * Base64解密
     * @param data
     * @return
     */
    public static byte[] decodeBase64(String data){
        return Base64.decode(data);
    }
}
```

## 2.不可逆加密

### 什么是不可逆加密？

不可逆加密就是明文加密后无法解密

### 经典不可逆算法
 
* md5加密
* sha加密

### md5算法

#### 简介

md5其实准确来说应该是消息摘要算法，用来确保信息传输的完整性，我们有时候下载东西一般都有个md5校验的工具，可以校验文件的完整性，不管文件多大，md5加密都会得到一个唯一的md5值，

#### MD5算法具有以下特点：

* 1、压缩性：任意长度的数据，算出的MD5值长度都是固定的。
* 2、容易计算：从原数据计算出MD5值很容易。
* 3、抗修改性：对原数据进行任何改动，哪怕只修改1个字节，所得到的MD5值都有很大区别。
* 4、弱抗碰撞：已知原数据和其MD5值，想找到一个具有相同MD5值的数据（即伪造数据）是非常困难的。
* 5、强抗碰撞：想找到两个不同的数据，使它们具有相同的MD5值，是非常困难的。

MD5的作用是让大容量信息在用数字签名软件签署私人密钥前被”压缩”成一种保密的格式（就是把一个任意长度的字节串变换成一定长的十六进制数字串）。除了MD5以外，其中比较有名的还有sha-1、RIPEMD以及Haval等。

### 实现

```maven
<!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.12</version>
</dependency>

```

```java
public class Md5Demo {

    /**
     * 方式一，导jar包：commons-codec-1.9
     * @param data
     * @return
     */
    public static String getMd5(byte[] data){
        return DigestUtils.md5Hex(data);
    }

    /**
     * 方式二：md5加密
     * @param decript 要加密的字符串
     * @return 加密的字符串
     * MD5加密
     */
    public static String getMd5(String decript) {
        char hexDigits[] = { // 用来将字节转换成 16 进制表示的字符
                '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd',
                'e', 'f'};
        try {
            byte[] strTemp = decript.getBytes();
            MessageDigest mdTemp = MessageDigest.getInstance("MD5");
            mdTemp.update(strTemp);
            byte tmp[] = mdTemp.digest(); // MD5 的计算结果是一个 128 位的长整数，
            // 用字节表示就是 16 个字节
            char strs[] = new char[16 * 2]; // 每个字节用 16 进制表示的话，使用两个字符，
            // 所以表示成 16 进制需要 32 个字符
            int k = 0; // 表示转换结果中对应的字符位置
            for (int i = 0; i < 16; i++) { // 从第一个字节开始，对 MD5 的每一个字节
                // 转换成 16 进制字符的转换
                byte byte0 = tmp[i]; // 取第 i 个字节
                strs[k++] = hexDigits[byte0 >>> 4 & 0xf]; // 取字节中高 4 位的数字转换,
                // >>> 为逻辑右移，将符号位一起右移
                strs[k++] = hexDigits[byte0 & 0xf]; // 取字节中低 4 位的数字转换
            }
            return new String(strs).toUpperCase(); // 换后的结果转换为字符串
        } catch (Exception e) {
            return null;
        }
    }
}
```

### sha算法

#### 简介

安全哈希算法（Secure Hash Algorithm）主要适用于数字签名标准（Digital Signature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA）。对于长度小于2^64位的消息，SHA1会产生一个160位的消息摘要。该算法经过加密专家多年来的发展和改进已日益完善，并被广泛使用。该算 法的思想是接收一段明文，然后以一种不可逆的方式将它转换成一段（通常更小）密文，也可以简单的理解为取一串输入码（称为预映射或信息），并把它们转化为 长度较短、位数固定的输出序列即散列值（也称为信息摘要或信息认证代码）的过程。散列函数值可以说是对明文的一种“指纹”或是“摘要”所以对散列值的数字 签名就可以视为对此明文的数字签名。

#### 实现

```java
public class Sha1Demo {

    /**
     * 方式一：依赖于第三方commons-codec-1.9 jar包实现sha1加密
     * @param data
     * @return
     */
    public static String getSha1(byte[] data){
        return DigestUtils.sha1Hex(data);
    }

    /**
     * 方式二：sha1加密
     * @param src
     * @return
     */
    public static String getSha1(String src) {
        try {
            //获取一个加密对象
            MessageDigest md = MessageDigest.getInstance("sha1");
            //加密
            byte[] digest = md.digest(src.getBytes());
            char[] chars= {'0','1','2','3','4','5','6','7','8','9','a','b','c','d','e','f'};
            StringBuilder sb = new StringBuilder();
            //处理加密结果
            for(byte b:digest) {
                sb.append(chars[(b>>4)&15]);
                sb.append(chars[b&15]);
            }
            return sb.toString();
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

### SHA-1与MD5的比较

因为二者均由MD4导出，SHA-1和MD5彼此很相似。相应的，他们的强度和其他特性也是相似，但还有以下几点不同：
* 1.对强行攻击的安全性：最显著和最重要的区别是SHA-1摘要比MD5摘要长32 位。使用强行技术，产生任何一个报文使其摘要等于给定报摘要的难度对MD5是2^128数量级的操作，而对SHA-1则是2^160数量级的操作。这 样，SHA-1对强行攻击有更大的强度。
* 2 对密码分析的安全性：由于MD5的设计，易受密码分析的攻击，SHA-1显得不易受这样的攻击。
* 3 速度：在相同的硬件上，SHA-1的运行速度比MD5慢。


## 对称加密

对称加密就是信息收发都有一条秘钥，消息的加密解密都依赖于这个秘钥

### 经典对称加密 --- AES

#### AES简介
高级加密标准，是下一代的加密算法标准，速度快，安全级别高；

#### 实现
```java
public class AES {

    /**
     * AES加密
     * @param data
     * @return
     */
    public static byte[] encrypt(String data,String key){
        byte[] result = null;
        try {
            Cipher cipher = Cipher.getInstance("AES");
            cipher.init(Cipher.ENCRYPT_MODE,getSecretKey(key));
            result = cipher.doFinal(data.getBytes());
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
    public static String decrypt(byte[] data,String key){
        String result = null;
        try {
            //获取cipher
            Cipher cipher = Cipher.getInstance("AES");
            //初始化cipher，并且指定模式
            cipher.init( Cipher.DECRYPT_MODE,getSecretKey(key));
            //加密
            result = new String(cipher.doFinal(data));
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 获得加密和解密所需的密钥
     * @return
     */
    private static SecretKey getSecretKey(String key){
        if (null == key || key.length() == 0) {
            throw new NullPointerException("key not is null");
        }
        SecretKey secretKey = null;
        try {
            KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
            SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
            secureRandom.setSeed(key.getBytes());
            keyGenerator.init(128,secureRandom);
            secretKey = keyGenerator.generateKey();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return secretKey;
    }

    public static void main(String[] args) {
        String pwd = "123456789";
        byte[] pwd1 = encrypt(pwd,"test");
        System.out.println("----->"+pwd1);
        System.out.println("----->"+decrypt(pwd1,"test"));
    }
}
```

## 非对称加密

非对称加密算法是一种密钥的保密方法。 非对称加密算法需要两个密钥：公开密钥（publickey）和私有密钥（privatekey）。 公开密钥与私有密钥是一对，如果用公开密钥对数据进行加密，只有用对应的私有密钥才能解密；如果用私有密钥对数据进行加密，那么只有用对应的公开密钥才能解密。

常见的非对称加密就是rsa加密算法

使用公钥进行加密，私钥进行解密

```java
/**
 * @Auther: Kevin
 * @Date:
 * @ClassName:RSA
 * @Description: TODO
 */
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
            keyPairGenerator.initialize(512,secureRandom);
            keyPair = keyPairGenerator.generateKeyPair();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return keyPair;
    }

    public static void main(String[] args) {
        KeyPair keyPair = getKeyPair();
        System.out.println("---->"+publicEncrytype("123456789",getPublicKey(keyPair)).toString());
        System.out.println("---->"+privateDecrytypr(publicEncrytype("123456789",getPublicKey(keyPair)),getPrivateKey(keyPair)));
    }
}
```





