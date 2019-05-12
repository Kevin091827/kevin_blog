今天在提交博客时，git commit -m"***"时出现了一些问题

### 问题如下：
```git
could not open '.git/COMMIT_EDITMSG': Permission denied
```
意思大概就是无法打开’.git/COMMIT_EDITMSG’：权限被拒绝

### 解决

#### 1.原因
这不是来自远程Git存储库的错误消息，这是您的本地文件的问题
我个人是使用Windows系统（win10）所以问题出现在我可能某些时候修改了隐藏文件而不再具有对隐藏文件的写入权限

#### 2.解决
对于Windows系统可以进入.git文件（隐藏文件）删除“COMMIT_EDITMSG”文件即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513022331219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

成功解决：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190513022421118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

暂时发现这个原因，欢迎大家补充，已形成更加完善的对该错误处理的方案