title: lzz's reading list 02
comments: true
toc: true
date: 2015-12-07 17:48:50
tags: [reading]
categories: [他山之石]
---

<!-- more -->
> 读书和博客栏目，一个是记录自己的读书列表，一个是把看到过的好站点分享出来，同时自己也可以在空闲时浏览阅读（减少某些刷XX的时间） 后面把openresty之前学的整理下

### 读书

*  《Conceptual Blockbusting 》中文版  很短小，但是很有用的书，编程珠玑第一章最后提到的，思维和创新，审视自己的思维过程
*  《FreeRADIUS Beginner's Guide》英文版   freeradius入门的书籍，从概念和实战操作讲解了经常用到的操作，对3A认证一般解决方案，还有运营商计费架构有更多的了解
*  《图解服务器端网络架构》中文版  主要从网络模型从下往上每一层的知识简明扼要的讲解了下，非常多的配图，重点突出，一点也不乏味，比上大学那会的那边 "计算机网络" 读起来感觉好多了
*   《Linux工具快速教程》电子书  @大CC 总结的最常用的Linux知识，都是些实战经验，细枝末节说的不多，对于有经验的人补缺补漏非常不错，可以根据知识点再回忆基础知识

### 博客
* [Oyster.com Tech Blog](http://tech.oyster.com/all-posts/)  有几篇文章分享了他们在大规模使用Python 作为Object Cache中遇到问题
* [David Beazley's Blog](http://www.dabeaz.com/talks.html)  几乎年年有Pycon的演讲，非常有料的博客，Pycookbook 作者，值得细细品味
* [木书架](http://www.me115.com/)  关于编程和互联网的读书，书评，和其他的一些书籍资源等。

### Python

* [Django under the hood](http://www.djangounderthehood.com/) 会有很多干货的Django大会，关注中
* [Python write one  NoSQL](http://www.jeffknupp.com/blog/2014/09/01/what-is-a-nosql-database-learn-by-writing-one-in-python/) 分析了下kv结构的nosql，然后用py简单实现了一版 能学习到一些东西。
* [Struct in Python](https://gebloggendings.files.wordpress.com/2012/07/struct.pdf) 讲述了标准库struct模块的几个用法，以及ASCII 和二进制的优势和劣势。 对于网络开发非常重要而又简单的知识点
* [Python2 docs](https://docs.python.org/2/library/functions.html#) 重温下文档中的重要部分，准备py3的知识更新

* [UnderstandingGIL](http://www.dabeaz.com/python/UnderstandingGIL.pdf) ppt 讲解了GIL py2 py3 中的实现，和一些奇怪的现象（例如cpu密集型任务，串行更好，多进程反而更慢，多核系统多进程比单核多进程更慢）  py2 GIL的切换是基于IO和cpu指令的， py3 GIL的切换是基于时间的
* [django memory leaks](http://blog.gingerlime.com/2011/django-memory-leaks-part-i/comment-page-1/) 2篇django 内存优化经验的分享  orm优化，一定不要用debug模式，还有fcgi的一些参数调整，没有太高级的技巧


### Lua

*  [Small is Beautiful:the design of Lua](http://web.stanford.edu/class/ee380/Abstracts/100310-slides.pdf)
*  [Web development with Lua Programming Language](http://presentations2015.s3.amazonaws.com/60_presentation.pdf)

### AAA
* [Freeradius 配置多个DB](http://piak.appstack.cc/2013/06/freeradius-checking-account-on-multiple.html) freeradis 类似Apache也有虚拟主机的概念，根据不同网卡，Client，认证类型来做一些策略上的隔离，log.conf 中可以配置多个数据库，然后再不同的 `Virtual Server` 配置就可以了

### 项目

* [Kong](https://github.com/Mashape/kong)  Openresty API Server， 依赖比较多，看起来比较重
* [Gin](https://github.com/ostinelli/gin) Openresty API Server

### 设计

* [Choosing an HTTP Status Code](http://racksburg.com/choosing-an-http-status-code/) rest 怎样选择状态码和对应的语意

### 其他

* [服务端开发C++ 开发学习建议](http://www.zhihu.com/question/22608820) 知乎上的回答，感受最深就是一定不要陈腐守旧，知识和科技不断发展，对于从前的结论一定要结合情景来看，一定要抓住主干理论来学和实践，分布式计算能力一定要具备

* [快速掌握一个语言的50%](http://blog.csdn.net/myan/article/details/3144661) 上来就系统的学习 不如系统的复习。抓住主干 少关注旁枝末节 实践之后 有了感性的认识 在系统的过一过 研究下感兴趣的 基本就很深入了

* [红教主谈程序员创业](http://geek.csdn.net/news/detail/45409) 针对程序员再创业这个事情上的问题做了非常深刻的分析，自负，不善沟通，不理解他人的工作都是要慢慢改掉的啊

* [Shell scripting Tutorail](http://bash.cyberciti.biz/guide/Main_Page) 查shell脚本经常遇到这个网站，顺手收藏下，没事过一遍

### 思考收获
* 最重要的能力来自于对自己所做事情的 `深入理解` , 现在对程序设计和程序运行中的各种参数指标不够深入
* 语言和职能的标签需要淡化，对整个软件开发的把控才是能力关键所在，学习能力和经验并重






