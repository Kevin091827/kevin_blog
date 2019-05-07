**对象流：**

有的时候，我们可能需要将内存中的对象持久化到硬盘下，或者将硬盘中的对象信息读到内存中，这个时候我们可能会用到流，java IO中提供了方便我们使用的流（对象流）方便我们进行对象的序列化和反序列化操作

**序列化：**

讲对象转换成一个字节序列的过程，相当于写入操作

**反序列化：**

将一个字节序列转换为对象的过程，相当于读操作

java IO 也提供了一个接口来标志是否可以序列化----Serializable接口

该接口源码里边什么内容都没有，这个接口只是一个标记接口，用来告诉我们的JVM虚拟机，实现了这个接口的对象，可以序列化，类似我们的Cloneable接口，也是标记接口
```java
public interface Serializable {
}

public interface Cloneable {
}
```
如果需要序列化操作的对象没有实现该接口，操作的时候，会出现序列化失败的错误

**serialVersionUID （序列化的版本ID）相关问题**

```java
private static final long serialVersionUID = 1L;
```
* **1.serialVersionUID如何生成？**

	* 1.一个是默认的1L，比如：private static final long 
    serialVersionUID = 1L;

	* 2.一个是根据类名、接口名、成员方法及属性等来生成一个64位的哈希字段，比如：  private static final long serialVersionUID = xxxxL;

* **2.**Java的序列化机制流程？****

	1. 在进行对象序列化时，系统会默认给每个对象生成一个serialVersionUID，系统默认生成的serialVersionUID是根据对象的类，超类，属性和方法等信息计算得到的，如果，对象信息发生变化，例如增加或者减少一个属性，增加或减少一个方法等，该serialVersionUID也会发生相应的变化，
	2. 每个序列化的对象都是采用一个序列化好来进行保存的，当序列化一个对象时，程序会检查该对象是否已经序列化过，如果没有则使用字节流中的数据构建序列化对象，并为该对象关联一个serialVersionUID，可以是系统生成也可以指定，如果对象已经序列化过，则程序直接输出该对象关联的序列号在和本地serialVersionUID进行比较，一致则反序列化输出对象
	3. java序列化机制是通过JVM在运行时判断传进来的字节流的serialVersionUID和本地相应实体类的serialVersionUID进行比较来验证版本的一致性。如果一致，则可以执行反序列化，不一致则会出现序列化版本不一致的异常

	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401162209568.png)

* **3.serialVersionUID作用？**

	维持版本一致性

 **transient关键字**

 在序列化的过程中，我们有时候并不想该对象所有的属性都参与序列化，这时候就需要用到transient关键字,如果用transient声明一个实例变量，当对象存储时，它的值不需要维持。换句话来说就是，用transient关键字标记的成员变量不参与序列化过程。
 当反序列化的时候，该属性就会输出他的默认值

```java
public class Dog implements Serializable{

	/**
	 * 序列化版本ID
	 */
	private static final long serialVersionUID = 1L;
	private String name;
	private transient int id;
	private String age;
	private String address;
	//省略get/set
	public Dog(String name, int id,String age, String address) {
		super();
		this.name = name;
		this.id = id;
		this.age = age;
		this.address = address;
	}
```
 IO 中的为我们提供了对象流用来操作对象的持久化
```java
/**
	 * 写入对象（序列化）
	 */
	public static void writeObject() {
		
		File file = new File("object.txt");
		if(!file.exists()) {
			try {
				file.createNewFile();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		try {
			OutputStream outputStream = new FileOutputStream(file);
			ObjectOutputStream objectOutputStream = new ObjectOutputStream(outputStream);
			
			Dog dog1 = new Dog("K", 12, "13","obejct");
			Dog dog2 = new Dog("o", 15, "21");
			Dog[] dogs = {dog1,dog2};
			objectOutputStream.writeObject(dog1);
			
			System.out.println("写入完毕");
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
```
```java
/**
	 * 读对象（反序列化）
	 */
	public static void readObject() {
		File file = new File("object.txt");
		if(!file.exists()) {
			try {
				file.createNewFile();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
		try {
			InputStream inputStream = new FileInputStream(file);
			ObjectInputStream objectInputStream = new ObjectInputStream(inputStream);
			//Dog[] dogs = (Dog[])objectInputStream.readObject();
			
			System.out.println("读取完毕");
			
			//for(Dog dog : dogs) {
			//	System.out.println("结果："+dog);
			//}
			Dog dog = (Dog)objectInputStream.readObject();
			System.out.println("结果："+dog);
		} catch (FileNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
```
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190401163217882.png)
 
 id属性没有参与到序列化的过程，反序列化得到的就是他的默认值，引用类型为null
 使用对象流也是非常简单，因为JAVA IO中使用装饰设计模式为我们解决了很多困难，我们只需要重新包装字节流就可以扩展对象流操作功能。


