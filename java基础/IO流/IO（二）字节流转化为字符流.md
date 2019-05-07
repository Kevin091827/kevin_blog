虽然说字节流可以处理任意类型的数据，但是字节流的使用不及字符流来的方便，在某些情况下，我们需要将字节流转化为字符流来简化我们的操作

**字节输入流转为字符输入流**

```java
//字节输入流转字符输入流
	public static void castIn(InputStream inputStream) {
		
		//指定字符编码
		Reader reader = new InputStreamReader(inputStream,Charset.defaultCharset());
		
		char[] cs = new char[1024];
		
		int len = -1;
		StringBuilder stringBuilder = new StringBuilder();
		try {
			while(((len = reader.read(cs))!= -1)) {
				
				stringBuilder.append(new String(cs,0,len));
			}
			System.out.println("转换输出结果："+stringBuilder);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			if(reader!=null) {
				try {
					reader.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
	
```

**字节输出流转为字符输出流**

```java
// 字节输出流转字符输出流
	public static void castOut(OutputStream outputStream) {
		
		Writer writer = new OutputStreamWriter(outputStream,Charset.defaultCharset());
		
		try {
			writer.write("gdut is my love");
			System.out.println("转换成功且写入完成");
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally {
			if(writer != null) {
				try {
					writer.close();
				} catch (IOException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
			}
		}
	}
```
**为什么能实现字节流向字符流的转换呢？**


看源码
```java
public OutputStreamWriter(OutputStream out, Charset cs) {
        super(out);
        if (cs == null)
            throw new NullPointerException("charset");
        se = StreamEncoder.forOutputStreamWriter(out, this, cs);
    }
```
StreamEncoder.forOutputStreamWriter(out, this, cs);源码
```java
 //初始化对象StreamEncoder
    public static StreamEncoder forOutputStreamWriter(OutputStream out,
            Object lock, String charsetName)
            throws UnsupportedEncodingException
    {
        String csn = charsetName;
        
        //当字符集未为null时，使用系统默认的字符集即"utf-8"
        if (csn == null)
            csn = Charset.defaultCharset().name();
        try
        {
             //检测设置的字符集是否为JAVA所支持
            if (Charset.isSupported(csn))
                return new StreamEncoder(out, lock, Charset.forName(csn)); //利用字符集名称创建字符编码器
        }
        catch (IllegalCharsetNameException x)
        {
        }
        throw new UnsupportedEncodingException(csn);
    }

```