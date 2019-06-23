>TCP是面向连接的协议，运输连接时用来传输TCP报文的

运输连接：

- 建立连接

- 数据传输 
- 连接释放

# 一，建立连接

## 1.TCP 连接建立过程中要解决的三个问题
- (1) 要使每一方能够确知对方的存在。
- (2) 要允许双方协商一些参数（如最大窗口值、是否使用窗口扩大选项和时间戳选项以及服务质量等）。
- (3) 能够对运输实体资源（如缓存大小、连接表中的项目等）进行分配。

## 2.建立连接

TCP连接的建立采用客户服务器方式。主动发起连接建立的应用进程叫做客户(client)，
被动等待连接建立的应用进程叫做服务器(server)。


>TCP 建立连接的过程叫做握手。
>握手需要在客户和服务器之间交换三个 TCP 报文段。称之为三报文握手。
>采用三报文握手主要是为了防止已失效的连接请求报文段突然又传送到了，因而产生错误。

### 握手过程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624011923716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

## 3.释放连接

>TCP 连接释放过程比较复杂。
>数据传输结束后，通信的双方都可释放连接。
>TCP 连接释放过程是四报文握手。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624012719840.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

### 为什么需要等待2MSL？
- 第一，为了保证 A 发送的最后一个 ACK 报文段能够到达 B。
- 第二，防止 “已失效的连接请求报文段”出现在本连接中。A 在发送完最后一个 ACK 报文段后，再经过时间 2MSL，就可以使本连接持续的时间内所产生的所有报文段，都从网络中消失。这样就可以使下一个新的连接中不会出现这种旧的连接请求报文段。

## 4.连接-释放

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190624012858740.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)