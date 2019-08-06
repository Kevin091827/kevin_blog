
lambda表达式是java8新引入的一个新的新特性，类似于js中的闭包，目的就是提供一个类似函数式编程的语法来简化我们的编码

## 一，lambda基本语法

其实lambda表达式跟原来写一个方法思路是很相似的，只是，lambda表达式可以使我们原来的方法简单化编码，更加清晰明了

基本结构：

```java
//(方法参数) -> 方法体
(args)->body
```

常见的基本写法可以有：
```java
/**
* 1. 参数类型可推导时，不需要指定类型，如 (a) -> System.out.println(a)
* 2. 当只有一个参数且类型可推导时，不强制写 (), 如 a -> System.out.println(a)
* 3. 参数指定类型时，必须有括号，如 (int a) -> System.out.println(a)
* 4. 参数可以为空，如 () -> System.out.println(“hello”)
* 5. body 需要用 {} 包含语句，当只有一条语句时 {} 可省略
*/

(a) -> a * a
(int a, int b) -> a + b
(a, b) -> {return a - b;}
() -> System.out.println(Thread.currentThread().getId())
```

## 二，函数式接口

java中的lambda表达式是以函数式接口为基础的

**什么是函数式接口？**

函数式接口（FunctionalInterface)就是只有一个方法的接口，这类接口的目的就是进行单一操作，这些函数式接口一般都带有注解@FunctionalInterface 

EG:

平时我们创建线程使用的Runnable接口就是一个函数式接口

原始创建线程的方式：
```java
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("没有使用lambda表达式");
            }
        }).start();
    }
```
接下来我们使用lambda表达式创建线程：

```java
new Thread(()->System.out.println("使用lambda表达式创建线程")).start();
```
是不是更加简单清晰了

注意看Thread的参数，原来我们需要传一个Runable实现类或者以匿名内部类的方法编写，但是现在，我们可以使用lambda表达式来作为函数参数，由此可见，Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）


之前我们写过一个按jsonArray某字段进行排序的一个方法，但是这个方法中的比较器是使用传统方式编写的，现在我们来用lambda试着简化一下：

原函数：

```java
    /**
     * 对json数组排序，
     * @param jsonArr
     * @param sortKey 排序关键字
     * @param is_desc is_desc-false升序列  is_desc-true降序 (排序字段为字符串)
     * @return
     */
    public static String jsonArraySort(JSONArray jsonArr, String sortKey, boolean is_desc) {
        JSONArray sortedJsonArray = new JSONArray();
        List<JSONObject> jsonValues = new ArrayList<JSONObject>();
        for (int i = 0; i < jsonArr.size(); i++) {
            jsonValues.add(JSONObject.parseObject(String.valueOf(jsonArr.getJSONObject(i))));
        }
        Collections.sort(jsonValues, new Comparator<JSONObject>() {
            private  final String KEY_NAME = sortKey;

            @Override
            public int compare(JSONObject a, JSONObject b) {
                String valA = new String();
                String valB = new String();
                try {
                    valA = a.getString(KEY_NAME);
                    valB = b.getString(KEY_NAME);
                } catch (JSONException e) {
                    e.printStackTrace();
                }
                if (is_desc){
                    return -valA.compareTo(valB);
                } else {
                    return -valB.compareTo(valA);
                }

            }
        });
        for (int i = 0; i < jsonArr.size(); i++) {
            sortedJsonArray.add(jsonValues.get(i));
        }
        return sortedJsonArray.toString();
    }
```

使用lambda表达式简化代码：

```java
    /**
     * 对json数组排序，
     * @param jsonArr
     * @param sortKey 排序关键字
     * @param is_desc is_desc-false升序列  is_desc-true降序 (排序字段为字符串)
     * @return
     */
    public static String jsonArraySort(JSONArray jsonArr, String sortKey, boolean is_desc) {
        JSONArray sortedJsonArray = new JSONArray();
        List<JSONObject> jsonValues = new ArrayList<>();
        jsonArr.forEach(a->jsonValues.add(JSONObject.parseObject(String.valueOf(a))));
        Collections.sort(jsonValues,(JSONObject a, JSONObject b)->{
            final String KEY_NAME = sortKey;
            String valA = a.getString(KEY_NAME);
            String valB = b.getString(KEY_NAME);
            if(is_desc){
                return -valA.compareTo(valB);
            }else {
                return valA.compareTo(valB);
            }
        });
        jsonArr.forEach(a -> sortedJsonArray.add(a));
        return sortedJsonArray.toString();
    }
```


JDK中的函数式接口
为了现有的类库能够直接使用 Lambda 表达式，Java 8 以前存在一些接口已经被标注为函数式接口的：

- java.lang.Runnable
- java.util.Comparator
- java.util.concurrent.Callable
- java.io.FileFilter
- java.security.PrivilegedAction
- java.beans.PropertyChangeListener

Java 8 中更是新增加了一个包 java.util.function，带来了常用的函数式接口：

- Function<T, R> - 函数：输入 T 输出 R
- BiFunction<T, U, R> - 函数：输入 T 和 U 输出 R 对象
- Predicate<T> - 断言/判断：输入 T 输出 boolean
- BiPredicate<T, U> - 断言/判断：输入 T 和 U 输出 boolean
- Supplier<T> - 生产者：无输入，输出 T
- Consumer<T> - 消费者：输入 T，无输出
- BiConsumer<T, U> - 消费者：输入 T 和 U 无输出
- UnaryOperator<T> - 单元运算：输入 T 输出 T
- BinaryOperator<T> - 二元运算：输入 T 和 T 输出 T

另外还对基本类型的处理增加了更加具体的函数是接口，包括：BooleanSupplier, DoubleBinaryOperator, DoubleConsumer, DoubleFunction<R>, DoublePredicate, DoubleSupplier, DoubleToIntFunction, DoubleToLongFunction, DoubleUnaryOperator, IntBinaryOperator, IntConsumer, IntFunction<R>, IntPredicate, IntSupplier, IntToDoubleFunction, IntToLongFunction, IntUnaryOperator, LongBinaryOperator, LongConsumer, LongFunction<R>, LongPredicate, LongSupplier, LongToDoubleFunction, LongToIntFunction, LongUnaryOperator, ToDoubleBiFunction<T, U>, ToDoubleFunction<T>, ToIntBiFunction<T, U>, ToIntFunction<T>, ToLongBiFunction<T, U>, ToLongFunction<T> 。结合上面的函数式接口，对这些基本类型的函数式接口通过类名就能一眼看出接口的作用


纵观上述函数式接口，其实可以分为四大类：

- 消费型（有参数，无返回值）
- 生成型（无参数，有返回值）
- 一般型（参数，返回值都有）
- 断言型（输出布尔型）
