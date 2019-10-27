## java中的字符串String

> 本文只是分析String的基本方法的源码

### 1.String类结构

String是一个不可变类，不可被继承，可以被序列化，可以比较大小，是一个有序字符的序列

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    
 //........   
}    
```

### 2. String数据结构定义

java中的String字符串对象，实际上也是一个字符数组

```java
private final char value[];
```

注意，该字符数组是不可变数组，final修饰，是不可变的

> 再谈不可变

可能有人说下面这段代码不就可以反驳String是不可变类这个观点了吗？

```java
String s = "ABCabc";
System.out.println("s = " + s);

s = "123456";
System.out.println("s = " + s);
```

输出：

```shell
s = ABCabc
s = 123456
```

这样String对象s不就改变了吗？其实不是这样的

首先创建一个String对象s，然后让s的值为“ABCabc”， 然后又让s的值为“123456”。 从打印结果可以看出，s的值确实改变了。那么怎么还说String对象是不可变的呢？ 其实这里存在一个误区： s只是一个String对象的引用，并不是对象本身。对象在内存中是一块内存区，成员变量越多，这块内存区占的空间越大。引用只是一个4字节的数据，里面存放了它所指向的对象的地址，通过这个地址可以访问对象。

也就是说，s只是一个引用，它指向了一个具体的对象，当s=“123456”; 这句代码执行过之后，又创建了一个新的对象“123456”， 而引用s重新指向了这个心的对象，原来的对象“ABCabc”还在内存中存在，并没有改变。内存结构如下图所示：

<img src="https://img-blog.csdnimg.cn/20191026162155882.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70" alt="在这里插入图片描述" style="zoom:50%;" />

实际上ABCabc，和123456在jvm中是存放在字符串常量池中的一个String Table中的一个引用



### 3.构造方法

> 构造空串

```java
    public String() {
        this.value = "".value;
    }
```

> 指定字符数组构造

```java
    public String(char value[]) {
        this.value = Arrays.copyOf(value, value.length);
    }
```

### 4.String常用方法

> 判断是否是空串和返回String长度

```java
    public int length() {       //所以String的长度就是一个value的长度
        return value.length;
    }


    public boolean isEmpty() {  //当char数组的长度为0，则代表String为"",空字符串
        return value.length == 0;
    }

```

> charAt函数，ChatAt是实现CharSequence 而重写的方法，是一个有序字符集的方法

```java
  	//获取指定下标的字符
	//T:O(1)
	public char charAt(int index) {
        //下标检查
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        //字符数组，直接按下标取字符
        return value[index];
    }
```

> codePointAt函数

```java
   	//返回String对象的char数组index位置的元素的ASSIC码(int类型)
	public int codePointAt(int index) {
        //检查下标
        if ((index < 0) || (index >= value.length)) {
            throw new StringIndexOutOfBoundsException(index);
        }
        //返回指定位置字符的ASCII码
        return Character.codePointAtImpl(value, index, value.length);
    }
