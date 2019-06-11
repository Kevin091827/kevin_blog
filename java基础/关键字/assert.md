
## 一，断言检查
assertion(断言)在软件开发中是一种常用的调试方式，assertion就是在程序中的一条语句，它对一个boolean表达式进行检查，一个正确程序必须保证这个boolean表达式的值为true；如果该值为false，说明程序已经处于不正确的状态下，系统将给出警告并且退出。一般来说，assertion用于保证程序最基本、关键的正确性。assertion检查通常在开发和测试时开启。为了提高性能，在软件发布后，assertion检查通常是关闭的。 

## 二，java中的断言

java中的断言主要是通过assert关键字来执行一个表达式判断进行

```java 
//如果表达式为false，则抛出java.lang.AssertionError异常
assert expression;

//如果表达式为false，则抛出java.lang.AssertionError异常并输出错误信息errormessage
assert expressiion : message;

```

## 三，示例

idea默认是关闭assert断言检查的，所以首先开启断言检查

在vm中输入 -ea即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019061113211523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


示例程序

```java
public class AssertTest {

    public static void main(String[] args) {

        test(-1);
    }

    /**
     * 断言检查
     * @param a
     */
    public static void test(int a){
        assert a>0 : "输入小于或等于0";
        System.out.println("断言正确，继续执行");
    }
}
```

结果：如果程序正确，则打印下边的一句话，不正确则在断言出抛出异常并且推出

a = -1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190611131924959.png)


a = 1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190611131958536.png)

## 四，spring中的断言

在spring中也为我们提供了断言的工具类
Assert 方便我们进行断言测试


常用断言
```java

Assert.state(expression, "message");

Assert.state(expression);
```