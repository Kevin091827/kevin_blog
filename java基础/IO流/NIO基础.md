**NIO简介**

>java.nio全称java non-blocking IO，是指jdk1.4 及以上版本里提供的新api（New IO） ，为所有的原始类型（boolean类型除外）提供缓存支持的数据容器，使用它可以提供非阻塞式的高伸缩性网络。为所有的原始类型提供(Buffer)缓存支持。字符集编码解码解决方案。 Channel ：一个新的原始I/O 抽象。 支持锁和内存映射文件的文件访问接口。 提供多路(non-blocking) 非阻塞式的高伸缩性网络I/O 。

**NIO和IO的区别**

* **数据传输形式**

	* IO：面向流
	* NIO：面向缓冲区 
	* IO：输入输出流	（单向传输）
	* NIO：通道（双向传输）

* **并发性**

	* IO：阻塞IO(阻塞的 浪费性能)
	* NIO：非阻塞IO

* **NIO具有选择器---针对网络通信**

**缓冲区：**

* 底层实现：数组，用于存储不同类型的数据，java根据数据类型的不同，提供了相应的缓冲区（除了Boolean）
* 缓冲区的管理方式几乎一致
* allocate(N)获取大小是N的非直接缓冲区

```java
//获取1024字节大小的非直接缓冲区
ByteBuffer buffer = ByteBuffer.allocate(1024);
```

* allocateDirect(N)获取大小是N的直接缓冲区

```java
//获取1024字节大小的直接缓冲区
ByteBuffer bufferDirect = ByteBuffer.allocateDirect(1024);
```

**直接缓冲区**

通过allocate()Direct() （工厂方法）分配直接缓冲区，将缓冲区建立在物理内存中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403140332519.png)

**非直接缓冲区**

通过allocate()分配非直接缓冲区，将缓冲区建立在JVM内存中

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403140641717.png)

字节缓冲区分为直接缓冲区和非直接缓冲区两种

直接字节缓冲区
1. 每次调用基础操作系统的每一次本机IO操作时，都会避免像非直接缓冲区操作那样，从中间缓冲区（用户地址空间，内核地址空间）中进行复制操作，而是通过内存映射文件进行中间操作。

2. 直接缓冲区在进行缓冲区分配和取消分配所需要ed成本高于非直接缓冲区



判断是直接缓冲区还是非直接缓冲区

```java
System.out.println("是否是直接缓冲区："+bufferDirect.isDirect());
System.out.println("是否是直接缓冲区："+buffer.isDirect());
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403124803271.png)

* put()将数据放入缓冲区
```java
//向缓冲区中写入元素
String str = "hello world";
buffer.put(str.getBytes());
```

* get()获取缓冲区中的数据

```java
//切换为读模式
buffer.flip();
//想缓冲区中读取数据
byte[] dst = new byte[1024];
buffer.get(dst, 0, buffer.limit());
System.out.println("========"+new String(dst,0,buffer.limit()));
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403131231605.png)

* **四个核心属性**

	* **capacity**：容量，缓冲区中最大存储数据的容量，就是在获取缓冲区时指定的缓冲区大小，一般情况下，是不会变化的
	* **limit**：界限，缓冲区中可以操作的数据的大小，limit后的数据不可操作
	* **position**：位置，缓冲区当前操作的位置，相当于是指向当前操作元素的一个指针
	* **mark**：标记，记录当前position的位置，可以通过reset()恢复到mark标记的位置
	* **关系**：0 <= mark <= position <= limit <= capacity 	 
	* 刚获取到缓冲区时的三大属性
