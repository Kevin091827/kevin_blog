>今天看了一篇gitchat的文章，标题是  聊聊 Java String 源码的排序算法，从中有所感悟和思考，因此打算总结下自己看的过程中的收获

## 一，java.lang.Comparable 接口

Comparable 接口强制了实现类对象列表的排序。其排序称为自然顺序，其 compareTo 方法，称为自然比较法

```java
public interface Comparable<T> {
    public int compareTo(T o);
}
```
如果用this代表当前调用该compareTo方法的对象，obj是方法传入参数
则：
```xml
    this  <  obj   ---- 返回负数
    this  =  obj   ---- 返回 0
    this  >  obj   ---- 返回正数
```
Comparable接口的compareTo是一种内比较，即支持跟当前对象比较


## 二，java.util.Comparator 接口

Comparator可以认为是是一个外比较器,一个对象不支持自己和自己比较（没有实现Comparable接口），但是又想对两个对象进行比较

```java
public interface Comparator<T> {
    int compare(T o1, T o2);
    //省略...........
}
```

比较逻辑
```xml
    o1  <  o2   ---- 返回负数
    o1  =  o2   ---- 返回 0
    o1  >  o2   ---- 返回正数
```

## 三，聊聊string中的compareTo方法

String中实现的是Comparable接口来为String对象作出比较逻辑
```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence{
        //........
    }
```

先看一段示例
```java
/**
 * 字符串比较案例
 */
public class StringComparisonDemo {

    public static void main(String[] args) {
        String foo = "ABC";

        // 前面和后面每个字符完全一样，返回 0
        String bar01 = "ABC";
        System.out.println(foo.compareTo(bar01));

        // 前面每个字符完全一样，返回：后面就是字符串长度差
        String bar02 = "ABCD";
        String bar03 = "ABCDE";
        System.out.println(foo.compareTo(bar02)); // -1 (前面相等,foo 长度小 1)
        System.out.println(foo.compareTo(bar03)); // -2 (前面相等,foo 长度小 2)

        // 前面每个字符不完全一样，返回：出现不一样的字符 ASCII 差
        String bar04 = "ABD";
        String bar05 = "aABCD";
        System.out.println(foo.compareTo(bar04)); // -1  (foo 的 'C' 字符 ASCII 码值为 67，bar04 的 'D' 字符 ASCII 码值为 68。返回 67 - 68 = -1)
        System.out.println(foo.compareTo(bar05)); // -32 (foo 的 'A' 字符 ASCII 码值为 65，bar04 的 'a' 字符 ASCII 码值为 97。返回 65 - 97 = -32)

        String bysocket01 = "泥瓦匠";
        String bysocket02 = "瓦匠";
        System.out.println(bysocket01.compareTo(bysocket02));// -2049 （泥 和 瓦的 Unicode 差值）
    }
}
```

结果：
```xml
0
-1
-2
-1
-32
-2049
```
再结合上边示例看看String中对compareTo方法的实现
```java
    public int compareTo(String anotherString) {
        //len1：当前字符串长度
        int len1 = value.length;
        //len2：参数字符串长度
        int len2 = anotherString.value.length;
        //len1和len2两者最小值
        int lim = Math.min(len1, len2);
        //分别转为字符数组
        char v1[] = value;
        char v2[] = anotherString.value;

        int k = 0;
        //比较逻辑
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            //字符不同，则返回两字符的ASCII 码的差值
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        //相同则返回两字符长度差值
        return len1 - len2;
    }
```
所以从上面的源码中可以看到，string中的compareTo逻辑大概可以整理为

* 字符串前面部分的每个字符完全一样，返回：后面两个字符串长度差；

* 字符串前面部分的每个字符存在不一样，返回：出现不一样的字符 ASCII 码的差值。

* 字符串的每个字符完全一样，返回 0；

在String内部还有个静态内部类CaseInsensitiveComparator也实现了该接口
```java
private static class CaseInsensitiveComparator
            implements Comparator<String>, java.io.Serializable{
                //.................
            }
```
该重写的接口方法是String对象的大小写不敏感比较方法
```java
        public int compare(String s1, String s2) {
            int n1 = s1.length();
            int n2 = s2.length();
            int min = Math.min(n1, n2);
            for (int i = 0; i < min; i++) {
                char c1 = s1.charAt(i);
                char c2 = s2.charAt(i);
                //转大写
                if (c1 != c2) {
                    c1 = Character.toUpperCase(c1);
                    c2 = Character.toUpperCase(c2);
                    //还不一样则转小写
                    if (c1 != c2) {
                        c1 = Character.toLowerCase(c1);
                        c2 = Character.toLowerCase(c2);
                        //还不一样则：返回不一样字符的ASCII 码的差值。
                        if (c1 != c2) {
                            // No overflow because of numeric promotion
                            return c1 - c2;
                        }
                    }
                }
            }
            return n1 - n2;
        }
```