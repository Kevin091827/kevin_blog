**git工作区域**

暂存区 暂存已经修改的文件到最后统一提交到git仓库中

工作区	--- working directory 最终确定的文件保存到仓库，成为一个新的版本，并且对他人可见

git仓库 --- git repository	---添加编辑修改文件等动作

**向仓库中添加文件流程**

```shell
git status		
git add 文件名	//将指定文件提交到暂存区
git status	
git commit -m“”提交信息“” 
git status	
```

**git安装完成之后，需要进行一些基本的信息设置**

1.设置用户名，邮箱
```shell
git config --global user.name "用户名"
git config --global user.email"邮箱"
```
2.查看设置
```shell
git config --list
```

**初始化git仓库**

1.新建文件夹
```shell
mkdir 文件夹名
```
2.在文件内初始化git仓库
```shell
git init 
```
完成后，会在新建文件夹内生成一个.git文件，刚问价是一个隐藏文件，如果看不见，则设置电脑显示隐藏文件，该文件是用来存储本地仓库信息的

**向仓库中添加文件**


1.先创建一个要添加的文件
```shell
touch gittest.txt
```
2.创建完成后查看状态
```shell
git status
```
结果显示：
```shell
$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        gittest.txt

nothing added to commit but untracked files present (use "git add" to track)

```
3.将该文件添加到暂存区
```shell
git add gittest.txt
```
4.再一次查看状态
```shell
git status
```
结果
```shell
$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   gittest.txt


```
5.将暂存区的文件添加到仓库
```shell
git commit -m "first commit"
```
结果
```shell
[master (root-commit) 12cfdd0] first commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 gittest.txt

```
这样则为提交成功

修改仓库文件

```shell
vi gittest
```
进入vim编辑器模式编辑完成后wq退出，查看文件内容
```shell
cat gittest.txt
```
结果
```shell
$ cat gittest.txt
hello world
im kevin
```
提交修改文件到暂存区
```shell
git add gittest.txt
```
查看状态后继续提交到仓库
```shell
git commit -m"second commit"
```

查看提交日志
```shell
git log
```
结果
```shell
$ git log
commit 1a966c9a2f465e7e96d4a08caf079e20a96d5bcb (HEAD -> master)
Author: kevin <1264222041@qq.com>
Date:   Sat Mar 16 15:46:27 2019 +0800

    third commit

commit 47f293513a5abef373101efe2f1a9d3c80484958
Author: kevin <1264222041@qq.com>
Date:   Sat Mar 16 15:40:15 2019 +0800

    second commit

commit 12cfdd0e66f69f556a76639fa276294e66afd982
Author: kevin <1264222041@qq.com>
Date:   Sat Mar 16 15:25:02 2019 +0800

    first commit
```

可以看到，现在头指针是指向我第三次提交，也就是最新一次提交的文件，
如果觉得 这个格式很难看，也可以使日志一次提交打印一行的方式显示出来

```shell
git log--oneline
```
结果
```shell
$ git log --oneline
1a966c9 (HEAD -> master) third commit
47f2935 second commit
12cfdd0 first commit
```
也可以这样
```shell
git reflog
```
结果
```shell
1a966c9 (HEAD -> master) HEAD@{0}: commit: third commit
47f2935 HEAD@{1}: commit: second commit
12cfdd0 HEAD@{2}: commit (initial): first commit
```

**版本回退**

版本回退的实质就是 头指针（HEAD）的移动

