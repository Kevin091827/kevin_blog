# web交互流程

web交互 --- 即我们平时的浏览器和服务器是怎么进行交互的呢？

大概流程是这样的：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312225102715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

客户端（浏览器）根据用户输入的地址信息（url）请求服务器，服务器在接收到用户的请求后进行处理，然后将处理结果（响应结果）展示给用户。

# 请求和相应

**请求**：客户端根据用户地址（url）将数据发送给服务器的过程。

**响应**：服务器将请求结果发送给浏览器的过程。

**这样会存在一个问题：**

>客户端也就是浏览器的版本是有很多很多的，服务器也一样会存在多版本的问题，那么在数据交互过程中怎么确保数据的一致性呢？

那么，如何实现不同版本的浏览器和不同版本的服务器之间的数据交互呢？
这就需要给浏览器和服务器之间规范一种数据交互的格式

这种格式就是http协议

# http协议是什么？

**概念**

超文本传输协议


**作用：**

规范了浏览器和服务器之间的数据交互

**特点：**

1. 简单快速：客户向服务器请求服务时，只需要传送请求方法和路径，请求方法常用的有get，post，每种方法规定了客户和服务器联系的类型不同，由于http协议简单，使得http服务器程序规模小，通信速度快

2. 灵活：http允许传输任意类型的数据对象，正在传输的类型由content-type加以标记
3. 无连接：每次连接只处理一个请求。服务器处理完客户端的请求并收到客户端的回应后就断开连接，采取这种方式可以节省传输时间，http1.1版本支持可持续连接
4. 无状态：http是无状态协议，无状态是指协议对事务处理没有记忆能力，对于后续请求如果需要用到前面的信息，则他必须重新传，这样可能导致每次连接传输的数据量增大，另一方面，在服务器不需要先前信息时，应答就比较快

**交互流程：**

1. 客户端和服务器建立连接

2. 客户端发送请求到服务器端
3. 服务器接收到请求后处理请求后将响应结果响应给客户端
4. 关闭客户端和服务器端的连接，http1.1版本后不会立即关闭

**请求格式：**

**请求头**：请求方式+请求地址+http协议版本

**请求行**：消息报头-->一般用来说明客户端要使用到的一些附加信息

**空行**：位于请求行和请求数据之间，必须有

**请求数据**：非必须

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312232126265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

我们平时百度搜索一样东西的时候，可能觉得我的请求应该就只有这一个呀，其实不是，
我们一些静态资源等等都是请求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312232512250.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

详细看一下请求头

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312233109733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

响应头部

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312234029188.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**get和post请求的区别？**

 1. get请求数据会以？的形式隔开拼接在请求头中，不安全，没有请求实体部分
http协议虽然没有规定请求数据的大小，但是浏览器对url的长度是由限制的，所以get请求不能携带大量数据

2. post请求数据会在请求实体中进行发送，在url中是看不到请求数据的，安全，适合数据量大的数据发送

**常见状态码**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312234134542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# Http协议版本比较

1. ## http1.0

>浏览器的每次请求都需要与服务器建立一个TCP连接，服务器处理完成后立即断开TCP连接（无连接），服务器不跟踪每个客户端也不记录过去的请求（无状态）。

2. ## http1.1

>HTTP/1.0中默认使用Connection: close。在HTTP/1.1中已经默认使用Connection: keep-alive，避免了连接建立和释放的开销，但服务器必须按照客户端请求的先后顺序依次回送相应的结果，以保证客户端能够区分出每次请求的响应内容。通过Content-Length字段来判断当前请求的数据是否已经全部接收。不允许同时存在两个并行的响应。

3. ## http2.0

>HTTP/2引入二进制数据帧和流的概念，其中帧对数据进行顺序标识，如下图所示，这样浏览器收到数据之后，就可以按照序列对数据进行合并，而不会出现合并后数据错乱的情况。同样是因为有了序列，服务器就可以并行的传输数据，这就是流所做的事情。

综合比较

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190312235228716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
