# 一，stream概述

**什么是stream？**

这里的stream不同于IO中的inputStream和OutputStream，stream位于java.util.stream中，是一组支持串行并行聚合操作的元素，也可以理解成集合或者迭代器的增强版

**特征：**

- 单次处理，一次处理结束后，当前stream就关闭了

- 支持并行操作

# 二，使用

- **allMatch和anyMatch**

    allMatch：检查 Stream 中的所有元素，全部都通过检测则返回 true，否则 false

    anyMatch：检查 Stream 中的所有元素，至少有一个通过检测则返回 true，否则 false 
    
    ```java
        //allMatch
        System.out.println("----->"+list.stream().allMatch(n->n.getClass() == Integer.class));
        //anyMatch
        System.out.println("----->"+list.stream().anyMatch(n->n>1));
    ```
- **max，min和sort**

    max：返回stream中的最大值

    min：返回stream中的最小值

    sort：对stream中的元素排序

    ```java
            //遍历
            list.stream().forEach(a -> System.out.println(a));
            //max
            System.out.println("----->"+list.stream().max((a,b)->a-b).get());
            //min
            System.out.println("----->"+list.stream().min((a,b)->a-b).get());
            //sort
            list.stream().sorted((a,b)->a-b).forEach(a->System.out.println(a));
    ```
- **filter和map**

    filter：筛选 Stream 元素，符合条件的留下并组成一个新的 Stream 。

    map：依次对 Stream 中的元素进行指定的函数操作，并将按顺序将函数操作的返回值组合到一个新的 Stream 中。

    ```java
            //筛选filter
            list.stream().filter(n->n>6).forEach(n->System.out.println(n));
            //map
            list.stream().map(n->n+1).forEach(n->System.out.println(n));
    ```

- **distinct和collect**

    distinct：去重

    collect：可以做collectors中的一些操作，如：连接，转list，分组等

    ```java
            //collect
            Stream.of("1","2","3").collect(Collectors.toList()).forEach(n->System.out.println(n));
            //去重distinct
            Stream.of("2","2","2").distinct().forEach(n->System.out.println(n));
    ```
还有很多详细的用法，没有举出，可以参考：

[stream api](https://buzheng.org/post/20160226-java-stream-api-notes/)

# 三,Optional类

Optional类是java8新增的一个类，用以解决程序中常见的NullPointerException异常问题

NullPointerException是我们很经常遇到的一个问题，大多人遇到这个异常的时候，可能会在出现异常的代码处添加if-else的判断，类似下边代码；
```java
public void bindUserToRole(User user) {
    if (user != null) {
        String roleId = user.getRoleId();
        if (roleId != null) {
            Role role = roleDao.findOne(roleId);
            if (role != null) {
                role.setUserId(user.getUserId());
                roleDao.save(role);
            }
        }
    }
}
```
但是，这样手动处理null，如果数据增多，那么，相应的判断也会增多，业务逻辑就会被这大量的if判断淹没，对于代码维护和可读性是很打击的

在java8推出的Optional类就是用以避免null值引发的种种问题

## 1.概况：

java.util.Optional<T>类是一个封装了Optional值的容器对象，Optional值可以为null，如果值存在，调用isPresent()方法返回true，调用get()方法可以获取值。

## 2.创建Optional对象

Optional类提供类三个方法用于实例化一个Optional对象，它们分别为empty()、of()、ofNullable()，这三个方法都是静态方法，可以直接调用。
```java
private final T value;//Optional维护的值
```

默认构造：(主要处理null值)
```java
    private Optional() {
        this.value = null;
    }
```
有参构造：
```java
    private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
```



empty()方法用于创建一个没有值的Optional对象：

```java
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
```
empty()方法创建的对象没有值，如果对emptyOpt变量调用isPresent()方法会返回false，调用get()方法抛出NullPointerException异常。

of()方法使用一个非空的值创建Optional对象：

```java
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }
```

## 3.接收值

ofNullable()方法接收一个可以为null的值：

```java
Optional<String> nullableOpt = Optional.ofNullable(str);
```
如果str的值为null，得到的nullableOpt是一个没有值的Optional对象。
源码分析：
```java
    public static <T> Optional<T> ofNullable(T value) {
        //根据值是否为null，调用相应构造方法构造相应Optional对象
        return value == null ? empty() : of(value);
    }
```
### 4.获取Optional对象中的值

如果我们要获取User对象中的roleId属性值，常见的方式是直接获取：
```java
String roleId = null;
if (user != null) {
    roleId = user.getRoleId();
}
```
复制代码使用Optional中提供的map()方法可以以更简单的方式实现：
```java
Optional<User> userOpt = Optional.ofNullable(user);
Optional<String> roleIdOpt = userOpt.map(User::getRoleId);
```
Optional类还包含其他方法用于获取值，这些方法分别为：

- orElse()：如果有值就返回，否则返回一个给定的值作为默认值；
- orElseGet()：与orElse()方法作用类似，区别在于生成默认值的方式不同。该方法接受一个Supplier<? extends T>函数式接口参数，用于生成默认值；
- orElseThrow()：与前面介绍的get()方法类似，当值为null时调用这两个方法都会抛出NullPointerException异常，区别在于该方法可以指定抛出的异常类型。

### 5.简单应用

**简化if-else**

```java
User user = ...
if (user != null) {
    String userName = user.getUserName();
    if (userName != null) {
        return userName.toUpperCase();
    } else {
        return null;
    }
} else {
    return null;
}
```
简化成：

```java
User user = ...
Optional<User> userOpt = Optional.ofNullable(user);

return userOpt.map(User::getUserName)
            .map(String::toUpperCase)
            .orElse(null);
```

注意点：

- 尽量避免在程序中直接调用Optional对象的get()和isPresent()方法，直接调用get()方法是很危险的做法，如果Optional的值为空，那么毫无疑问会抛出NullPointerException异常，而为了调用get()方法而使用isPresent()方法作为空值检查，这种做法与传统的用if语句块做空值检查没有任何区别。

- 避免使用Optional类型声明实体类属性，Optional没有实现Serializable序列化接口 