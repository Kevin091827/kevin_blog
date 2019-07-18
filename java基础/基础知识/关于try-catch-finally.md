
先看这一道题目，输出结果：
```java
public class test {

    public static void main(String[] args) {
        System.out.println(testTry(9));
    }

    public static String testTry(int i){
        try{
            int a = i/0;
            return " --- --- ";
        }catch (Exception e){
            System.out.println("error");
            return "--- error --- ";
        }finally {
            return "--- finally ---";
        }
    }
}
```
结果是：

```shell
error
--- finally ---
```

如果把报错语句去掉
```java
    public static String testTry(int i){
        try{
           // int a = i/0;
            return " --- --- ";
        }catch (Exception e){
            System.out.println("error");
            return "--- error --- ";
        }finally {
            return "--- finally ---";
        }
    }
```
输出：
```shell
--- finally ---
```

去掉finally语句
```java
    public static String testTry(int i){
        try{
            int a = i/0;
            return " --- --- ";
        }catch (Exception e){
            System.out.println("error");
            return "--- error --- ";
        }
    }
```
输出：
```shell
error
--- error --- 
```


结论：

- 对于try-catch-finally机制，如果存在finally块，有返回值的话，返回值都是在finally块返回

- 对于不存在finally块的，如果正常执行，没报错则在try返回，报错则在catch返回