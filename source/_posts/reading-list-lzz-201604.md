title: lzz's reading list 05
comments: true
toc: true
date: 2016-04-23 08:57:26
tags: [reading]
categories:
---

<!-- more -->

### 书单 && 博客

* [C语言编程透视](https://github.com/tinyclub/open-c-book) 开源电子书，主要说说 `Hello World` 背后发生的故事
* [高性能网络编程](http://blog.csdn.net/column/details/high-perf-network.html)  陶辉大神网络编程的系列分享
* [跟我一起写Makefile](http://how-to-write-makefile.readthedocs.org/zh_CN/latest/index.html) MakeFile 最广为流传的教程之一


* [一线码农 博客](http://www.cnblogs.com/huangxincheng/)携程的一个同行，很多基础和实用的分享，算法，redis，mongodb，还有WIN相关的开发经验。


### Python

* [Python 面试15问](https://www.codementor.io/python/tutorial/essential-python-interview-questions) 比较经典的面试问题
* [How fast can we make interpreted Python?](https://www.reddit.com/r/Python/comments/4fdi6j/how_fast_can_we_make_interpreted_python/) reddit上的一个关于Python解释器性能相关的讨论，CPython2，Cpython3，Pypy还是Falcon，或者其他的呢？
* [Faster CPython](http://faster-cpython.readthedocs.org/cpython36.html) cpython36 解释器相关的内容，各种优化的方案以及测试等信息, 对Python解释器关注的可以看看。



### C

* [如何学好C语言](http://coolshell.cn/articles/4102.html) 皓哥的建议和心得，老文章，多拿来看看. 那么多年都还是看了C就头晕，这一次要克服这个坎。语言、算法和数据结构、系统调用和设计，其实还是基础不够，所以那些上层建筑也是空中楼阁呀，深入问几句就不知所以然了。要有个长期的心里准备，别被其他的小狐狸吸引走，健全的知识体系是要专注才能慢慢建立起来的。




### 数据库

* [一步一步进行数据库设计](http://www.cnblogs.com/DBFocus/archive/2011/10/12/2208580.html) 博客园上的一系列数据库设计的经验分享，针对系统开发中数据设计进行的经验总结，包括案例的思考设计过程，类似的文章看到的比较少，值得学习。
* [剖析Force.com的多租户架构](http://dbanotes.net/arch/salesforce_multitenancy_intro.html)  讲解了saleforce多租户架构的设计以及实现方式的探讨，对于需要做这方面需求的有个很直观的认识


### 开源项目

* [500lines](https://github.com/aosabook/500lines) 这本书和《开源框架》是一个网站出的，里面讲的的是一些常用的web的或者是开发的组件，几百行的代码来实现一个核心功能的过程，非常值得一看。
* [像IDE一样使用vim](https://github.com/yangyangwithgnu/use_vim_as_ide) 作者基于众多的实用插件打造自己的 `C/C++` vim ide，依赖插件比较多，效果还是挺好的
* [命令行的艺术](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md) github上start超过25000，一篇简短的文章包含丰富的shell编程知识，仔细读一读，能收获不少。


### 架构*经验

* [见微知著-技术分享](http://www.jikexueyuan.com/course/2658.html)  雨痕的经验分享，大牛的实战思考
* [Multi Tenant Saas using MySQL5](http://web.archive.org/web/20120211232457/http://www.reachcrm.com/2010/03/11/multi-tenant-strategy-for-saas-using-mysql5)  一个实例一个DB来实现SAAS中的多租户，这位作者讲述了他做一个SAAS系统的经验。 tenant id对终端用户和租户均不可知，base表和应用表权限分离，把租户信息和应用信息存在不同的数据库。 他这里会给每个租户创建不同的MySQL用户，每个租户的用户连接数据库时候会使用当前租户的MySQL配置，通过一个定制的视图（视图和base的区别就是增加了租户ID的过滤条件）来获取用户的数据。主要是通过MySQL和`USER( )`来建立自动数据过滤视图，web层的开发就可以从无数的 `where tid=xxx` 中解脱出来。 YII 框架的wiki上有个php版本的[简单实现](http://www.yiiframework.com/wiki/603/a-multi-tenant-strategy-using-yii-and-mysql/)


### 生与活

* 对于某些不切实际的东西过于偏执，特别是技术的东西，方向上有问题。 太贪多，缺乏从一而终，从潜入深的过程，其他的能力的自我要求太低，更多的从非技术层面去认识和理解人事。

* 之前我每天都花好多时间在技术群，看的多聊的少，每天看很多技术博客，泛读很多，到现在我才发现，很多时间都是毫无收获的，看完一堆东西之后，我问自己收获到了什么，很多时候都是啥也没有记住，也没有印象深刻的新知识呀。 原来之前那么久效率那么低，怪不得总是觉得事倍功半。还是踏踏实实把读书计划落实了，每天进步一点点就足够了呀。



