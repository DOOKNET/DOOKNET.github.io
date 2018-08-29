---
title: Github搭建博客教程
date: 2018-02-5 13:55:07
categories:
- HEXO
tags:
- 教程
- 搭建博客
---
# 一、前言：

关于本博客的搭建，参考了很多网上的教程，虽然说网上教程很多，但总觉得很分散，每个教程的着重点都不一样，因此我顺便整理了下在自己搭建博客过程中参考过的教程。

---

# 二、搭建步骤：

## 2.1 软件安装：

不详细讲，具体教程网上很多，nodejs和git的安装基本都默认下一步。

## 2.2 搭建本地博客

- 安装hexo

打开cmd命令（WIN+R），执行命令

> npm install hexo-cli -g

![ ](https://user-images.githubusercontent.com/29295862/36348926-8ea5e94c-14b5-11e8-91e1-93371a45b716.png)
![npm install hexo-cli -g](https://user-images.githubusercontent.com/29295862/36348927-92344536-14b5-11e8-9316-7132577650da.png)

等待自动安装完成后，输入

> hexo -v

![hexo -v](https://user-images.githubusercontent.com/29295862/36348929-94b68e18-14b5-11e8-92ab-7d21ebc32054.png)

可以检查是否安装成功，如果没问题可以继续。

- 初始化hexo

新建一个文件夹**Hexo**，用于存放博客，配置等。右键

> Git Bash Here

![git bash here](https://user-images.githubusercontent.com/29295862/36348939-1aa58ccc-14b6-11e8-9e0b-a413dc25a1ff.png)

直接定位到当前文件夹,然后执行

> hexo init

![hexo_init](https://user-images.githubusercontent.com/29295862/36348947-42fd22de-14b6-11e8-8ce8-141602c0c205.png)

初始化配置文件,可能时间比较长，耐心等待。完成后，输入

> npm install

![npm_install](https://user-images.githubusercontent.com/29295862/36348976-ffb403f2-14b6-11e8-9877-90747a73ead0.png)

配置node。接下来输入

> hexo g

![hexo_g](https://user-images.githubusercontent.com/29295862/36348979-1387fb4a-14b7-11e8-95a0-831f6522f5ec.png)

加载hexo基础html、css、js等文件。
在这完成后等于已经在本地创建了一个网页，想查看的话，输入

> hexo s

![hexo_s](https://user-images.githubusercontent.com/29295862/36348980-1efd7b58-14b7-11e8-957b-44503dc1882d.png)

这相当于开启了一个本地的服务器，会提示你拷贝url到浏览器。（Ctrl + C 是停止的意思，不是复制！）

**这里注意：不要Ctrl+C！不要Ctrl+C！不要Ctrl+C！**

把网址输入到浏览器就能访问本地网页了。

一定要先浏览网页再执行Ctrl+C结束，顺序颠倒会导致访问不了网页。

但如果提示错误**解决办法如下：**[Hexo server not working；Hexo 无法访问页面](http://laker.me/blog/2017/03/16/17_0316_hexo_sever/)

---

打开文件**Hexo/_config.yml**,添加如下代码

```
server:
  port: 5000 # or anohter number
  log: false
  ip: 0.0.0.0
  compress: false
  header: true
```

如图所示

![ ](https://user-images.githubusercontent.com/29295862/36348993-6d522d58-14b7-11e8-8c44-911212d7f582.png)

重新输入

> hexo s

端口号会变成5000。同样，先输入网址。

![localhost:5000](https://user-images.githubusercontent.com/29295862/36349001-a3dfcd76-14b7-11e8-9ee4-6489aa568883.png)

---

至此，本地博客框架搭建完成

- DIY博客主题

之前的博客界面主题是默认的，当然是要换成别的啦。说是DIY，其实就是搬运别人的主题。下面按照我的主题举例介绍如何搬运。

首先输入

> git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo

把对应的主题clone到本地**Hexo/theme/3-hexo**下

![git clone](https://user-images.githubusercontent.com/29295862/36349056-df41cf26-14b8-11e8-84cf-0d25f7c9b957.png)

这时在theme文件夹下面多了一个**3-hexo**的文件夹，这就是下载下来的主题。

修改根目录下的 **_config.yml**文件

> theme: 3-hexo

![3-hexo](https://user-images.githubusercontent.com/29295862/36349061-fc36ae94-14b8-11e8-8181-7d0774ed1508.png)

再次运行

> hexo s

![hexo s](https://user-images.githubusercontent.com/29295862/36349062-0cca4d38-14b9-11e8-8f3c-4e955e47bfc6.png)

到这里为止，主题算是搬运完了，接下来就是细节上的修改，将别人的主页变成自己的，具体修改方法参考原作者在GitHub上的说明。

## 2.3 部署到GitHub

- 配置远程仓库地址

在根目录下的 **_config.yml**文件中修改

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/用户名/用户名.github.io.git
  branch: master
```

- 其他

具体可以参考其他参考教程。

关于新建分支用于存放源文件的操作其他参考教程上也有。

# 三、使用方法：

## 3.1 编写博客

- 打开git：

右键 **Git Bash Here**，定位到当前文件夹
![git bash here](https://user-images.githubusercontent.com/29295862/36348939-1aa58ccc-14b6-11e8-9e0b-a413dc25a1ff.png)

- 新建文章：

> hexo new "文件名字"

## 3.2 部署博客

- 本地写完后，将本地源文件备份到github：

```
git add .
git commit -m "备注"
git push origin hexo
```

- 将文件转换为网页，部署到博客：

> hexo clean && hexo g && hexo d

---

# 四、参考网站：

[next主题](http://theme-next.iissnan.com/getting-started.html)
配合[简书](https://www.jianshu.com/p/cc08e3e509e0)

也可以看[Hexo置顶及排序问题](http://yelog.org/2017/02/24/hexo-top-sort/)
等主要参考如何设置主题

主要看[使用hexo+github免费搭建个人博客网站超详细教程](http://blog.csdn.net/wapchief/article/details/54602515)另外
[GitHub Pages + Hexo搭建博客](https://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)\
以及
[Hexo+Github搭建个人博客两个分支方便维护](https://haoshuai6.github.io/2016-10-28-hexo-github.html)等主要参考如何部署和维护

其他有关修改主题内容的教程：
[为 hexo NexT 添加 Gitment 评论插件](https://meesong.github.io/StaticBlog/2017/NexT+Gitment/)
和[Hexo Next 主题博客添加gitment评论功能](https://icessun.github.io/Hexo-Next-%E4%B8%BB%E9%A2%98%E5%8D%9A%E5%AE%A2%E6%B7%BB%E5%8A%A0gitment%E8%AF%84%E8%AE%BA%E5%8A%9F%E8%83%BD.html)