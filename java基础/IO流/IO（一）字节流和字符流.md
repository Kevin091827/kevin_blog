**流**

* **概述**

	流是一个连续的数据对象，可以从这个对象中获取数据，也可以写入数据，流操作往往会涉及三个概念，数据源，媒介，应用程序
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330171518778.png)
	我们的流，往往就是应用程序和数据源交互的媒介


**IO流**

* **概述**

	* 1，IO流是用来处理设备与设备之间的数据传输的
	* 2，java对数据传输的操作是通过流的形式进行的
	* 3，java操作IO流的API都在IO包下

* **流分类**

	* **按照流向类型分**：

		* 输入流：设备----> 程序
		* 输出流：程序----->设备 

	* **按操作类型分**：

		* 字节流 --- 字节流可以处理任意类型的数据，计算机中数据都以字节的形式存储

		* 字符流 --- 字符流只能处理字符数据，操作简单
		* 具体分类
							
        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330170152170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

* **字节流**

	* **字节输入类继承结构图**

	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330171827916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

**字节输入流读文件**

```java
	/**
	 * 字节输入流读文件
	 */
	public static void in() {
		
		File file = new File("abc.txt");
		try {
			InputStream inputStream = new FileInputStream(file);
			
			byte[] bytes = new byte[1024];
			
			StringBuilder stringBuilder = new StringBuilder();
			
			int len;
			
			// 当返回 -1 则表示文件读完，读取到的内容在bytes中
			while(((len = inputStream.read(bytes))!= -1)) {
				
				// 防止重复读
				stringBuilder.append(new String(bytes,0,len));
			}
			// 记得关闭流
			inputStream.close();
			
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

* **字节输出流继承结构图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330171937576.png)

**字节输出流**

```java
public static void out() {
		
		File file = new File("abc.txt");
		
		try {
			// true:追加输入		false：覆盖输入
			OutputStream outputStream = new FileOutputStream(file,true);
			
			String text	= "我爱广工大！";
			
			outputStream.write(text.getBytes());
			
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

* **字符输出流继承类图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330174009508.png)

```java
public static void outWriter() {
		
		File file = new File("abc.txt");
		
		try {
			Writer writer = new FileWriter(file,true);
			
			writer.write("i love gdut!");
			
			writer.close();
			
			System.out.println("写入完毕");
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```
* **字符输入流继承类图**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330174046317.png)

```java
public static void inWriter() {
		
		File file = new File("abc.txt");
		
		try {
			Reader reader = new FileReader(file);
			
			char[] cs = new char[1];
			
			int len = -1;
			
			StringBuilder stringBuilder = new StringBuilder();
			
			while(((len = reader.read(cs))!= -1)) {
				
				stringBuilder.append(new String(cs,0,len));
			}
			
			reader.close();
			
			System.out.println(stringBuilder.toString());
			
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```
	
* **字节流和字符流的区别**：

	* 1.操作的对象不同		字节流操作是基于字节，而字符流操作是基于一个字符
	* 2.操作流程不同
		* 字节流操作时，直接就是操作文件（数据源）
		* 字符流操作时，会用到一个缓冲区（缓存区）而不是直接操作文件（数据源）本身

		![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330180044332.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
1.字节流假设我没有手动关闭流
```java
//outputStream.close();
```
输出的内容还是会输出到文件中，可见，字节流直接操作的文件对象

**原理**

```java
String text	= "我爱广工大！";
outputStream.write(text.getBytes());

public void write(byte b[]) throws IOException {
        write(b, 0, b.length);
   }

public void write(byte b[], int off, int len) throws IOException {
        //判断是否是空指针或者越界异常，传入字节数组长度为0则直接返回
        if (b == null) {
            throw new NullPointerException();
        } else if ((off < 0) || (off > b.length) || (len < 0) ||
                   ((off + len) > b.length) || ((off + len) < 0)) {
            throw new IndexOutOfBoundsException();
        } else if (len == 0) {
            return;
        }
        // 一个字节一个字节的写入，并没有涉及缓冲区，也就是一个字节一个字节的和文件交互
        for (int i = 0 ; i < len ; i++) {
            write(b[off + i]);//本地方法
        }
    }
```
2.字符流如果我不手动关闭流
```java
writer.write("i love gdut!");			
//writer.close();
```
此时输出的内容并没有写道文件中去

**原理**

```java
writer.write("i love gdut!");	

public void write(String str) throws IOException {
        write(str, 0, str.length());
    }

private char[] writeBuffer; // 暂时存放写入的内容的字符数组

private static final int WRITE_BUFFER_SIZE = 1024; // 缓存区大小

 protected Writer() {
        this.lock = this;
  }

public void write(String str, int off, int len) throws IOException {
        // 同步代码块，锁住当前对象，可见字符流的写入操作是线程安全的操作
        synchronized (lock) {
            char cbuf[];
            // 如果要写入的内容小于缓存区的默认大小，则直接构造一个指定缓存区大小的字符数组暂时存放要写入的数据
            if (len <= WRITE_BUFFER_SIZE) {
                if (writeBuffer == null) {
                    writeBuffer = new char[WRITE_BUFFER_SIZE]; 
                }
                cbuf = writeBuffer;
            }else {    // Don't permanently allocate very large buffers.
                cbuf = new char[len];
            }
            str.getChars(off, (off + len), cbuf, 0);
            write(cbuf, 0, len);
        }
    }
```
如果我们没有强制关闭字符流，那么当缓存区的大小满 WRITE_BUFFER_SIZE = 1024 时，就会自动写入文件

**意义：**

提高性能，某些情况下，如果一个程序频繁地操作一个资源（如文件或数据库），则性能会很低，此时为了提升性能，就可以将一部分数据暂时读入到内存的一块区域之中，以后直接从此区域中读取数据即可，因为读取内存速度会比较快，这样可以提升程序的性能。在字符流的操作中，所有的字符都是在内存中形成的，在输出前会将所有的内容暂时保存在内存之中，所以使用了缓冲区暂存数据。如果想在不关闭时也可以将字符流的内容全部输出，则可以使用Writer类中的flush()方法完成

跟我们使用redis做缓存的作用是很类似的

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190330235753731.png)

