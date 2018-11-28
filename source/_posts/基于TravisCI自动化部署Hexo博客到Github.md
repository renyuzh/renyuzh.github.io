---
title: 基于TravisCI自动化部署Hexo博客到Github
date: 2018-11-28 13:10:39
tags: 
     - 效率工作
     - 自动化
categories: 前端
---
### 前言
什么是[Hexo](https://hexo.io/zh-cn/)，什么是[TravisCI](https://www.travis-ci.org/)就不介绍了，不知道的自行Google。因为自己现在的笔记本快要退休了，工作公司一台电脑，自己一台，所以就想着能不能在不同电脑上都能愉快的写自己的博文。查了一圈，[把Hexo放在阿里云上](https://qiming.info/)，每次编写好md文件再拷到服务器发布算比较好的方式了，但不是人人都是学生党能买到这么便宜的服务器，写个博客买台服务器也不划算，恰好自己最近在弄Jenkins，最后就决定采用TravisCI来自动化部署博文了。
### 本地环境安装（Windows）
Hexo依赖Node.js，另外需用Git来进行版本控制，下载对应系统的Node.js和Git一路确定安装即可~~
- [Node.js](https://nodejs.org/)
- [Git](https://git-scm.com/)

安装完成后，使用以下命令确认是否安装成功，成功会输出具体的软件版本
```
node -v
git --version
```
确认无误后，这里使用 Git bash 来执行之后的操作，不使用CMD的原因是我使用CMD的过程中出现了permission denied等一些估计和权限有关的错误。Git bash 是Windows下的命令行工具，如下所示

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-51855a0bc03e8336.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>

安装Hexo
```
npm install hexo-cli -g
```
完成后进入一个新目录存放博客资源，这里我放在D盘下
```
cd d:
hexo init blog
cd blog
npm install
hexo server
```
若一切正常，你将会看到如下输出
```
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```
打开浏览器输入地址http://localhost:4000/，你将会看到
<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-7871aa37e85ef973.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600) </div>
### 开始博客 
#### 编写博客
Hexo编写博客很简单，打开一个新的git bash，切换到blog目录，使用下面的命令可以初始化一篇博客的md文件，产生的源码位置在\blog\source\_posts下
```
hexo new post test
```
使用编辑器打开文件，适当编辑
```
---
title: test
date: 2018-11-26 22:50:36
tags:
---
### 这是测试文档
```
刷新http://localhost:4000/页面，会发现新建的测试博客已经生效
<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-938a0e7b8f8e9b19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
博客站点的一些基本信息，可以在根目录的_config.yaml中配置，如
```
title: 與誠的博客小站
subtitle: To be honest ~~~
language: zh-CN
```
***note:key和value之间有空格***，更多的使用方式详见Hexo的[官方文档](https://hexo.io/docs/)。编写博客使用的是markdown，具体的语法可以自行查询，都比较简单，简书一早也就支持了markdown语法，详细介绍见[这里](https://www.jianshu.com/p/q81RER)
#### 主题定制
Hexo有很多优秀的主题，可以在[主题页面](https://hexo.io/themes/)选择一款你钟爱的主题。这里我选择的是经典的NexT主题，因为之后的自动化部署，Hexo和主题都是即安即用的，本地的配置不再起作用，所以用自己的github账号fork一份[NexT](https://github.com/theme-next/hexo-theme-next)主题的代码，在这上面更新个人配置。按NexT的[使用文档](http://theme-next.iissnan.com)一步一步配置就可完成主题的定制，使用hexo server 重启，刷新页面，就可以看到主题生效了，所有配置完成后就可以推送自己的Github仓库中。具体配置可以参考我的[配置文件](https://github.com/renyuzh/hexo-theme-next/blob/master/_config.yml)。
***NexT由iissnan开发，现在已经交给组织，这里使用文档有个别有误的地方，官方使用文档最近还在更新中。***
### 在线部署
以上都只能在本地访问我们的博客，要想在互联网上都能访问到，需要有一个博客托管的地方，这里使用免费的Github Pages来部署博客。
#### SSH key 配置
每次操作使用账号密码比较麻烦，这里先配置ssh密钥来免密访问Github和Coding。首先配置一下用户名和邮箱。
```
git config --global user.name "your name"
git config --global user.email "youremail@example.com"
```
使用命令行生成SSH key或者Git GUI工具的帮助--》show ssh key--》generate key，产生的文件具体位置在C:\Users\用户名\.ssh下
```
ssh-keygen -t rsa
```
按提示一路回车即可。或者

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-4b9d930f61430376.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>

拷贝一份生成的id_rsa.pub公钥信息到github和coding账号中。

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-029acaaa9c452a70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>

验证ssh是否生效
```
ssh -T git@github.com
Hi renyuzh! You've successfully authenticated, but GitHub does not provide shell
 access.
```
#### 创建托管仓库
Github中创建一个仓库名为yourname.github.io的仓库

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-bafa41f9ffb95b47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>

#### 发布配置
Hexo支持 Git、Heroku、Rsync、OpenShift、FTPSync 等部署方式，这里使用Git，先安装对应的插件
```
npm install hexo-deployer-git --save
```
在博客根目录下有一个_config.yaml文件，拉到最后配置关联仓库地址
```
deploy:
  type: git
  repo: git@github.com:renyuzh/renyuzh.github.io.git
  branch: master
```
保存配置后使用命令发布
```
hexo generate
hexo deploy
或者
hexo g -d
```
第一行命令将source目录中的markdown文件转化为html文件，第二行将内容发布到配置的仓库中。本质上pages展示的是public中的内容，所以不使用hexo deploy的方式，直接将public 中的内容push到远程仓库也是一样的效果。访问[https://renyuzh.github.io](https://renyuzh.github.io)，预览最新的博客
<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-8d8ee0b9d0d82136.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
#### 自定义域名
自带的二级域名稍微难看点，我们可以自己购买想要的域名，价格几元到几万不等，这里使用在阿里云上购买的域名，登录阿里云账号，进入控制台，选择左侧域名
<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-cd4d23a5f7e6a7ff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
首先完善域名服务里的信息模板以便进行实名认证。然后选择自己购买的域名，点击解析，新建两个CNAME指向yourname.github.io地址，同时在Github的仓库根目录新建一个CNAME文件，内容是你的域名地址。等待10分钟后就可以使用我们购买的域名访问博客了。

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-798967cfe64312f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
### TravisCI
TravisCI是开源的持续集成项目，和Github的整合度很高，使用持续集成开源自动化的发布构建项目。整个自动化部署的原理很简单：
- 新建分支blog-source维护源码，一旦有push等更改操作时通知TravisCI拉取源码
- TravisCI根据分支blog-source中.travis.yml配置信息自动构建发布博客，再把内容推送到master分支上

首先使用Github账号登录[TravisCI](https://www.travis-ci.org/)官网，同步我们的Github项目，开启pages项目

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-a5c997f64b7c6710.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
要想TravisCI有操作Github的权限，必须使用Github提供的Personal access tokens，如图所示在Settings-》Developer settings下

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-5221e2807e1a7343.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
复制token信息，添加到对应的TravisCI项目中，这里只选择push时触发自动构建

<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-db3c73598b710160.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>
上面说到源码是托管在blog-source分支中的，生成的文件放在master分支中，现在任意打开一个目录，初始化git仓库，在本地新建分支，配置好.travis.yml文件即可推送到Github中
```
git init
git remote add origin git@github.com:renyuzh/renyuzh.github.io.git
git checkout -b blog-source
```
分支中，复制原blog根目录下的db.json、package.json、source目录以及.travis.yml和_config.yml配置文件，这些是必须的
```
language: node_js
node_js: stable

# assign build branches
branches:
  only:
    - blog-source

# cache this directory
cache:
  directories:
    - node_modules
    #- themes 主题没有更改时可以缓存
# S: Build Lifecycle
before_install:
  - npm install -g hexo-cli # 安装 hexo
  - git clone https://github.com/renyuzh/hexo-theme-next.git themes/next # 主题没有更改时不用每次都下载安装一遍

install:
  - npm install # 安装 package.json 中的插件

script:
  - hexo generate

after_success:
  - git config --global user.name "zhangrenyu"
  - git config --global user.email "zrysunshine@gmail.com"
  - sed -i "s/gh_token/${GH_TOKEN}/g" _config.yml #使用travisCI中配置的token替换掉_config.yml中对应的占位符
  - hexo deploy
# E: Build LifeCycle
```
同时更改_config.yaml中的发布地址
```
deploy:
  type: git
  repo: https://gh_token@github.com/renyuzh/renyuzh.github.io.git
```
将更改后的分支推送到远程分支
```
git add .
git commit -m"test TravisCI"
git push origin blog-source
```
可以看到TravisCI自动部署成功了
<div align="center">![](https://upload-images.jianshu.io/upload_images/5928994-79869b8c51b95884.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)</div>

### 参考文章
1.[基于 Hexo 的全自动博客构建部署系统](https://kchen.cc/2016/11/12/hexo-instructions/)
2.[Hexo遇上Travis-CI：可能是最通俗易懂的自动发布博客图文教程](https://blog.csdn.net/Xiong_IT/article/details/78675874)
3.[hexo搭建个人博客--NexT主题优化](https://blog.csdn.net/weixin_38796712/article/details/79372789)
4.[30分钟快速搭建hexo3.7.0 + next主题6.4教程(持续更新)](https://blog.csdn.net/marvinboy/article/details/83350437)
### 关于我
我的博客地址：[與誠的博客小站](https://zhangrenyu.com/)


