
>BufferReader类是从字符输入流中读取文本并缓冲字符，以便有效的读取字符，数组和行

BufferReader
```java
	/**
	 * BufferReader
	 */
	public static void inBufferWriter() {
		
		File file = new File("abc.txt");
		
		try {
			Reader reader = new FileReader(file);
			
			BufferedReader bufferedReader = new BufferedReader(reader);
			
			char[] cs = new char[1];
			
			int len = -1;
			
			StringBuilder stringBuilder = new StringBuilder();
			
			while(((len =bufferedReader.read(cs))!= -1)) {
				
				stringBuilder.append(new String(cs,0,len));
			}
			
			bufferedReader.close();
			//reader.close();
			
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

bufferWriter
```java
	/**
	 * bufferWriter
	 */
	public static void outBufferWriter() {
		
		File file = new File("abc.txt");
		try(	
			
			Writer writer  = new FileWriter(file);	
				
			BufferedWriter bufferedWriter = new BufferedWriter(writer);	
		){
			bufferedWriter.write("BufferWriter测试");
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
```
测试结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190331200318601.png)

原理：

1.FileWriter底层还是FileOutputStream，通过解码器完成字节类转换为字符流
```java
public FileWriter(File file) throws IOException {
        super(new FileOutputStream(file));
    }

//通过解码器完成字节类转换为字符流
public OutputStreamWriter(OutputStream out) {
        super(out);
        try {
            se = StreamEncoder.forOutputStreamWriter(out, this, (String)null);
        } catch (UnsupportedEncodingException e) {
            throw new Error(e);
        }
    }
```
2.BufferedWriter可以通过传入字符流和指定缓冲区大小对字符流进行增强
```java
//没有传入缓冲区大小则使用默认大小
public BufferedWriter(Writer out) {
        this(out, defaultCharBufferSize);
    }
    
// 指定缓冲区大小，构造指定大小的字符数组
 public BufferedWriter(Writer out, int sz) {
        super(out);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.out = out;
        cb = new char[sz];
        nChars = sz;
        nextChar = 0;

        lineSeparator = java.security.AccessController.doPrivileged(
            new sun.security.action.GetPropertyAction("line.separator"));
    }

```
3.默认缓冲区大小是8kb
```java
 private static int defaultCharBufferSize = 8192;
```

4.read()方法源码
```java
public int read() throws IOException {
        
        //线程安全
        synchronized (lock) {
            //判断流是否关闭
            ensureOpen();
            for (;;) {
            	//使用fill()来读取字符
                if (nextChar >= nChars) {
                    fill();
                    if (nextChar >= nChars)
                        return -1;
                }
                //如果涉及跳过字节数和换行符问题
                if (skipLF) {
                    skipLF = false;
                    if (cb[nextChar] == '\n') {
                        nextChar++;
                        continue;
                    }
                }
                return cb[nextChar++];
            }
        }
    }
```
fill()方法
```java

    private static final int UNMARKED = -1;
	private char cb[];
    private int nChars, nextChar;

    private static final int INVALIDATED = -2;
    private static final int UNMARKED = -1;
    private int markedChar = UNMARKED;
    private int readAheadLimit = 0; 

private void fill() throws IOException {
        //缓冲区存储起始位置
        int dst;
        //如果true-->则现在确认缓冲区的存储开始位置是0
        if (markedChar <= UNMARKED) {
            /* No mark */
            //没有标记，缓冲区初始存储位置是0
            dst = 0;
        } else {
            /* Marked */
            //标记情况下，重新确定起始存储位置
            //delta ：要存储的字符实际长度
            int delta = nextChar - markedChar;
            //readAheadLimit：最多可存储的字符长度
            //如果delta>=readAheadLimit，则标记无效，存储起始位置从0开始
            if (delta >= readAheadLimit) {
                /* Gone past read-ahead limit: Invalidate mark */
                markedChar = INVALIDATED;
                readAheadLimit = 0;
                dst = 0;
            } else {
            	//当readAheadLimit小于缓冲区大小
                if (readAheadLimit <= cb.length) {
                    /* Shuffle in the current buffer */
                    //置于cb[]的前面，cb[]剩余的字节从in里面获取 下一部分直至读取完毕
                    System.arraycopy(cb, markedChar, cb, 0, delta);
                    markedChar = 0;
                    //起始位置是delta
                    dst = delta;
                } else {
                    /* Reallocate buffer to accommodate read-ahead limit */
                    char ncb[] = new char[readAheadLimit];
                    System.arraycopy(cb, markedChar, ncb, 0, delta);
                    cb = ncb;
                    markedChar = 0;
                    dst = delta;
                }
                nextChar = nChars = delta;
            }
        }

        int n;
        do {
        	//cb：缓冲区
            //dst：当前所在的缓冲区的字符数组下标
            //cb.length - dst：可以存入的字符
            //in.read(cb, dst, cb.length - dst)--->返回每次实际读取的字符数量
            // //调用InputStreamReader的方法，实际是调用StreamDecoder的read(char cbuf[], int offset, int length)方法   
            n = in.read(cb, dst, cb.length - dst);
        } while (n == 0);//读完--->循环结束
        if (n > 0) {
            nChars = dst + n;
            nextChar = dst;
        }
    }

```

write
```java
public void write(int c) throws IOException {
        synchronized (lock) {
            ensureOpen();
            //缓冲区满则刷新缓冲区
            if (nextChar >= nChars)
                flushBuffer();
            //否则写入缓冲区
            cb[nextChar++] = (char) c;
        }
    }
```