```

> equals方法
>
> - 先判断是否是同一对象
> - 在判断是否是String类型
> - 转为String后，先比较长度
> - 最后一个字符一个字符的比较

```java
    //判断两个字符串对象的内容是否相同
	public boolean equals(Object anObject) {
        //先判断是否是同一个对象
        if (this == anObject) {
            return true;
        }
        //判断是否是String类型
        if (anObject instanceof String) {
            //转为String类型
            String anotherString = (String)anObject;
            //获取长度
            int n = value.length;
            //先判断长度是否相同
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                //一个字符一个字符的比较
                while (n-- != 0) {
                    //只要不同就返回false
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```

补充：关于java中equals和 ‘==’和hashcode的关系：

```java
    public static void main(String[] args) {
        String s1 = "abc";
        String str3 = new String("abc");
        System.out.println(s1.equals(str3));//true
        System.out.println(s1 == str3);//false
        System.out.println(s1.hashCode() == str3.hashCode());//true
    }
```

- 如果两个对象equals()方法相等则它们的hashCode返回值一定要相同，如果两个对象的hashCode返回值相同，但它们的equals()方法不一定相等。
- 两个对象的hashCode()返回值相等不能判断这两个对象是相等的（可能刚好出现哈希冲突，映射到哈希表的同一位置），但两个对象的hashcode()返回值不相等则可以判定两个对象一定不相等。
- 若 == 返回true（同一对象），则两边的对象的hashCode()返回值必须相等，若 == 返回false，则两边对象的hashCode()返回值可能相等，也可能不等

> 所以，java中重写equals也应尽量重写hashcode的原因如下：

当 equals 方法被重写时通常有必要重写 hashCode 方法来维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码，如果不这样做的话就会违反 hashCode 方法的常规约定，从而导致该类无法结合所有基于散列的集合一起正常运作，这样的集合包括 HashMap、HashSet、Hashtable 等。

hashCode 方法的常规约定如下：

- 程序执行期间只要对象 equals 方法比较操作所用到的信息没有被修改，则对这同一个对象无论调用多次 hashCode 方法都必须返回同一个整数。
- 如果两个对象根据 equals 方法比较是相等的则调用这两个对象中任意一个对象的 hashCode 方法都必须产生同样的整数结果。（对应上文中，两个对象equals()方法相等则它们的hashCode返回值一定要相同）
- 如果两个对象根据 equals 方法比较是不相等的，则调用这两个对象中任意一个对象的 hashCode 方法不一定要产生相同的整数结果（对应上文中，equals不同，hashcode可以相同可以不同）

> 忽略大小写的equals

```java
    public boolean equalsIgnoreCase(String anotherString) {
        return (this == anotherString) ? true
                : (anotherString != null)
                && (anotherString.value.length == value.length)
                && regionMatches(true, 0, anotherString, 0, value.length);
    }
```

> 字符串比较的compareTo方法

```java
    public int compareTo(String anotherString) {
        //当前String对象的长度
        int len1 = value.length;
        //参数即要比较对象的长度
        int len2 = anotherString.value.length;
        //取最小长度
        int lim = Math.min(len1, len2);
        //分别转为字符数组
        char v1[] = value;
        char v2[] = anotherString.value;
        int k = 0;
        //从较短字符串的第一个字符开始比较
        while (k < lim) {
            char c1 = v1[k];
            char c2 = v2[k];
            //只要其实一个不相等，返回字符ASSIC的差值,int类型
            if (c1 != c2) {
                return c1 - c2;
            }
            k++;
        }
        //如果两个字符串同样位置的索引都相等，返回长度差值，完全相等则为0
        return len1 - len2;
    }
```

s1的ASII码是97 + 98 + 99

s2的ASII码是97 + 98 + 100

s1 < s2，所以返回-1

```java
        String s1 = "abc";
        String s2 = "abd";
        System.out.println("abc : " + Integer.valueOf(s1.compareTo(s2)));//-1
```

补充：Comparable 和 Comparator的理解

- comparable是一个单方法的接口，支持当前对象和参数对象的比较

  ```java
  public interface Comparable<T> {
      public int compareTo(T o);
  }
  ```

  比较结果返回值

  ```shell
  this < o  //返回-1
  this = o  //返回0
  this > o  //返回1
  ```

- comparator接口是多方法接口，也可以用来作为一个比较器，但是这个比较器是两个参数对象之间的比较

  ```java
  int compare(T o1, T o2);
  
  ```

  结果返回

  ```shell
  o1 > o2 //返回1
  o1 = o2 //返回0
  o1 < o2 //返回-1
  ```

- 两个接口都可以用来实现自定义比较器



> indexOf函数

```java
    //返回目标字符串的下标
	public int indexOf(int ch) {
        return indexOf(ch, 0);
    }
	//从fromIndex开始，找到ch的下标
	public int indexOf(int ch, int fromIndex) {
        //max:字符串长度
        final int max = value.length;
        //检查fromIndex
        if (fromIndex < 0) {
            fromIndex = 0;
        } else if (fromIndex >= max) {
            //没找到ch
            return -1;
        }
        if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
            final char[] value = this.value;
            //从fromIndex开始往后遍历，直到找到ch，返回ch下标
            for (int i = fromIndex; i < max; i++) {
                if (value[i] == ch) {
                    return i;
                }
            }
            //没找到返回-1
            return -1;
        } else {
            return indexOfSupplementary(ch, fromIndex);
        }
    }
```

> subString方法

```java
//字符串截取，[beginIndex,endIndex)
public String substring(int beginIndex, int endIndex) {
        //检查beginIndex
    	if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
    	//检查endIndex
        if (endIndex > value.length) {
            throw new StringIndexOutOfBoundsException(endIndex);
        }
    	//截取长度
        int subLen = endIndex - beginIndex;
        //检查截取长度
    	if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
    	//如果刚好是0到this.value.length，就返回本字符串对象
    	//否则新new一个String对象，保证String不可变类这个特性
        return ((beginIndex == 0) && (endIndex == value.length)) ? this
                : new String(value, beginIndex, subLen);
    }
```

截取字符串实质还是字符数组的移动

```java
    public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count <= 0) {
            if (count < 0) {
                throw new StringIndexOutOfBoundsException(count);
            }
            if (offset <= value.length) {
                this.value = "".value;
                return;
            }
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
    }
```

### 5.String真的不可变吗？

```java
    public static void main(String[] args) {
        //反射
        String s = "abc";
        System.out.println("反射前："+s);
        try {
            Field value = String.class.getDeclaredField("value");
            value.setAccessible(true);
            char[] v = (char[]) value.get(s);
            v[0] = '5';
            System.out.println("反射后："+s);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

结果：

```shell
反射前：abc
反射后：5bc
```

通过反射，我们改变了底层的字符数组的值，实现了字符串的 “不可变” 性，这是一种骚操作，不建议这么使用，违反了 Java 对 String 类的不可变设计原则，会造成一些安全问题。







