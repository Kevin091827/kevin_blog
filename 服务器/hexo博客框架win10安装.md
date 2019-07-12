Hexo是一款基于Node.js的静态博客框架，依赖少易于安装使用，可以方便的生成静态网页托管在GitHub和Coding上，是搭建博客的首选框架。大家可以进入[hexo官网](https://hexo.io/zh-cn/)进行详细查看，因为Hexo的创建者是台湾人，对中文的支持很友好，可以选择中文进行查看。

整体安装步骤：

- 安装node.js
- 安装hexo
- 配置hexo
- 发布第一篇博客
- 部署到远端github

## 一，安装node.js

1.先到官网下载安装包

[node.js](https://nodejs.org/en/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712163634583.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

2.下载完成直接安装

默认会安装到
```shell
C:\Program Files\nodejs
```
这个安装过程会把npm也相应安装完成


## 二，安装hexo

利用npm安装hexo
```shell
npm install -g hexo-cli
```
结果：
```shell
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\hexo-cli\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})
```
查看hexo版本
```shell
C:\Program Files\nodejs>hexo -v
hexo-cli: 2.0.0
os: Windows_NT 10.0.17134 win32 x64
http_parser: 2.8.0
node: 10.16.0
v8: 6.8.275.32-node.52
uv: 1.28.0
zlib: 1.2.11
brotli: 1.0.7
ares: 1.15.0
modules: 64
nghttp2: 1.34.0
napi: 4
openssl: 1.1.1b
icu: 64.2
unicode: 12.1
cldr: 35.1
tz: 2019a
```

## 三，基本配置hexo

初始化hexo，注意一定要找个空的文件夹

```shell
hexo init
```

出现以下结果可以说配置完成:
```shell
INFO  Cloning hexo-starter https://github.com/hexojs/hexo-starter.git
Cloning into 'C:\Users\12642\Desktop\hexo'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 77 (delta 4), reused 5 (delta 2), pack-reused 68
Unpacking objects: 100% (77/77), done.
Submodule 'themes/landscape' (https://github.com/hexojs/hexo-theme-landscape.git) registered for path 'themes/landscape'
Cloning into 'C:/Users/12642/Desktop/hexo/themes/landscape'...
remote: Enumerating objects: 33, done.
remote: Counting objects: 100% (33/33), done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 929 (delta 12), reused 15 (delta 3), pack-reused 896
Receiving objects: 100% (929/929), 2.56 MiB | 30.00 KiB/s, done.
Resolving deltas: 100% (492/492), done.
Submodule path 'themes/landscape': checked out '73a23c51f8487cfcd7c6deec96ccc7543960d350'
[32mINFO [39m Install dependencies
npm WARN deprecated core-js@1.2.7: core-js@<2.6.8 is no longer maintained. Please, upgrade to core-js@3 or at least to actual version of core-js@2.
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.9 (node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.9: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

added 340 packages from 500 contributors and audited 6879 packages in 550.574s
found 0 vulnerabilities

INFO  Start blogging with Hexo!
```

## 四，发布第一遍博客

生成博客，会默认生成一篇hello word.md
```shell
hexo generate
```
然后hexo会开始生成博客，生成结束后，会多出一个public的文件夹，这个文件夹就是hexo生成的一个完整的静态网站，也就是我们的博客。网站生成好了，我们要浏览它，所以要开启一下服务器，运行命令：
```shell
hexo server  //可以简写成 hexo s
```
然后打开浏览器，输入 localhost:4000 就可以浏览我们的博客了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190712164558615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTkyMjI4OQ==,size_16,color_FFFFFF,t_70)

## 五，部署到远端github

远程部署指的是，博客在我们本地生成好了以后部署到远程仓库去，如果远程仓库支持pages服务的话，那就可以通过这样的方法发布和更新博客。

要使用远程部署需要先安装hexo-deployer-git，注意，这是适用于git类型仓库的方法

```shell
npm install hexo-deployer-git --save
```

安装好hexo-deployer-git后，修改博客目录配置文件(_config.yml)中的deploy字段：

```yml
deploy:
  type: git
  repo: git仓库项目地址
  branch: 分支
  message: 自定义提交说明，这个字段可以没有
```
注意：

- hexo的git仓库名字要和github用户名一样
- 如果git仓库是ssh，则需要生成.ssh

修改完配置文件后直接部署：

```shell
hexo d
```

如果访问出现404，可以查看该博客[hexo远端部署访问404解决方案](https://blog.csdn.net/qq32933432/article/details/87955133)








