
### 简介

http协议是一个基于tcp/ip的应用层协议，是一种详细规定了浏览器和万维网(WWW = World Wide Web)服务器之间互相通信的规则，通过因特网传送万维网文档的数据传送协议。

### 工作流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801153817250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
一次HTTP操作称为一个事务，所以也称http是面向事务的协议

所以一次http请求的时间 = 2*往返rtt + 文档传输时间

**特点：**

1. 简单快速：客户向服务器请求服务时，只需要传送请求方法和路径，请求方法常用的有get，post，每种方法规定了客户和服务器联系的类型不同，由于http协议简单，使得http服务器程序规模小，通信速度快

2. 灵活：http允许传输任意类型的数据对象，正在传输的类型由content-type加以标记

3. 无连接：每次连接只处理一个请求。服务器处理完客户端的请求并收到客户端的回应后就断开连接，采取这种方式可以节省传输时间，http1.1版本支持可持续连接

4. 无状态：http是无状态协议，无状态是指协议对事务处理没有记忆能力，对于后续请求如果需要用到前面的信息，则他必须重新传，这样可能导致每次连接传输的数据量增大，另一方面，在服务器不需要先前信息时，应答就比较快

### 请求和响应

请求报文格式：

![](https://s1.51cto.com/images/20180426/1524747772856125.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

响应报文格式：

![](https://s1.51cto.com/images/20180426/1524748488887423.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

请求和响应报文的不同之处在于请求行和响应行

在请求行中比较重要的就是请求方法：

- get ：  从服务器获取资源    
- post  ：请求修改服务器资源
- put   ：向服务器上传资源
- delete  ：  删除服务器资源
- head     ： 获得服务器文档首部

在响应行中最重要的就是状态码：

- 1xx	：表示HTTP请求已经接受，继续处理请求
- 2xx	：表示HTTP请求已经处理完成
- 3xx	：表示把请求访问的URL重定向到其他目录
- 4xx	：表示客户端出现错误
- 5xx	：表示服务端出现错误

常见状态码：

- 200 OK ： 请求处理成功
- 301 MOVE PERMANENTLY ；永久跳转
- 403 FORBIDDEN ：客户端无权限访问
- 404 NOT FOUND：服务器找不到响应资源
- 500 INTERNAL SERVER ERROR : 服务器出错
- 502 BAD GATEWAY ：后端没有按照http协议返回响应
- 503 SERVICE UNAVALIBILE ：服务不可用
- 504 GATEWAY TIMEOUT : 请求超时

http响应模型：

服务器收到HTTP请求之后，会有多种方法响应这个请求，下面是HTTP响应的四种模型：

- 单进程I/O模型

    服务端开启一个进程，一个进程仅能处理一个请求，并且对请求顺序处理；

- 多进程I/O模型

    服务端并行开启多个进程，同样的一个进程只能处理一个请求，这样服务端就可以同时处理多个请求；

- 复用I/O模型

    服务端开启一个进程，但是呢，同时开启多个线程，一个线程响应一个请求，同样可以达到同时处理多个请求，线程间并发执行；

- 复用多线程I/O模型

    服务端并行开启多个进程，同时每个进程开启多个线程，这样服务端可以同时处理进程数M*每个进程的线程数N个请求。

### session和cookie

cookie原理：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801163401145.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

session原理：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019080116352992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


区别：

- session是服务器端技术，数据保存在服务器端，cookie是客户端技术，数据保存在客户端文件
- session的安全性高于cookie
- session保存在服务端，当服务端访问增多时，会比较占用性能

### http版本比较

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190801164239808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
