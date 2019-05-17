# 一、jvm内存区域

关于jvm内存模型的具体总结，这篇博客有详细的总结：
[jvm内存区域](https://blog.csdn.net/weixin_41922289/article/details/89184550)

![](https://images2015.cnblogs.com/blog/1006828/201706/1006828-20170626190944430-2098579049.png)

我们主要关注jvm中最主要的三块内存 --- 堆，栈，方法区，而最容易最经常出现的内存错误OutOfMemoryError就很经常的很频繁的出现在这三个区域，因而，值得我们深究

## 1.堆

### 概念
java堆是jvm中所占内存最大的一块区域，是被所有线程共享的一块区域，堆也是垃圾收集器重点照顾的区域，也有gc堆之称

### 作用
所有的对象实例和数组都要在堆上分配内存

### 堆分区
根据垃圾回收的次数，堆又可分为新生代和老年代


新生代又可分为Eden空间、From Survivor空间（s0区）、To Survivor空间（s1区）so和s1区是两个大小相等且可以角色互换的区域，垃圾回收算法中的复制算法，利用的两个空间就是s0区和s1区

新生代中所占空间比，Eden：s0：s1 = 8:1:1

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513231514887.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

对象新new出来时，会存放在堆中Eden区，在一次垃圾回收后，如果对象还存活之后会进入s0区或s1区中的一个，当垃圾回收次数达到15次（jvm默认垃圾回收次数达到15次便进入老年代）进入老年代

### 堆内存OutOfMemoryError
如果在堆中内有内存完成实例分配，并且堆也无法再扩展时，则将抛出OutOfMemoryError异常


### 堆参数调优

```java

// -xx：一般是对系统（jvm）级别的设置
// 非-xx，一般是应用级别的设置
// +：启用
// -：禁用

// 格式：unit：单位 GB的“g”，MB的“m”和KB的“k”
// -Xms<heap size>[unit] 

-Xms设置堆的最小空间大小。//堆内存越大，就越不容易FullGC，越小，垃圾回收越频繁，增加垃圾回收时间，降低吞吐量

-Xmx设置堆的最大空间大小。

-XX:NewSize设置新生代最小空间大小。

-XX:MaxNewSize设置新生代最大空间大小。

-XX:PermSize设置永久代最小空间大小。

-XX:MaxPermSize设置永久代最大空间大小。

-Xss设置每个线程的堆栈大小

-XX:+UseParallelGC:选择垃圾收集器为并行收集器。此配置仅对年轻代有效。即上述配置下,年轻代使用并发收集,而年老代仍旧使用串行收集。

-XX:ParallelGCThreads=20:配置并行收集器的线程数,即:同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
```

## 2.方法区

### 概念
与java堆一样，也是线程共享的内存区域，也称为永久代（jdk1.8前），jdk1.8后
推出metaspace（本地内存）关于metaspace可以参考这篇推文 [metaspace总结](https://mp.weixin.qq.com/s?__biz=MzI4NDY5Mjc1Mg==&mid=2247483949&idx=1&sn=8b69d833bbc805e63d5b2fa7c73655f5&chksm=ebf6da52dc815344add64af6fb78fee439c8c27b539b3c0e87d8f6861c8422144d516ae0a837&scene=21#wechat_redirect)

### 作用
存储已被虚拟机加载的类信息，静态变量，常量

### 运行时常量池
运行时常量池是方法区的一部分，用于存放编译期生成的各种字面量和符号引用

### 方法区参数调优
```java
-XX:PermSize=64MB 最小尺寸，初始分配 （jdk1.8会出现警告）

-XX:MaxPermSize=256MB 最大允许分配尺寸，按需分配 （jdk1.8会出现警告）

XX:+CMSClassUnloadingEnabled -XX:+CMSPermGenSweepingEnabled 设置垃圾不回收 默认大小 

-server选项下默认MaxPermSize为64m 

-client选项下默认MaxPermSize为32m

```

## 3.栈

### java虚拟机栈

#### 概念
线程私有，生命周期和线程一样，描述java方法执行的内存模型

#### 分区
- 局部变量表

    存储方法形参和内部定义的局部变量

- 操作数栈

    存储方法执行过程中计算的中间结果

- 帧数据区

    保存访问常量池的指针



### 本地方法栈

#### 概念
本地方法栈为虚拟机执行本地方法服务

### 内存错误

- StackOverflowError

    线程请求的栈深度大于虚拟机所允许的深度

- OutOfMemoryError

    如果在栈中内有内存完成实例分配，并且栈也无法再扩展时，则将抛出OutOfMemoryError异常

## 4.处理OOM

### dump形式处理oom
JVM提供了一些参数，保证程序在内存溢出的时候能够将当前的堆信息保存在磁盘上，以至于你事后能更具这个快照信息找到问题根源：

```java
-XX:+HeapDumpOnOutOfMemoryError //表示dump堆信息到磁盘

-XX:HeapDumpPath=./java_pid<pid>.hprof //dump文件的文件路径和文件名，可以是任意的文件名，如果文件名中包含<pid>，会被替换成JVM应用的pid

-XX:OnOutOfMemoryError="< cmd args >;< cmd args >"  //是在发送内存溢出的时候执行的命令，例如：我想在内存溢出的时候重启服务器。-XX:OnOutOfMemoryError="shutdown -r" 

-XX:+UseGCOverheadLimit //设置GC消失时间的百分比限制，如果GC时间过长，超过了这个限制，那么就会触发内存溢出错误。
```
### dump概述

Heap Dump 是 Java进程所使用的内存情况在某一时间的一次快照。以文件的形式持久化到磁盘中。

Heap Dump的格式有很多种，而且不同的格式包含的信息也可能不一样。但总的来说，Heap Dump一般都包含了一个堆中的Java Objects, Class等基本信息。同时，当你在执行一个转储操作时，往往会触发一次GC，所以你转储得到的文件里包含的信息通常是有效的内容（包含比较少，或没有垃圾对象了） 。

### Heap Dump 包含的信息

* 所有的对象信息
对象的类信息、字段信息、原生值(int, long等)及引用值
* 所有的类信息
类加载器、类名、超类及静态字段
* 垃圾回收的根对象
根对象是指那些可以直接被虚拟机触及的对象
* 线程栈及局部变量
包含了转储时刻的线程调用栈信息和栈帧中的局部变量信息


### jvm参数获取dump
1. -XX:+HeapDumpOnOutOfMemoryError

    当OutOfMemoryError发生时自动生成 Heap Dump 文件。


2. -XX:+HeapDumpBeforeFullGC    

    当 JVM 执行 FullGC 前执行 dump。

3. -XX:+HeapDumpAfterFullGC

    当 JVM 执行 FullGC 后执行 dump。

4. -XX:+HeapDumpOnCtrlBreak

    交互式获取dump。在控制台按下快捷键Ctrl + Break时，JVM就会转存一下堆快照。

5. -XX:HeapDumpPath=d:\test.hprof

    指定 dump 文件存储路径。