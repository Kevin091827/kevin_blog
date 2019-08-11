#### 多叉树简述

##### b树

b树是最普通的一类多叉树，一个m阶的b树具有如下几个特征：

- b树中所有节点的的子节点数的最大值称为b树的阶
- 一个节点有k个孩子，那么必有k-1个关键字才能将子节点划分为k个子集
- 根结点至少有两个子女。
- 每个中间节点都包含k-1个元素和k个孩子，其中 ceil（m/2） ≤ k ≤ m
- 每一个叶子节点都包含k-1个元素，其中 ceil（m/2） ≤ k ≤ m
- 所有的叶子结点都位于同一层。
- 每个节点中的元素从小到大排列，节点当中k-1个元素正好是k个孩子包含的元素的值域划分
- 每个结点的结构为：（n，A0，K1，A1，K2，A2，…  ，Kn，An）
    其中，Ki(1≤i≤n)为关键字，且Ki<Ki+1(1≤i≤n-1)。
Ai(0≤i≤n)为指向子树根结点的指针。且Ai所指子树所有结点中的关键字均小于Ki+1。
n为结点中关键字的个数，满足ceil(m/2)-1≤n≤m-1。

例如：

三阶b树:最大子节点数量为3,节点关键字为2个

![](https://img-blog.csdn.net/2018032420113990?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**查询操作**

　　以上图为例：若查询的数值为５： 

　　第一次磁盘ＩＯ：在内存中定位（与17、35比较），比17小，左子树； 

　　第二次磁盘ＩＯ：在内存中定位（与８、12比较），比８小，左子树； 

　　第三次磁盘ＩＯ：在内存中定位（与3、5比较），找到5，终止。 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190811112208920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

整个过程中，我们可以看出：比较的次数并不比二叉查找树少，尤其适当某一节点中的数据很多时，但是磁盘IO的次数却是大大减少。比较是在内存中进行的，相比于磁盘IO的速度，比较的耗时几乎可以忽略。所以当树的高度足够低的话，就可以极大的提高效率。相比之下，节点中的元素多点也没关系，仅仅是多了几次内存交互而已，只要不超过磁盘页的大小即可。现在重点关注的就是磁盘IO的次数

**插入操作**

若被插入节点的关键字个数 < m-1 ，则直接插入

若被插入节点的关键字个数等于m-1，则会引起该节点分裂，将节点中间值插入到该节点的父节点

重复上述工作，最坏情况一直分裂到根结点，建立一个新的根结点，整个B树增加一层。

eg：插入4

![](https://img-blog.csdn.net/20180324225635106?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

会引起节点分裂，将中间值4提到父节点,若扔引起分裂，重复操作

![](https://img-blog.csdn.net/20180324230149255?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

之所以要分裂，其实就是维护多叉树平衡的一个性质，相当于平衡树中的旋转

**删除操作**
Ｂ树中关键字的删除比插入更复杂，在这里说下跟b树插入节点类似的一种删除方法:

eg:删除11

![](https://img-blog.csdn.net/20180324232514619?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

但是删除后12只有右子节点了，需要判断其左右兄弟结点中有“多余”的关键字，即：原关键字个数n>=ceil(m/2) - 1

右子节点分裂

![](https://img-blog.csdn.net/20180324232544302?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


**补充：**

- B树主要用于文件系统以及部分数据库索引，例如： MongoDB。而大部分关系数据库则使用B+树做索引，例如：mysql数据库； 
- 从查找效率考虑一般要求B树的阶数m >= 3; 
- B-树上算法的执行时间主要由读、写磁盘的次数来决定，故一次I/O操作应读写尽可能多的信息。因此B-树的结点规模一般以一个磁盘页为单位。一个结点包含的关键字及其孩子个数取决于磁盘页的大小。


##### b+树

Ｂ＋树是Ｂ树的变种，有着比Ｂ树更高的查询效率

特征：

1. 有k个子树的中间节点包含有k个元素（B树中是k-1个元素），每个元素不保存数据，只用来索引，所有数据
都保存在叶子节点。

2. 所有的叶子结点中包含了全部元素的信息，及指向含这些元素记录的指针，且叶子结点本身依关键字的大小
自小而大顺序链接。

3. 所有的中间节点元素都同时存在于子节点，在子节点元素中是最大（或最小）元素。

eg：三阶b+树

![](https://img-blog.csdn.net/20180325001555181?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**查找**

B+树通常有两个指针，一个指向根结点，另一个指向关键字最小的叶子结点。因些，对于B+树进行查找两种运算：一种是从最小关键字起顺序查找，另一种是从根结点开始，进行随机查找。

B+树相比B树的优势：

1. b+树中间节点存储的是数据在叶节点的索引，可以包含的关键字更多,单一节点存储更多的元素，使得查询的IO次数更少； 
2. 所有查询都要查找到叶子节点，查询性能稳定； 
3. 所有叶子节点形成有序链表，便于范围查询。

eg:范围查询

b树范围查询：

![](https://img-blog.csdn.net/20180325111523599?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

b+树：

![](https://img-blog.csdn.net/20180325111559207?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pfcnlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)