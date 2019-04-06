---
title: 我的Github+Hexo个人博客是怎样炼成的
date: 2016-08-26 12:39:43
categories: 技术水波文
tags:
   - hexo 
---
看到别人帅帅的博客，内心有一丢丢的羡慕。前几天恰好有时间，就查阅了一些资料，开始琢磨怎么搭建自己的blog。这一路走来，摸爬滚打，总算是搭建成功。然后自己再重新安装部署一遍，把完整步骤分享给大家。同时奉上一些传送门供参考。。。
<!--more-->

# 准备工作
在搭建博客开始之前，您必须做好以下准备工作
1. 安装了nodejs、npm，傻瓜式安装即可，此处不再赘述。
2. 安装了[git for windows](https://git-for-windows.github.io/)，git bash可打开黑窗口，并配置Git的user name和email(第一次安装Git)
3. 在GitHub上注册一个账号，并配置SSH key

**SSH key配置步骤：**
- 首先需要检查电脑上现有的ssh key
```
cd ~/.ssh
```
- 生成新的SSH key：
```
ssh-keygen -t rsa -C "1293485100@qq.com"
Enter file in which to save the key (/c/Users/CXH/.ssh/id_rsa): <回车>
```
- 输入加密串（Passphrase）：注意：输入的密码是不显示的
```
Enter passphrase (empty for no passphrase): <输入加密串>
Enter same passphrase again: <再次输入加密串>
```
- 添加SSH Key到GitHub上
在c/Users/CXH/.ssh目录下找到并打开id_rsa.pub，复制内容到GitHub的主页上的SSH Keys项，点击Add Key按钮即可。



# 安装hexo
可以参考【[Hexo的官方文档](https://hexo.io/zh-cn/docs/)】进行安装，首先输入命令
```
npm install -g hexo
```

![](/img/我的Github+hexo个人博客是怎样炼成的/1.png)
出现hexo的版本号表示安装成功。接下来，创建一个hexo文件夹，我的是在E盘下，E:\hexo，在Hexo文件下，右键运行Git Bash，进行初始化，输入命令

```
hexo init
```
初始化成功后生成的一系列文件
![](/img/我的Github+hexo个人博客是怎样炼成的/3.png)

```
输入命令本地生成并预览
hexo g  //生成
hexo s  //启动本地预览
```
打开浏览器访问【[localhost:4000](http://localhost:4000)】 即可看到内容，按Ctrl+C停止预览
![](/img/我的Github+hexo个人博客是怎样炼成的/4.png)
执行以上命令之后，hexo就会在public文件夹生成相关html文件
![](/img/我的Github+hexo个人博客是怎样炼成的/5.png) 
预览的网页效果如下
![](/img/我的Github+hexo个人博客是怎样炼成的/6.png)


# 修改主题
可以进入[hexo官网主题](https://hexo.io/themes/)选择自己喜欢的主题，我比较喜欢hexo-theme-next，【[GitHub地址](https://github.com/xinhua224/hexo-theme-next)】，clone下来
首先下载这个主题，进入hexo\themes文件夹，右键git bash

```
git clone https://github.com/xinhua224/hexo-theme-next.git
```

 
 
然后在站点配置文件中_config.yml中，进行基础配置，并将主题修改为theme: hexo-theme-next

如果您使用的是next主题，想定制自己喜欢的主题的样式，可以参考
- [NexT-使用文档](http://theme-next.iissnan.com/getting-started.html)


# 部署到Github上
## new repository
在Github上new repository，新建一个仓库
![](/img/我的Github+hexo个人博客是怎样炼成的/10.png) 
clone仓库地址
![](/img/我的Github+hexo个人博客是怎样炼成的/11.png)

## 配置deployment 
在hexo文件夹下，\_config.yml进行Deployment配置（注意仓库地址repository的写法，并不是简单复制）
![](/img/我的Github+hexo个人博客是怎样炼成的/12.png) 

## 安装hexo-deployer-git插件
该插件是git的一款自动部署发布工具。
```
npm install hexo-deployer-git --save
```

![](/img/我的Github+hexo个人博客是怎样炼成的/13.png)
## 发布到Github

```
hexo g  //生成
hexo d  //部署
```

![](/img/我的Github+hexo个人博客是怎样炼成的/14.png) 
在GitHub上刷新，public上的文件就发布上来了
![](/img/我的Github+hexo个人博客是怎样炼成的/15.png)
测试访问，在浏览器打开xinhua224.github.io
![](/img/我的Github+hexo个人博客是怎样炼成的/16.png) 



# 写博客
## 创建新文章
在git bash输入
```
hexo new 我的Github+hexo个人博客是怎样炼成的
```
![](/img/我的Github+hexo个人博客是怎样炼成的/17.png) 
执行完毕，你会发现本地E:\hexo\source\_posts目录下就生成了新添加页的md文件

在sublime中打开新生成的md文件，用markdown语法书写即可。【[Markdown 语法说明 (简体中文版)](http://www.appinn.com/markdown/)】【 [Markdown——入门指南 - 简书](http://www.jianshu.com/p/1e402922ee32/)】
写博客时，$ hexo s启动本地服务，localhost:4000，查看本地效果。

## 写完后发布更新到GitHub上
public文件夹是要部署到Git上的，首先清除

```
hexo clean 
```
重新生成输入命令

```
hexo g 
```
更新到Github，输入如下命令：
```
hexo d 
```
>小贴士：每次修改本地文件后，都要进行hexo clean-->hexo g-->hexo d三个命令后才能更新到GitHub上，hexo g和hexo d命令可以合并为一个hexo d -g，亲测hexo d -g比较好用啦！！！
 


# 重做系统后如何配置

- 重新配置SSH Key
- $ npm install -g hexo //注意：不需要命令$ hexo init
- $ npm install hexo-deployer-git --save

***此处遇到一个坑***
如果hexo d -g出现git安装完没有配置的问题，会提示Please tell me who you are，按照下图进行配置即可。
![](/img/我的Github+hexo个人博客是怎样炼成的/setup.png)




# 可以参考的传送门
- [搭建个人博客-hexo+github详细完整步骤](http://www.jianshu.com/p/189fd945f38f)
- [使用hexo+github搭建免费个人博客详细教程](http://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html)
- [手把手教你用Hexo+Github 搭建属于自己的博客](http://blog.csdn.net/gdutxiaoxu/article/details/53576018)

<- - - 本文 ღ 结束 - - - >


