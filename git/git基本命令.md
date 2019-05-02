1. 初始化git仓库

 ```shell
  git init 
```
这个仓库会对我们的代码进行备份文件

>如何备份？即如何放到.git文件

1. 先配置个人信息 -- 就是在git中设置当前使用的用户是谁？
	```shell
	git config --global user.name "username"
	git config --global user.email "userEmail"
	```
2. 将代码存储到仓库中
	a.打开仓库门 
	```shell
	git add ./代码文件路径  -- 暂存区
	```
	b. 把代码存到仓库中 
	```shell
	git commit -m "提交消息"  -- 工作区 
	如果没有写上提交信息，会进入vim编辑器，退出 ：q 强制退出：q！
	```
	c.关闭仓库门


查看状态  
```shell
git status
```

对git add git commit的补充
```shell
git add ./ 	-- 将当前项目路径下的所有文件统一添加到暂存区
git commit --all 	-- 将所有修改过的代码提交到工作区
```

查看提交记录
```shell
git log
git log --oneline 一行来显示一条日志
```

版本回退
1.先查看提交日志	
```shell
git log --oneline
```
2.回退版本		
```shell
git reset --hard head~回退数
```

通过版本号切换版本	
```shell
git reset --hard 版本号
```
查看每一次切换版本的记录	
```shell
git reflog
```

默认有一个主分支master

创建分支
```shell
	git branch 分支名
```
查看当前的分支	
```shell
git branch
```
切换分支	
```shell
git checkout 分支名 
```
删除分支	
```shell
git branch -d 要删除的分支名
```
合并分支	
```shell
git merge 指定要合并的分支名
```

ssh方式上传代码
```shell
ssh keygen -t加密方式 邮箱
```

解决合并冲突
1. 两个分支修改同一处地方，修改内容不一致：
	手动解决冲突，然后在手动提交
2. 本地版本和服务器上版本不一样？
	在本地提交之前先pull 在push，尽量在本地解决冲突

提交代码到github
1. 前提：在github中新建一个仓库
2. 拿到仓库的ssh链接
3. git push ssh链接 分支名
4. 可能需要用户名密码，是github的用户名密码

	
从github中拉代码
1. 初始化一个新的本地仓库 git init
2. 拉代码	git pull ssh链接 分支名







