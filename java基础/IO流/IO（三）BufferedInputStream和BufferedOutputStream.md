前面我们知道FileInputStream和FileOutputStream直接操作的文件，这样其实文件的读取和写入性能很低，如果是边读边写，就会很慢，也伤硬盘，我们可以使用之前提到过的缓冲区的概念，先把数据放入缓冲区，当缓冲区满了或者数据读完了，就一次性的吧缓冲区的内容输出到文件。缓冲区就是内存里的一块区域，把数据先存内存里，然后一次性写入，类似数据库的批量操作，这样效率比较高。

这时我们需要用到BufferedInputStream和BufferedOutputStream对我们的FileInputStream和FileOutputStream进行包装，使得我们的字节类也具有缓冲区

**buffer输出**

```java
public static void inBuffer() {
		
		File file = new File("abc.txt");
		try {
			InputStream inputStream = new FileInputStream(file);
			
			BufferedInputStream biStream = new BufferedInputStream(inputStream);
			
			byte[] bytes = new byte[1024];
			
			StringBuilder stringBuilder = new StringBuilder();
			
			int len;
			
			// 当返回 -1 则表示文件读完，读取到的内容在bytes中
			while(((len = inputStream.read(bytes))!= -1)) {
				
				// 防止重复读
				stringBuilder.append(new String(bytes,0,len));
			}
			
			biStream.close();
			
			//inputStream.close();
			
			System.out.println("输出结果："+stringBuilder);
			
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
```

**buffer写入**

```java
public static void outBuffer() {
		
		File file = new File("abc.txt");
		try {
			// true:追加输入		false：覆盖输入
			OutputStream outputStream = new FileOutputStream(file,true);
			
			BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(outputStream);
			
			String text	= "BufferedOutputStream";
			
			outputStream.write(text.getBytes());
			
			//outputStream.close();
			
			outputStream.close();
			
			System.out.println("写入成功！");
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```
测试
```java
		IOTest.outBuffer();
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		IOTest.inBuffer(); 
```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331021903461.png)

**原理：**

* **BufferedInputStream**

缓冲区 -- 因为操作的是字节，所以用到的是字节数组
```java
 protected volatile byte buf[];
```

默认缓冲区大小 --- 8kb
```java
private static int DEFAULT_BUFFER_SIZE = 8192;
```

最大缓冲区大小 
```java
 private static int MAX_BUFFER_SIZE = Integer.MAX_VALUE - 8;
```

**构造方法**

构造一个默认大小缓冲区
```java
public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }

 public BufferedInputStream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
```
**流的关闭**

当关闭BufferedInputStream时，InputStream也会被关闭
```java
 public void close() throws IOException {
        byte[] buffer;
        while ( (buffer = buf) != null) {
            if (bufUpdater.compareAndSet(this, buffer, null)) {
                InputStream input = in;
                in = null;
                if (input != null)
                    input.close();
                return;
            }
            // Else retry in case a new buf was CASed in fill()
        }
    }
```


* **BufferedOutputStream**

缓冲区大小
```java
 protected int count;
```

缓冲区
```java
protected byte buf[];
```

构造方法  --- 默认缓冲区大小也是8kb
```java
 public BufferedOutputStream(OutputStream out) {
        this(out, 8192);
 }
    
 public BufferedOutputStream(OutputStream out, int size) {
        super(out);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
```
**write()**

每一次输出，都会先输出到缓冲区，缓冲区大小会每次+1
```java
public synchronized void write(int b) throws IOException {
        if (count >= buf.length) {
            flushBuffer();
        }
        buf[count++] = (byte)b;
    }
```

刷新缓冲区 --- 如果手动调用这个方法会将缓冲区中数据输出到文件，缓冲区大小重新为0
```java
 private void flushBuffer() throws IOException {
        if (count > 0) {
            out.write(buf, 0, count);
            count = 0;
        }
    }

```

**关流**

也会自动帮我们把OutputStream关闭，同时刷新缓冲区
如果我们的缓冲区满了，缓冲区中的内容会自动写入到文件，之后大小变为0
或者手动调用flush()将缓冲区内容写入到文件
或者关流
```java
@SuppressWarnings("try")
    public void close() throws IOException {
        try (OutputStream ostream = out) {
            flush();
        }
    }
```
