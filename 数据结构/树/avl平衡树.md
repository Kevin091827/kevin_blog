## AVL：完全平衡的二叉查找树

### 1.简介

AVL树是带有平衡条件的二叉查找树

二叉查找树可以表示动态的数据集合，对于给定的数据集合，在建立一棵二叉查找树时，二叉查找树的结构形态和关键字的插入顺序有关，如果全部或者部分的按照关键字的递增或者递减顺序插入二叉查找树的节点，则会导致所建立的二叉查找树全部或者局部退化成单支结构，最坏情况下，二叉查找树可能完全偏斜，倘若树的高度为n，则平均和最坏情况下的查找时间都是O(N) ，而最好情况是要查找的节点应该尽可能接近根节点，所以，平衡树的出现就是为了希望二叉查找树始终处于良好的结构形态

![](https://segmentfault.com/img/remote/1460000006769983)

AVL平衡树：是一种在高度上相对平衡的二叉查找树，其平均和最坏情况下的查找时间都是O(logN),同时插入和删除也会保持O(logN)，删除和插入之后的树仍然保持平衡

平衡条件：

左右子树的高度差的绝对值不超过1

### 2.节点高度

```java
    /**
     * 求当前节点高度
     * @param k
     * @return
     */
    private int hight(AvlNode k){
        return k == null ? -1 : k.hight;
    }
```


### 3.树旋转

在对二叉查找树进行插入删除之后如何保持平衡呢？答案就是通过树的旋转

树的旋转分为两种：

- 左旋
- 右旋

**左旋掕右左挂右，右旋掕左右挂左**

#### 左旋
当新插入的节点是右子树的右子节点时，需要通过左旋来保持此部分子树继续处于平衡状态

![](https://segmentfault.com/img/remote/1460000006123258)

```java
    /**
     * 单旋 --- 左旋
     * 
     * 适用情况：
     *          1.新插入节点是右子树的右子节点
     *          2.右子树的右子节点高度 >= 右子树左子节点的高度
     * @return
     */
    private AvlNode leftBalance(AvlNode root){
        //左旋掕右左挂右
        AvlNode node = root.right;
        root.right = node.left;
        node.left = root;

        //高度重置
        root.hight = Math.max(hight(root.left),hight(root.right))+1;
        node.hight = Math.max(hight(node.right),hight(root))+1;

        return node;
    }
```

#### 右旋
当新插入的节点是左子树的左子节点时，需要通过右旋来保持此部分子树继续保持平衡状态

![](https://segmentfault.com/img/remote/1460000006123262)

```java
    /**
     * 单旋 --- 右旋
     *
     * 使用情况：
     *          1.新插入节点是左子树的左子节点
     *          2.左子树的左子节点的高度 >= 左子树右子节点的高度
     *
     * @param root
     * @return
     */
    private AvlNode rightBalance(AvlNode root){
        //右旋掕左右挂左
        AvlNode node = root.left;
        root.left = node.right;
        node.right = root;

        //高度重置
        root.hight = Math.max(hight(root.left),hight(root.right))+1;
        node.hight = Math.max(hight(node.left),hight(root))+1;

        return node;
    }
```

#### 双旋
>在某些情况下，我们需要旋转两次来保持树结构的平衡

**先左旋再右旋**

当新插入的节点是左子树的右子节点时，需要通过先左旋在右旋来保持此结构的平衡

![](https://segmentfault.com/img/remote/1460000006123272)

```java
    /**
     * 双旋 --- 先左旋后右旋
     *
     * 适用情况：
     *         1.新插入节点是左子树的右子节点
     *         2.左子树左子节点高度 <= 左子树右子节点高度
     * @param root
     * @return
     */
    private AvlNode doubleLeftThenRight(AvlNode root){
        //左旋
        root.left = leftBalance(root.left);
        //右旋
        return rightBalance(root);
    }
```

**先右旋在左旋**
当新插入的节点是右子树的左子节点时，需要通过先右旋在左旋来保持树的平衡

![](https://segmentfault.com/img/remote/1460000006123285)

```java
    /**
     * 双旋 --- 先右旋后左旋
     * 
     * 适用情况：
     *          1.新插入节点是右子树的左子节点
     *          2.右子树的左子节点高度 >= 右子树右子节点高度
     * @param root
     * @return
     */
    private AvlNode doubleRightThenLeft(AvlNode root){
        //右旋
        root.right = rightBalance(root.right);
        //左旋
        return leftBalance(root);
    }
```

依靠左旋和右旋来保持树的平和结构

**平衡树结构**

```java
    /**
     * 平衡树结构
     * @param root
     */
    private AvlNode balance(AvlNode root) {
        if(root == null){
            return root;
        }
        //新节点插入的左子树
        if (hight(root.left) - hight(root.right) > 1){
            //新节点插入的左子树的左子节点
            if(hight(root.left.left) >= hight(root.left.right)){
                //右旋
                root = rightBalance(root);
            }else{
                //新节点插入的是左子树的右子节点
                //先左旋后右旋
                root = doubleLeftThenRight(root);
            }
        }
        //新节点插入的右子树
        else
        if(hight(root.right) - hight(root.left) > 1){
            //新节点插入的是右子树的右子节点
            if(hight(root.right.right) >= hight(root.right.left)){
                //左旋
                root = leftBalance(root);
            }else{
                //新节点插入的右子树的左子节点
                //先右旋后左旋
                root = doubleRightThenLeft(root);
            }
        }
        //高度重置
        root.hight = Math.max(hight(root.left),hight(root.right))+1;
        return root;
    }
```

### 4.构造平衡树
构造平衡树的方法和构造二叉查找树一样，只是多了平衡结构建立而已
```java
    /**
     * 构建平衡树
     * @param x
     * @param root
     * @return
     */
    public AvlNode insert(int x,AvlNode root){
        if(root == null){
            return new AvlNode(null,null,x);
        }
        if(x > root.val){
            root.right = insert(x,root.right);
        }else if(x < root.val){
            root.left = insert(x,root.left);
        }else{
            root.val = x;
        }
       return balance(root);
    }
```

### 5.删除平衡树中指定节点

删除节点和二叉查找树一致，但是删除后，需要恢复平衡结构

```java
    /**
     * 删除平衡树中的指定节点
     * @param x
     * @param root
     * @return
     */
    public AvlNode remove(int x,AvlNode root){
        if(root == null){
            return root;
        }
        //寻找指定节点
        if(x > root.val){
            root.right = remove(x,root.right);
        }else if(x < root.val){
            root.left = remove(x,root.left);
        }else{
            //找到了指定节点
            //判断是否存在子节点，以及是单个子节点还是两个子节点
            if(root.left != null && root.right != null){
                //左右子节点都存在
                //找出右子树中的最小值
                root.val = findMin(root.right).val;
                //转成一个子节点的情况
                root.right = remove(root.val,root.right);
            }else{
                //存在一个子节点,不空则覆盖
                if(root.left != null){
                    root = root.left;
                }
                else if(root.right != null){
                    root = root.right;
                }
                //叶节点情况
                else {
                    root = null;
                }
            }
        }
        //重新平衡
        return balance(root);
    }
```