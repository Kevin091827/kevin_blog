### 一.浅谈前后端分离

**核心思想**

前端html页面通过ajax调用后端的restuful api接口并使用json数据进行交互。

**为什么要前后端分离？**

最重要一点就是术业有专攻，前端和后端各自负责各自擅长的领域，做出来的东西必定比通通包揽一起做要好
前端追求的是：页面表现，速度流畅，兼容性，用户体验等等。后端追求的是：三高（高并发，高可用，高性能），安全，存储，业务等等。
这篇文章解释的比较清楚，推荐一看：[为什么要前后端分离？](https://blog.csdn.net/dream_cat_forever/article/details/80709503)


### 二.样例

后端

```java
@RequestMapping("/login")
@CrossOrigin
@Slf4j
@Controller
public class LoginController {

    @RequestMapping("/index")
    public String index(){
        return "index";
    }

    @ResponseBody
    @RequestMapping("/testAjax")
    public AjaxResult testAjax(){
        return new AjaxResult().ok("hello ajax");
    }

    @ResponseBody
    @RequestMapping("/testAjax1")
    public AjaxResult testAjax1(@RequestParam("args")String args){
        return new AjaxResult().ok(args+"服务端返回");
    }
}

```

前端
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>登录</title>
    <link rel="stylesheet" th:href="@{/css/bootstrap.min.css}">
    <script th:src="@{/js/jquery.min.js}"></script>
    <script th:src="@{/js/popper.min.js}"></script>
    <script th:src="@{/js/bootstrap.min.js}"></script>
</head>
<body>

<script>
    //以上所有方法全部都可以用ajax()实现
    $.ajax({
        //发送请求地址（默认当前页地址）
        url      : String,
        //请求方式(默认get) 可选：get/post/put(部分)/delete(部分)
        type     : String,
        //延迟毫秒数
        timeout  : Number,
        //发送至服务器的数据
        data     : Object/String,
        //预期服务器返回类型 可选：xml/html/script/json/jsonp/text
        dataType : String,
        //发送前修改XMLHttpRequest对象函数
        //以下this均为本次调用Ajax传递的options参数
        beforeSend : function(XMLHttpRequest){ this; },
        //完成后的回调函数
        complete : function(XMLHttpRequest,textStatus){ this; },
        //调用成功的回调函数  data可能是html/text/..
        success : function(data,textStatus){ this; }
        //请求失败时返回的函数 通常后两个参数只有一个包含信息
        error : function(XMLHttpRequest,textStatus,errorThrown){ this;}
        //是否出发全局Ajax事件
        global: Boolean
    })
</script>

<div>
    <button id="send" class="send" onclick="123"></button>
</div>
<!--前端接收后端结果-->
<script id="123">
    $(function(){
        $('#send').click(function(){
            $.ajax({
                type: "GET",
                url: "http://localhost:8080/login/testAjax",
                //data: {username:$("#username").val(), content:$("#content").val()},
                dataType: "json",
                success: function(data){
                    alert(data["sData"]);
                },
                error:function () {
                    alert("请求失败");
                }
            });
        });
    });
</script>

<div>
    <form>
        <input id="i" name="args" type="text">
        <input type="button" onclick="111">
    </form>
</div>
<!--前端传参给后端，接收后端处理后的结果-->
<script id="111">
    $(function(){
        $('#send').click(function(){
            $.ajax({
                type: "GET",
                url: "http://localhost:8080/login/testAjax1",
                data: {args:$("#i").val()},
                dataType: "json",
                success: function(data){
                    alert(data["sData"]);
                }
            });
        });
    });
</script>
</body>
</html>
```