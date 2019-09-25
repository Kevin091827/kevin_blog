# Git多人协作

> 每次项目都是使用的git，整理下多人协作时git的流程。以一个在Gitlab上的TestProject作为例子

## 首先创建远程库

在对应的Git代码管理平台上创建一个项目，比如Gitlab，就new出一个项目

![](https://dolphin-blog.oss-cn-shenzhen.aliyuncs.com/20190310135652.png)

<!-- more -->

### 如果本地没有版本库，也没有项目文件

直接使用如下命令：

```Git
git clone git@gitlab.com:dolphin/testproject.git
cd testproject
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```



### 如果本地没有版本库，但有项目文件

直接使用如下命令

```git
cd <你的项目文件夹>
git init
git remote add origin git@gitlab.com:dolphin/testproject.git
git add .
git commit -m "Initial commit"
git push -u origin master
```



### 如果本地有版本库，又有项目文件

直接使用如下命令

```
cd <你的项目文件夹>
git remote rename origin old-origin
git remote add origin git@gitlab.com:dolphin/testproject.git
git push -u origin --all
git push -u origin --tags
```



> 备注： 上面的origin是git默认生成的，可以自定义，同时ssh的git连接是按照远程库给的连接的（上面都是远程库创建时，git仓库给出的代码）



## 多人协作

> 上面贴了一些如何创建代码库的命令，下面就开始多人协作流程

![](https://dolphin-blog.oss-cn-shenzhen.aliyuncs.com/git多人协作.jpg)

上面的重点就是**每次在开始新工作时先pull远程库的代码下来，然后再工作，工作结束后再add. commit pull, push**

原因：

1. 每天早上开始新工作先pull新的下来，是为了确保自己工作在新的代码之下

2. 工作结束后add，commit，pull，push的原因是为了确保在自己工作中，别人已经push一个新的上去，工作结束时自己并不知道，所以要先pull下来，看下自己的代码在远程库是否是最新的。在这里先commit，后pull就是为了应对多人开发的情况： 

    - **commit 是为了告诉git我这次提交改了什么东西，不然你只是改了，git却不知道，也无从判断**

    - **pull是为了将本地commit与远程commit的对比记录，git 是按照文件的行数操作进行对比的,如果同时操作了某文件的同一行那么就会产生冲突,git 也会把这个冲突给标记出来,这个时候就需要先把和你冲突的那个人拉过来问问保留谁的代码,然后在 git add && git commit && git pull 这三连,再次 pull 一次是为了防止再你们协商的时候另一个人给又提交了一版东西,如果真发生了那流程重复一遍,通常没有冲突的时候就直接给你合并了,不会把你的代码给覆盖掉**



## 如果有Bug或者新功能处理......

> 上面讲解了多人协作流程，下面讲下如果有bug或者新功能开发如何处理



如果有bug或者新功能feature要处理，应该要在本地新建一个分支出来，如bug-01分支，feature-01分支

新建分支有讲究，要么在你当前工作的分支上新建分支，要么在其他干净的分支上新建分支

- **在当前工作分支上新建分支：**

1. 先把当前的修改保存起来

```shell
git stash
```

stash的是修改，是上一次commit到现在的所有更改。

2. **新建，切换bug/feature分支**

```shell
git checkout -b bug-01
或者
git checkout -b feature-01
```

3. 。。。。。。。。。。

4. **修复完bug后，add，commit**

```shell
git add .
git commit -m "fix bug"
```

5. **切换到dev分支**

```shell
git checkout dev
```

6. **合并bug/feature分支**

```shell
git merge --no-ff -m "finish tash" bug-01
```

7. 将暂存的修改释放出来

```shell
git stash pop # 将保存的修改pop出来，同时删除stash区里的修改
git stash apply # 将保存的修改拿出来，但不删除
git stash list # 查看暂存区
```



这里极有可能会产生冲突，因为pop出来的修改与你merge的可能在同一行产生冲突

8. **删除bug/feature分支**

```shell
git branch -d bug-01 # 不是强制删除，如果有修改过的没提交，不会删除
git branch -D bug-01 # 强制删除
```

