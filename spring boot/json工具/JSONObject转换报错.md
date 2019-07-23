## net.sf.json.JSONException: Object is null

```shell
2019-07-23 18:44:45.503  WARN 24317 --- [nio-8082-exec-5] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Object is null; nested exception is com.fasterxml.jackson.databind.JsonMappingException: Object is null (through reference chain: com.shanyutech.mingyu.utils.AjaxResult["oData"]->net.sf.json.JSONArray[0]->net.sf.json.JSONObject["consumeId"]->net.sf.json.JSONNull["empty"])]
```

其实已经转换已经成功了，可以打印出结果，但是使用new AjaxResult()的时候jsonArray中的有些数据是null，一个name一个value，name是String，value是Object，而且没有任何关联项，就是做为值处理的。最后把输出的结果一个一个核对，才发现有的value是null，这样就报错的，把为null的值进行修改，一切ok了，这真是，唉。

这个问题的确比较坑爹，我也遇到了。 别用net.sf.json.JSONArray或JSONObject。 用com.alibaba.fastjson.JSONArray或JSONObject就行了， 完美解决。