```java
		System.out.println("position：位置，当前在所操作缓冲区的位置"+buffer.position());
		System.out.println("limit:界限：界限后边的元素不能操作"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403132032921.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)		
		
* put()时，核心属性的变化 --- 写数据写入缓冲区
```java
		//向缓冲区中写入元素
		String str = "hello world";//10个字节==相当于10个元素
		buffer.put(str.getBytes());
		//观察写入元素后，三大属性如何变化
		System.out.println("---------观察写入元素后，三大属性如何变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403132257656.png)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403132858966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

* flip()时，核心属性的变化  -- 切换为读数据模式
```java
		//切换为读模式
		buffer.flip();
		//观察三大属性的切换后的变化
		System.out.println("---------观察切换模式后，三大属性如何变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019040313271852.png)

flip()做了什么呢？

```java
public final Buffer flip() {
        limit = position;
        position = 0;
        mark = -1;
        return this;
    }
```
此时如果读取的话，limit后面的元素是不操作的，只操作position到limit这部分，读取数据的长度即 limit - position

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403132945529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

* get()时，核心属性变化	--从缓冲区读数据
```java
		//想缓冲区中读取数据
		byte[] dst = new byte[1024];
		buffer.get(dst, 0, buffer.limit());
		System.out.println("========"+new String(dst,0,buffer.limit()));
		//观察三大属性从缓冲区读取数据后的变化
		System.out.println("---------观察三大属性从缓冲区读取数据后的变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403133550654.png)

position指针随着操作，最后指向了和limit指向相同的元素，此时不能再读取了

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019040313364718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

* rewind()时，核心属性变化 -- 可重复读取数据
调用方法后，position指针指向数组开头，limit指针不动，可读取长度：limit - position
```java
		//可重复读取数据
		buffer.rewind();
		System.out.println("---------观察三大属性变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
		byte[] dst1 = new byte[1024];
		buffer.get(dst1, 0, buffer.limit());
		System.out.println("========"+new String(dst1,0,buffer.limit()));
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403134243906.png)

```java
 public final Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }
```

* clear(0时，核心属性变化 -- 清空缓冲区，但是缓冲区中的数据仍然存在，
```java
		buffer.clear();
		System.out.println("---------观察三大属性变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403134632600.png)
此时又变回了一开始分配完缓冲区之后的样子，clear不清楚元素，只是改变指针位置
```java
    public final Buffer clear() {
        position = 0;
        limit = capacity;
        mark = -1;
        return this;
    }
```
**综上所述：**

1. capacity指针始终不变，从定义分配完成到最后，始终都是指向缓冲区数组尾部，始终等于 缓冲区容量
2. position指针经常移动，相当于一个指示下一个要操作元素位置的指示指针
3. limit指针在缓冲区写入数据前，跟capacity指针都指向数组尾部，但是写入数据后，指向写入的最后一个数据的下一个元素的位置，读的时候，不变化，相当于`new String(bytes,0,len)`中的len，limit后面的元素不可操作，是可操作元素和不可操作元素的界限
4. 通过mark()设置标记位置，当从缓冲区读取位置position位置改变后，可以通过reset()返回到之前标记的位置。
```java
		//读取数据前position位置
		System.out.println("读取数据前position："+buffer.position());
		//设置此位置为标记位置，通过reset()方法，回到该位置
		buffer.mark();
		//想缓冲区中读取数据
		byte[] dst = new byte[1024];
		buffer.get(dst, 0, buffer.limit());
		System.out.println("========"+new String(dst,0,buffer.limit()));
		//观察三大属性从缓冲区读取数据后的变化
		System.out.println("---------观察三大属性从缓冲区读取数据后的变化--------");
		System.out.println("position："+buffer.position());
		System.out.println("limit:"+buffer.limit());
		System.out.println("容量："+buffer.capacity());
		//position指针回到mark标记位置
		buffer.reset();
		System.out.println("mark标记位置position："+buffer.position());
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190403140019410.png)

5. 关系：0 <= mark <= position <= limit <= capacity 	 

**通道**：

**通道和流的区别？**

通道面向块，流面向字节

负责缓冲区的数据传输，本身不存在数据

* java.nio.Channel 接口:
	* | -- FileChannel		--- 本地操作
	* | -- SocketChannel    --- 网络套接字
	* | -- ServerSocketChannel     --- 网络套接字
	* | -- DatagramChannel    --- 针对UDP协议

**获取通道**
```java
		//打开管道，并指定相应文件地址，和操作模式
		FileChannel inFileChannel = FileChannel.open(Paths.get("caa.txt"), StandardOpenOption.READ);
		FileChannel outFileChannel = FileChannel.open(Paths.get("cba.txt"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);
		
		FileInputStream inputStream = new FileInputStream("abcdas.txt");
		FileOutputStream outputStream = new FileOutputStream("caa.txt");
		
		//开启管道
		FileChannel inChannel = inputStream.getChannel();
		FileChannel outChannel = outputStream.getChannel();
```

**管道数据传输**

**1.方式一** 
```java
public static void readAndWrite() throws Exception {
		
		FileInputStream inputStream = new FileInputStream("abcdas.txt");
		FileOutputStream outputStream = new FileOutputStream("caa.txt");
		
		//开启管道
		FileChannel inChannel = inputStream.getChannel();
		FileChannel outChannel = outputStream.getChannel();
		
		//分配缓冲区
		ByteBuffer buffer = ByteBuffer.allocate(1024*2);
		
		//传输数据
		while((inChannel.read(buffer)) != -1) {
			
			//切换为读模式
			buffer.flip();
			//将读取的数据写入管道
			outChannel.write(buffer);
			//清空缓冲区
			buffer.clear();
		}
		
		//显示关闭资源
		inChannel.close();
		outChannel.close();
		inputStream.close();
		outputStream.close();
	}
```
**2.方式二** 
```java
	public static void readAndWriteDirest() throws Exception {
		
		//打开管道，并指定相应文件地址，和操作模式
		FileChannel inFileChannel = FileChannel.open(Paths.get("caa.txt"), StandardOpenOption.READ);
		FileChannel outFileChannel = FileChannel.open(Paths.get("cba.txt"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);
		
		//内存映射文件
		MappedByteBuffer inMappedByteBuffer = inFileChannel.map(MapMode.READ_ONLY, 0, inFileChannel.size());
		MappedByteBuffer outMappedByteBuffer = outFileChannel.map(MapMode.READ_WRITE, 0, inFileChannel.size());
		
		//直接操作缓冲区
		byte[] dst = new byte[inMappedByteBuffer.limit()];
		inMappedByteBuffer.get(dst);
		outMappedByteBuffer.put(dst);
		
		//关闭资源
		inFileChannel.close();
		outFileChannel.close();
	}
```
**3.方式三** 
```java
public static void readAndWriteTrant() throws Exception {
		
		//打开管道，并指定相应文件地址，和操作模式
		FileChannel inFileChannel = FileChannel.open(Paths.get("caa.txt"), StandardOpenOption.READ);
		FileChannel outFileChannel = FileChannel.open(Paths.get("cba.txt"), StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE);
	
		//管道传输
		inFileChannel.transferTo(0, inFileChannel.size(), outFileChannel);
		//outFileChannel.transferFrom(inFileChannel, 0, inFileChannel.size());
		
		//资源关闭
		inFileChannel.close();
		outFileChannel.close();
	}
```