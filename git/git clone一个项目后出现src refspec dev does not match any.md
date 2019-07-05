>今天在提交代码时git push xxx xxx 之后出现如下错误，记录一下

```shell
git push origin dev
error: src refspec dev does not match any
```
>本人之前因为新建模块错误有删过项目重新再github上clone的经历，项目在github上有两个分支，dev和master

附上git拉取过程
```shell
git clone xxx

git pull origin dev
```
以上过程都是正常的

在修改完代码后我想提交
```java
git pull origin dev

git add .

git commit -m"kevin update something"

git pull origin dev
```
上面的过程都是正常的

异常重现
```shell
git push origin dev

error: src refspec dev does not match any
```

在看了一些博客之后，了解到；

- Git Reference简写为refs，是管理本地分支的

    - 1)本地分支的Reference格式：refs/heads/<local_branch_name>
        如refs/heads/master，在保证唯一的情况下可以简写为master
    - 2)远程追踪分支的Reference格式：refs/remotes/            <remote_repository>/<remote_branch_name>

- Reference Specification简称refspec
在执行push或fetch操作时，refspec用以给出本地Ref和远程Ref之间的映射关系

所以问题应该是出在了本地分支和远程当前分支不匹配上


查看当前本地分支
```shell
git branch 
```
发现当前只有master默认分支，而我提交的是到远程的dev分支

新建并切换分支
```shell
git checkout -b dev

Switched to a new branch 'dev'

git branch

* dev
  master
```

重新提交
```shell
git pull origin dev

git add .

git commit -m"xxx"

git pull origin dev

git push origin dev
```

成功提交
