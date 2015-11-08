title: hexo 博客使用记
comments: true
toc: true
date: 2015-11-04 15:25:04
tags: [hexo]
categories: [折腾]
---

> Github Pages用了大概一年多，之前的都是基于jekll的，但是比较懒，加上弄得主题不好看，也就没怎么写文章了，这两日看到以前的伙伴弄了个hexo主题的静态博客，于是也来尝尝鲜，学习学习，花了几个小时搭建好了。

总得来说还是很容易的，当然如果需要一些自己想要的功能还是要看看源码，修改修改才行。

20151106更新

前两天弄的博客，今天一不小心都删了，好像这是第三次删除重要的东西了，前两次都有备份，这一次就没了，都是些github上的仓库，有自己的，也有一些经典的。看了下数据恢复，想要完整的恢复基本是不可能，想想还是从新弄一遍吧。这次一定要记得把该备份的备份到远程服务器，记住，记住，记住呀。

### 粗略的流水

之前的主题都是随便找的，这次准备多找几个看看，定下来，然后自己在开发和补充。 [官方的主题库](https://hexo.io/themes/), 看来看去还是[next](https://github.com/iissnan/hexo-theme-next)主题比较合心意

#### 安装hexo和主题

nodejs, git, npm都安装好,用到hexo以及本地插件都直接安装到本文件夹，不放在全局


```
mkdir hexo_blog
cd hexo_blog

npm install -g hexo-cli
npm install hexo --save

#生成最基本的一个结构,包含一个基本的主题
hexo init

#安装基本的插件（一定要有这一步哦）
npm install

#安装next主题, 主题都是在theme下面
cd themes
git clone git@github.com:orangle/hexo-theme-next.git

#修改 hexo_blog 下面的_config.yml
theme: next
```

到这里的时候，最基本的东西就搭建好了。
一般hexo会有2个文件，一个是主文件夹的 `_config.yml` 一个是主题文件夹里的 `_config.yml`, 也就相当于全局配置和主题自定义配置。 接下来可以尝试写个文章，再本地server看看效果，然后根据next的文档改改主题的自定义的配置。


#### 常用的操作

具体的请认真阅读下官方的文档

```
#新文章，一般再主文件夹的的source目录下
hexo new 文章名称

#启动本地server
hexo s --debug

#生成网站
hexo g

#清除
hexo clean

#部署
hexo d
```

#### 继续流水

下面简单记录下，遇到的各种问题，和咋解决的，如果你也遇到了，也就不用查半天了。

* 主题修改

如果要对主题再次定制，请fork一个项目，然后即时提交更改。 防止本地文件删除了，啥都没了的悲剧

* github 访问慢

可以在gitcafe上也部署一个Pages，然后通过dns区分国内外用户访问github，或者gitcafe的Pages的服务 （gitcafe需要添加一个 `.nojekyll`的文件在source目录下）

* 文章备份

可以使用插件来解决


### 参考链接
* [hexo官网](https://hexo.io/zh-tw/docs/) 基本的操作和配置
* [next主题](https://github.com/iissnan/hexo-theme-next)
* [hexo系列教程：（五）hexo博客的优化技巧续](http://zipperary.com/2013/06/02/hexo-guide-5/) 简单的优化