1.基于索引值
```shell
git reset --hard 索引值
```
我们先查看下当前第三次提交的文件内容
```shell
$ cat gittest.txt
hello world
im kevin
im so handsome
im so smart`
```
回退版本到第二次提交
```shell
$ git reset --hard 47f2935
HEAD is now at 47f2935 second commit
```
结果：查看下当前文件内容
```shell
$ cat gittest.txt
hello world
im kevin
```
查看下提交日志
```shell
$ git log --oneline
47f2935 (HEAD -> master) second commit
12cfdd0 first commit
```
现在的头指针指向了第二次提交的版本

现在我们又想回到第三次提交的版本

同样道理，只需要改变索引值
```shell
$ git reset --hard 1a966c9
HEAD is now at 1a966c9 third commit
```
现在头指针又重新指向第三次也就是我们最新提交的版本

查看当前文件内容
```shell
$ cat gittest.txt
hello world
im kevin
im so handsome
im so smart`
```
这两个版本前进和回退过程的日志信息也被记录下来了
```shell
$ git reflog
1a966c9 (HEAD -> master) HEAD@{0}: reset: moving to 1a966c9
47f2935 HEAD@{1}: reset: moving to 47f2935
1a966c9 (HEAD -> master) HEAD@{2}: commit: third commit
47f2935 HEAD@{3}: commit: second commit
12cfdd0 HEAD@{4}: commit (initial): first commit
```
2.基于^符号 --- 只可以回退
```shelll
git reset --hard head^   //一个^表示回退一个版本，两个则表示回退2个，以此类推
```
我回退两个版本到第一次提交,退出完成后并且查看一下日志

```shell
$ git reset --hard head^^
HEAD is now at 12cfdd0 first commit

12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ cat gittest.txt

12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ git log --oneline
12cfdd0 (HEAD -> master) first commit
```
3.基于~符号
```shell
git reset --hard head~2
```
~符号后边的数字表示回退的次数，

上边语句的意思是，回退两个版本，相当于
```shell
git reset --hard head^^
```
现在我回退两个版本回到第一次提交的时候
```shell
12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ git reset --hard head~2
HEAD is now at 12cfdd0 first commit

12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ git log --oneline
12cfdd0 (HEAD -> master) first commit
```
补充：

 -  --sort参数  仅仅在本地移动头指针
 - --mixed参数 在本地移动头指针，并且重置暂存区
 - --hard参数 在本地移动头指针，并且重置暂存区和工作区

**文件比较**

比较两个文件内容的差异可以使用命令
我先进入第三次提交的文件的中增加一行“”difference“”
```shell
$ vi gittest.txt
```
随后保存退出，比较差异
```shell
$ git diff gittest.txt
diff --git a/gittest.txt b/gittest.txt
index e9d73f6..262cdfc 100644
--- a/gittest.txt
+++ b/gittest.txt
@@ -2,3 +2,5 @@ hello world
 im kevin
 im so handsome
 im so smart`
+difference
```
也可以和第二次提交的版本进行比较
```shell
$ git diff head^ gittest.txt
diff --git a/gittest.txt b/gittest.txt
index 4db463f..262cdfc 100644
--- a/gittest.txt
+++ b/gittest.txt
@@ -1,3 +1,6 @@
 hello world
 im kevin
+im so handsome
+im so smart`
+difference
```

分支

创建分支
```shell
git branch dev
```
查看分支
```shell
git branch -v
```
结果显示
```shell
$ git branch -v
  dev    1a966c9 third commit
* master 1a966c9 third commit
```
但是现在是在master主分支上，我想切换到dev分支上进行开发
切换到dev分支上进行开发
```shell
git checkout dev
```
再次查看分支
```shell
$ git checkout dev
Switched to branch 'dev'
M       gittest.txt

12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (dev)
$ git branch -v
* dev    1a966c9 third commit
  master 1a966c9 third commit
```
现在已经切换到dev分支上

现在dev分支上文件和主分支上是一样，我们现在需要在dev分支修改文件然后在合并会主分支

现在在dev分支上修改文件内容，保存退出后提交到暂存区后在提交到本地库

现在dev分支上的功能开发完了，但是master分支上文件还是原来没有修改的，
我们需要将dev分支上的内容合并到master分支上 

**合并分支**

