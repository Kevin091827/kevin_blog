# 一，图是一种什么数据结构？

我们知道树是一种一对多的数据结构，同样道理，在计算机科学中，图就是一种多对多的结构

# 二，图的表示

## 图分类

### 按有无方向

* 无向图

![](https://img-blog.csdn.net/20160714185732762)

* 有向图

![](https://img-blog.csdn.net/20160714185825820)

### 按有无权值

* 有权图

![](https://img-blog.csdn.net/20160714190245727)

![](https://img-blog.csdn.net/20160714190302770)

* 无权图

![](https://img-blog.csdn.net/20160714185732762)

## 表示

### 邻接矩阵，邻接表，关联矩阵

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190522002729190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/2019052200274075.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

# 三，图的遍历

图的遍历算法是图的很多其他算法的基础，类似树的遍历算法一样同样是这种数据结构中的最重点的基础知识

## 深度优先遍历

类似于树的前序遍历，可以使用栈来辅助实现

### 思路
从图中的某一个顶点x出发，访问x，然后遍历任何一个与x相邻的未被访问的顶点y，再遍历任何一个与y相邻的未被访问的顶点z……依次类推，直到到达一个所有邻接点都被访问的顶点为止；然后，依次回退到尚有邻接点未被访问过的顶点，重复上述过程，直到图中的全部顶点都被访问过为止。

### 步骤
1. 访问起始顶点，并将其压入栈中；

2. 从栈中弹出最上面的顶点，将与其相邻的未被访问的顶点压入栈中；

3. 重复第二步，直至栈为空栈。


### 实现
```java
    /**
     * 深度优先遍历
     */
    public void dfs(){
        Stack<Integer> stack = new Stack<>();
        //访问第一个顶点，修改状态
        vertexList[0].isVisited = true;
        //获取第一个顶点名
        System.out.println(vertexList[0].getLabel());
        //入栈
        stack.push(0);
        while(!stack.isEmpty()){
            //获取该顶点邻接的未被访问的顶点
            int v = getUnvisitedVertex(stack.peek());
            if(v == -1){
                //都访问过则回退
                stack.pop();
            }else {
                //未访问过则修改访问状态，再压进栈重复该过程
                vertexList[v].isVisited = true;
                System.out.println(vertexList[v].getLabel());
                stack.push(v);
            }
        }
        // 重置，防止后边再次使用dfs
        for (int i = 0; i < nVertex; i++) {
            vertexList[i].isVisited = false;
        }

    }
```


## 广度优先遍历

类似于树的层次遍历，可以借助队列来实现


### 思路
从图中的某一个顶点x出发，访问x，然后访问与x所相邻的所有未被访问的顶点x1、x2……xn，接着再依次访问与x1、x2……xn相邻的未被访问的所有顶点。依次类推，直至图中的每个顶点都被访问。

### 步骤
1. 访问起始顶点，并将插入队列；

2. 从队列中删除队头顶点，将与其相邻的未被访问的顶点插入队列中；

3. 重复第二步，直至队列为空。

### 实现

```java
    /**
     * 广度优先遍历
     */
    public void bfs(){

        Queue<Integer> queue = new ConcurrentLinkedQueue<>();
        //访问第一个顶点，修改状态
        vertexList[0].isVisited = true;
        //获取第一个顶点名
        System.out.println(vertexList[0].getLabel());
        //入队
        queue.add(0);
        int i;
        //从队列中删除队头顶点，将与其相邻的未被访问的顶点插入队列中；
        while(!queue.isEmpty()){
            int v = queue.remove();
            while ((i = getUnvisitedVertex(v)) != -1){
                vertexList[i].isVisited = true;
                System.out.println(vertexList[i].getLabel());
                queue.add(i);
            }
        }
        // 重置，防止后边再次使用dfs
        for (int j = 0; j < nVertex; j++) {
            vertexList[j].isVisited = false;
        }
    }
```

完整代码：觉得有帮助的话，可以点点星星支持一下哈，谢谢大家

[图](https://github.com/Kevin091827/java-dataStruct/tree/master/src/com/graph/demo)