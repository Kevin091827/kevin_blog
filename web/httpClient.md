这段时间在做项目，需要请求第三方接口获取数据并解析，用到了HttpClient，今天来总结下HttpClient的使用


## 什么是HttpClient？

Http协议可能是当今互联网上使用比较广泛，比较重要的网络协议了，在java编程里，我们有时候有些业务可能也会需要用到Http协议来访问网络资源（比如，请求第三方接口，网络爬虫等等）

>虽然在 JDK 的 java.net 包中已经提供了访问 HTTP 协议的基本功能，但是对于大部分应用程序来说，JDK 库本身提供的功能还不够丰富和灵活。HttpClient 是 Apache Jakarta Common 下的子项目，用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。

## HttpClient功能

>（1）实现了所有 HTTP 的方法（GET,POST,PUT,HEAD 等）

>（2）支持自动转向

>（3）支持 HTTPS 协议

>（4）支持代理服务器等


## 基本使用

项目中用到httpClient去请求第三方接口，所以使用上会以此为例总结

**1. get请求**

基本步骤：

1. 创建HttpClient对象，返回一个可关闭的HttpClient实例  

```java
 CloseableHttpClient httpclient = HttpClients.createDefault();
 ```
 
2. 创建uri = url+拼接参数 ，因为GET请求的参数都是拼装到URL后面进行传输的，所以这地方不能直接添加参数，需要组装好一个带参数的URI传递到HttpGet的构造方法中，构造一个带参数的GET请求。构造带参数的URI使用URIBuilder类

3. 创建http GET请求

```java
 HttpGet httpGet = new HttpGet(uri);
```

4. 执行请求获取响应

```java
 response = httpclient.execute(httpGet);
```

execute传入一个request对象，返回一个response对象，当发出HTTP请求时可能会抛出两种异常，分别是ClientProtocolException和IOException，第一种是通常是因为协议错误导致，如在构造HttpGet对象时传入的协议不对（例如不小心将"http"写成"htp"），或者服务器端返回的内容不符合HTTP协议要求等，第二种大多是因为网络原因引起的

5. 获取响应

```java
response.getEntity()
```

6. 关闭连接（无论连接是否成功）

```java

public static String doGet(String url, Map<String, String> param) {

        // 创建Httpclient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();

        String resultString = "";
        CloseableHttpResponse response = null;
        try {
            // 创建uri
            URIBuilder builder = new URIBuilder(url);
            if (param != null) {
                for (String key : param.keySet()) {
                    builder.addParameter(key, param.get(key));
                }
            }
            URI uri = builder.build();

            // 创建http GET请求
            HttpGet httpGet = new HttpGet(uri);

            // 执行请求
            response = httpclient.execute(httpGet);
            // 判断返回状态是否为200
            if (response.getStatusLine().getStatusCode() == 200) {
                resultString = EntityUtils.toString(response.getEntity(), "UTF-8");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (response != null) {
                    response.close();
                }
                httpclient.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return resultString;
    }
```

**2. post请求**

同理poet请求的流程也差不多，但是在参数拼接上有些不同

**Http请求协议格式：**

请求

请求行 --- request line  --- 用来说明请求类型，要访问的资源URL 和http版本

请求头 --- request header --- 服务器要使用的附加信息

请求体 --- request body 

**注意：**

请求行（request-line）中的URL部分必须以application/x-www-form-urlencoded方式编码。
主体数据（request-body）的编码方式由头部（headers）信息中的Content-Type指定。
主体数据（request-body）的长度由头部（headers）信息中的Content-Length指定。

例如我们请求以下网址：

```shell
http://localhost:8000/hello/index.html
```

http的头部信息如下：

```shell
GET /hello/index.html HTTP/1.1
Accept: */*
Accept-Language: zh-cn
Accept-Encoding: gzip, deflate
Host: localhost:8000
Connection: Keep-Alive
Cookie: JSESSIONID=BBBA54D519F7A320A54211F0107F5EA6
```

上述信息没有request-body部分，这是以GET方式发送的HTTP请求。如果请求中需要附加主体数据，即增加request-body部分，则必须使用POST方式发送HTTP请求
GET方式在request-line中传送数据；POST方式在request-line及request-body中均可以传送数据。

**所以：**

只有在post请求的情况下，才能在请求体中提供参数，在HttpClient中有两个类可以完成这项工作 它们分别是UrlEncodedFormEntity类与MultipartEntity类 ，因为他们都实现了HttpEntity接口

```java
 UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList);
```

NameValuePair是简单名称值对节点类型。多用于Java像url发送Post请求。在发送post请求时用该list来存放参数

