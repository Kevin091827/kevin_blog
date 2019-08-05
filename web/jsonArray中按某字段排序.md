在一些业务场景中，我们需要对一些json数组进行按json数组中某个字段大小排序，例如这段json

```json
{
    "name":"网站",
    "num":3,
    "sites": [
        { "name":"Google", "info":[ "Android", "Google 搜索", "Google 翻译" ] },
        { "name":"Runoob", "info":[ "菜鸟教程", "菜鸟工具", "菜鸟微信" ] },
        { "name":"Taobao", "info":[ "淘宝", "网购" ] }
    ]
}
```

现在业务要求我们根据json数组sites中的name字段排序，我们应该怎么做呢？

```java
    /**
     * 对json数组排序，
     * @param jsonArr
     * @param sortKey 排序关键字
     * @param is_desc is_desc-false升序列  is_desc-true降序 (排序字段为字符串)
     * @return
     */
    public static String jsonArraySort(JSONArray jsonArr,String sortKey,boolean is_desc) {
        //存放排序结果json数组
        JSONArray sortedJsonArray = new JSONArray();
        //用于排序的list
        List<JSONObject> jsonValues = new ArrayList<JSONObject>();
        //将参数json数组每一项取出，放入list
        for (int i = 0; i < jsonArr.size(); i++) {
            jsonValues.add(JSONObject.fromObject(jsonArr.getJSONObject(i)));
        }
        //快速排序，重写compare方法，完成按指定字段比较，完成排序
        Collections.sort(jsonValues, new Comparator<JSONObject>() {
            //排序字段
            private  final String KEY_NAME = sortKey;
            //重写compare方法
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
                //是升序还是降序
                if (is_desc){
                    return -valA.compareTo(valB);
                } else {
                    return -valB.compareTo(valA);
                }

            }
        });
        //将排序后结果放入结果jsonArray
        for (int i = 0; i < jsonArr.size(); i++) {
            sortedJsonArray.add(jsonValues.get(i));
        }
        return sortedJsonArray.toString();
    }
```
补充：

在java中有两个比较器

- java.lang.Comparable接口

    ```java
    public interface Comparable<T> {
        public int compareTo(T o);
    }
    ```

- java.util.Comparator 接口

   ```java
    public interface Comparator<T> {
        int compare(T o1, T o2);
        //省略...........
    }
   ```    

两种比较器的区别：

- java.lang.Comparable支持的是一种内比较，何为内比较：即是外对象和当前对象this的比较

    ```java
        this  <  obj   ---- 返回负数
        this  =  obj   ---- 返回 0
        this  >  obj   ---- 返回正数
    ```
- java.util.Comparator支持的是一种外比较，何为外比较，即是支持两个外对象的比较，不支持和当前对象比较

    ```java
        o1  <  o2   ---- 返回负数
        o1  =  o2   ---- 返回 0
        o1  >  o2   ---- 返回正数
    ```

    