1.切换到接受修改的分支上
```shell
$ git checkout master
Switched to branch 'master'
```
```shell
12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ git merge dev
```
结果正在合并
```shell
Updating 1a966c9..aeb5ba7
Fast-forward
 gittest.txt | 3 +++
 1 file changed, 3 insertions(+)
```
现在看看文件内容
```shell
12642@LAPTOP-DPV17Q35 MINGW64 ~/Desktop/test (master)
$ cat gittest.txt
hello world
im kevin
im so handsome
im so smart`
difference
dev----modify
devawdawdawdawd
```

**操作远程仓库**

以上我们的操作操作的只是本地仓库，类型gitlab或者github这样的远程仓库上并没有发生改变

1.查看有无地址别名
```shell
git remote -v
```
2.起别名
```shell
git remote add origin http://git.shanyutech.net/kevin/gittest.git
```
别名就是origin
```shell
$ git remote -v
origin  http://git.shanyutech.net/kevin/gittest.git (fetch)  //拉取
origin  http://git.shanyutech.net/kevin/gittest.git (push)   //推送
```

之后我们便可以通过push将本地修改推送到远程库
```shell
git push origin dev
```

也可以从远程库拉取
```shell
git pull origin dev
```
git pull

拉取成功后便可以开始一天的工作
我们又修改了一些内容


**配置ssh连接远程库**

生成key
```shell
ssh-keygen -t rsa -C "1264222041@qq.com"
```
完成以下步骤后：
```shell
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/12642/.ssh/id_rsa): test
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test.
Your public key has been saved in test.pub.
The key fingerprint is:
SHA256:jImVMPeGSXG6v8fqasJ9Zl9K0DkI245cq+GnNEZAMrs 1264222041@qq.com
The key's randomart image is:

```
把公钥复制下来，粘贴到gitlab或者github指定地方

1.配置github
```shell
ssh -T git@github.com
```
结果显示：
```shell
Hi Kevin091827! You've successfully authenticated, but GitHub does not provide shell access.
```

2.配置gitlab


查看是否连接成功
```shell
ssh -T git@gitlab.com
```
结果：
```shell
Welcome to GitLab, @kevin!
```

现在配置完了，我们就可以参与协作并且将代码提交到远程库了

>1.如果本地没有版本库，也没有项目文件
```shell
git clone git@gitlab.com:dolphin/testproject.git
cd testproject
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master
```
>2.如果本地没有版本库，但有项目文件
```shell
cd <你的项目文件夹>
git init
git remote add origin git@gitlab.com:dolphin/testproject.git
git add .
git commit -m "Initial commit"
git push -u origin master
```
3.如果本地有版本库，有项目文件
```shell
cd <你的项目文件夹>
git remote rename origin old-origin
git remote add origin git@gitlab.com:dolphin/testproject.git
git push -u origin --all
git push -u origin --tags
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190319011012784.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)
上面的重点就是**每次在开始新工作时先pull远程库的代码下来，然后再工作，工作结束后再add. commit pull, push**

原因：

1. 每天早上开始新工作先pull新的下来，是为了确保自己工作在新的代码之下

2. 工作结束后add，commit，pull，push的原因是为了确保在自己工作中，别人已经push一个新的上去，工作结束时自己并不知道，所以要先pull下来，看下自己的代码在远程库是否是最新的。在这里先commit，后pull就是为了应对多人开发的情况： 

    - **commit 是为了告诉git我这次提交改了什么东西，不然你只是改了，git却不知道，也无从判断**

    - **pull是为了将本地commit与远程commit的对比记录，git 是按照文件的行数操作进行对比的,如果同时操作了某文件的同一行那么就会产生冲突,git 也会把这个冲突给标记出来,这个时候就需要先把和你冲突的那个人拉过来问问保留谁的代码,然后在 git add && git commit && git pull 这三连,再次 pull 一次是为了防止再你们协商的时候另一个人给又提交了一版东西,如果真发生了那流程重复一遍,通常没有冲突的时候就直接给你合并了,不会把你的代码给覆盖掉 **



## 如果有Bug或者新功能处理......

> 上面讲解了多人协作流程，下面讲下如果有bug或者新功能开发如何处理



如果有bug或者新功能feature要处理，应该要在本地新建一个分支出来，如bug-01分支，feature-01分支

新建分支有讲究，要么在你当前工作的分支上新建分支，要么在其他干净的分支上新建分