因为数据库其实是很脆弱的，如果网站的并发量很大，数据访问量很高，此时如果频繁的访问数据库，操作数据库，会对数据库造成很大的压力，但如果此时我们在数据库访问之前加上一层保护膜（redis）缓存一部分变动不大的数据，外界访问时，先从缓存中找，找不到在访问数据库，便可以减轻数据库的压力
* 3，字节流会出现乱码情况，字符流不会出现
	 * 字节流读取中文的问题
		* 字节流在读中文的时候有可能会读到半个中文,造成乱码 
	* 字节流写出中文的问题
		* 字节流直接操作的字节,所以写出中文必须将字符串转换成字节数组 
		* 写出回车换行 `write("\r\n".getBytes());`


* **补充**

* 1close() 和 flush()的区别

首先不关闭OutputStream
```java
public static void out() {
		
		File file = new File("abc.txt");
		try {
			// true:追加输入		false：覆盖输入
			OutputStream outputStream = new FileOutputStream(file,true);
			
			String text	= "close和flush测试OutputStream";
			
			outputStream.write(text.getBytes());
			
			//outputStream.close();
			
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

不关闭Writer
```java
public static void outWriter() {
		
		File file = new File("abc.txt");
		
		try {
			Writer writer = new FileWriter(file,true);
			
			writer.write("close和flush测试 Writer");
			
			//writer.close();
			
			System.out.println("写入完毕");
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
```
同时调用看看写入结果如何
```java
public static void main(String[] args) {
		// TODO Auto-generated method stub
		
		IOTest.out();
		try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		IOTest.in(); 
		
		IOTest.outWriter();	
		try {
			Thread.sleep(1500);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		IOTest.inWriter();
		
	}
```
结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/201903310018529.png)

1.outputstream的close()方法注释掉后，写入的内容依然输出，再次证明了，outstream直接操作的是文件（数据源），而writer的close注释掉后却没有输出，是因为要写入的内容还在缓冲区中。

当我们手动使用flush()方法

```java
writer.write("close和flush测试 Writer");			
//writer.close();
writer.flush();
```
此时结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331002449481.png)

所以，flush()作用是刷新缓存区，将缓存区的数据写出到文件，刷新后可以继续写出
close()方法是用来关闭流的，但是如果是已经实现了缓冲区的close(0方法，不仅仅会关闭流，还会刷新缓冲区，将缓冲区的数据写出到文件，这也是为什么writer注释close方法不能
写出内容，而带有close方法可以写出的原因，close方法内部也实现了flush的作用，但是关闭流后，不能再继续写出，因为已经关闭了输出流，没有了应用程序和数据源的交互媒介

2.使用 try()实现自动关流

* 流的标准处理异常代码1.6版本及其以前)资源的关闭一般在finally中
```java
public static void outWriter() {
		
		File file = new File("abc.txt");
		Writer writer = null;
		try {
			writer = new FileWriter(file);
			writer.write("close和flush测试 Writer");
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			try {
				if(writer != null)
					writer.close();
			} catch (Exception e2) {
				// TODO: handle exception
			}
		}
	}
```
* 流的标准处理异常代码1.7版本)
```java
	try(	
			//流的声明和初始化
			Writer writer  = new FileWriter(file);	
		){
			// 具体IO操作
			writer.write("close和flush测试 Writer");
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```
这样代码就可以简单很多，不会那么多finally臃肿
	 