```java
if (param != null) {
                List<NameValuePair> paramList = new ArrayList<>();
                for (String key : param.keySet()) {
                    paramList.add(new BasicNameValuePair(key, param.get(key)));
                }
```


```java
public static String doPost(String url, Map<String, String> param) {
        // 创建Httpclient对象
        CloseableHttpClient httpClient = HttpClients.createDefault();
        CloseableHttpResponse response = null;
        String resultString = "";
        try {
            // 创建Http Post请求
            HttpPost httpPost = new HttpPost(url);
            // 创建参数列表
            if (param != null) {
                List<NameValuePair> paramList = new ArrayList<>();
                for (String key : param.keySet()) {
                    paramList.add(new BasicNameValuePair(key, param.get(key)));
                }
                //UrlEncodedFormEntity类创建的对象可以模拟传统的HTML表单传送POST请求中的参数
                UrlEncodedFormEntity entity = new UrlEncodedFormEntity(paramList);
                httpPost.setEntity(entity);
            }
            // 执行http请求
            response = httpClient.execute(httpPost);
            resultString = EntityUtils.toString(response.getEntity(), "utf-8");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return resultString;
    }
```

**3. 发送json请求json**

有时候我们对接的接口需要的数据交互形式是json，这时，我们可以通过修改请求头部

实现

```java
 httpPost.setHeader("Content-Type","application/json");
```

当数据交互形式换成xml，同理

```java
 httpPost.setHeader("Content-Type","text/xml");
```

关于httpclient的addHeader和setHeader有几点补充：

1.同名header可以有多个， `Header[] headers = httpPost.getAllHeaders();`

2.运行时使用的是第一个， `Header getFirstHeader(String name)`

3.addHeader ：如果同名header在添加前已经存在，则追加到同名header尾部

4.setHeader  ：如果同名header在添加前已经存在，则覆盖一个同名header

在一些接口业务中，可能需要签名加密，这时很大可能会用到设置或者添加头部，需要看情况使用

```java 
public static String doPostJson(String url, String json) {
        // 创建Httpclient对象
        CloseableHttpClient httpClient = HttpClients.createDefault();
        CloseableHttpResponse response = null;
        String resultString = null;
        try {
            // 创建Http Post请求
            //HttpPost httpPost = new HttpPost(url);
            HttpPost httpPost = new HttpPost(url);
            //设置请求头
            httpPost.setHeader("Content-Type","application/json");
            httpPost.setHeader("charset", "utf-8");
            // 创建请求内容
            StringEntity entity = new StringEntity(json, "utf-8");
            httpPost.setEntity(entity);
            // 执行http请求
            response = httpClient.execute(httpPost);
            //判断状态
            if (response.getStatusLine().getStatusCode() == HttpStatus.SC_OK) {
                //ok
                resultString = EntityUtils.toString(response.getEntity(), "utf-8");
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                response.close();
            } catch (IOException e) {
                //  log.warn("无法获取响应");
                System.out.println("无法获取响应");
            }
        }
        return resultString;
    }
```

**4. 文件上传**
```java
/**
 * 上传文件
 * @param url 上传接口地址
 * @param filePath
 * @param fileName
 * @throws ClientProtocolException
 * @throws IOException
 */
public static void testPostFile(String url,String filePath,string fileName) throws ClientProtocolException, IOException {
    RequestConfig defaultRequestConfig = RequestConfig.custom().setSocketTimeout(5000).setConnectTimeout(3000)
            .setConnectionRequestTimeout(3000).build();

    CloseableHttpClient httpClient = HttpClients.custom().setDefaultRequestConfig(defaultRequestConfig).build();
    httpClient = HttpClients.createDefault();

    HttpPost post = new HttpPost(url);
    // 设置超时时间
    post.setConfig(defaultRequestConfig);

    // 传文件
    MultipartEntityBuilder builder = MultipartEntityBuilder.create();
    builder.setMode(HttpMultipartMode.BROWSER_COMPATIBLE);
    builder.addTextBody("name", "test");

    builder.addBinaryBody("file", new File(filePath), ContentType.DEFAULT_BINARY,
            fileName);
    post.setEntity(builder.build());

    ResponseHandler<String> responseHandler = new ResponseHandler<String>() {
        public String handleResponse(HttpResponse response) throws ClientProtocolException, IOException {
            int status = response.getStatusLine().getStatusCode();
            if (status >= 200 && status < 300) {
                HttpEntity entity = response.getEntity();
                return entity != null ? EntityUtils.toString(entity) : null;
            } else {
                throw new ClientProtocolException("Unexpected response status: " + status);
            }
        }
    };

    System.out.println(httpClient.execute(post, responseHandler));
}
```