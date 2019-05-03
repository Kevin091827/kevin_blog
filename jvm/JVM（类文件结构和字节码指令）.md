### 一.类文件结构
#### 1.概述
各种不同的平台的虚拟机与所有平台都统一使用的程序存储格式 --- 字节码是实现平台无关性的基石

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190424235821833.png)

所以我们有必要要了解class文件的文件结构是怎样的。

#### 2.Class文件结构

class文件是一组以8位字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑的排列在class文件之中，中间没有添加任何间隔符，这使得整个class文件中存储的内容几乎全部是程序运行的必要数据，没有空隙存在，当遇到需要占用8位字节以上空间的数据项时，则会按照高为在前的方式分割成若干个8位字节存储。
每个class文件的结构如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019042500011892.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 1.魔数（magic）

- **位置**：class文件的头4个字节
- **作用**：确定该class文件是否是一个能被虚拟机接受的class文件。
- **java的魔数**：0xCAFEBABE

##### 2.版本信息
- **位置**：紧接着魔数的4个字节
- **分类**：minor_version（次版本号），major_version(主版本号)
- **作用**：确定class文件的版本格式，不同版本的jdk支持的class文件的版本格式可能会不同

##### 3.常量池
- **位置**：紧接着版本信息
- **特点**：class文件的资源仓库，占用class文件空间最大的数据项目之一，与其他数据项目关联最多的数据类							型，第一个出现的表类型的数据项目。
- **组成**：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425002534305.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425003027409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 4.访问标志
- **位置**：紧接着常量池的2个字节
- **作用**：识别类或者接口的层次的访问信息，包括：这个class是类还是接口，访问修饰符是哪个，是否抽象等等
##### 5.字段表集合
- **作用**：描述接口或者类中声明的变量，包括类级变量以及实例级变量，不包括局部变量

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425004214136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
### 二.字节码指令
#### 字节码和数据类型

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425004713447.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425004727907.png)

l：long，s：short，b:byte，f：float，d：double，a：类型

但是从表中可以看出，大部分指令都没有支持byte，short，char和boolean，编译器会在编译器或运行期将byte，short类型的数据带符号扩展为int类型数据，boolean和char则按零位扩展成int类型数据，所以大多数对于byte，short，char和boolean的操作都是基于int
#### 字节码指令
##### 加载和存储指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005221454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 运算指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005256608.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 类型转换指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005326954.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 对象创建和访问指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005345889.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

##### 操作数栈管理指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005408802.png)

##### 控制转移指令

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005359236.png)

##### 方法调用和返回指令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190425005426186.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
