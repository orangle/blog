title: lzz的阅读记录201601
comments: true
toc: true
date: 2016-01-27 14:11:14
tags: [阅读]
categories: [他山之石]
---

<!-- more -->
> 16年新的开始，保持习惯，继续静心 踏实学习.

### 读书 & 博客
* [12Factor](http://12factor.net/zh_cn/) 主要说的是代码部署的智慧，以SasS应用作为背景，但是对于我们自己的应用部署有非常借鉴意义。
* [Brendan D. Gregg](http://www.brendangregg.com/index.html)  火焰图(Flame Graphs) 发明者 非常多的性能优化分享，包含大部分linux发行版
* [RFC 2617](https://tools.ietf.org/html/rfc2617) HTTP 基本认证和摘要认证说明
* [Python3 Patterns Recipes and Idioms](http://python-3-patterns-idioms-test.readthedocs.org/en/latest/PythonForProgrammers.html) py3 的设计模式和一些实战经验的总结


### Python

* [Simple Python Queue  with Redis](http://peter-hoffmann.com/2012/python-simple-queue-redis-queue.html) 利用Redis list fifo做简单的队列，小技巧，但是非常实用。 特别是一些系统之间的异步数据处理。
* [Django Con 2015 Europe](https://vimeo.com/channels/952478/videos) 视频集锦
* [Django Con 2015 US](https://www.youtube.com/playlist?list=PLE7tQUdRKcyaRCK5zIQFW-5XcPZOE-y9t) 视频集锦
* [Pycon 2015](https://speakerdeck.com/pycon2015)  Pycon 2015 所有幻灯片（PPT）集锦
* [template engines](http://www.simple-is-better.org/template/) 对模板引擎的理解和对py模板的一个简单汇总和测试（文章比较早）

### Lua

* [luaporwer 软件库](https://luapower.com/files/luapower/) 这里有大量的ffi 和 lua的封装，出了 lua-resty* 之外有一个学习lua的好资源
* [wrk advanced example ](http://czerasz.com/2015/07/19/wrk-http-benchmarking-tool-example/) wrk是一个非常高效实用的http基准测试工具 高级用法例子

### 前端

* [Web 性能优化-图片优化](http://www.cnblogs.com/wizcabbit/p/web-image-optimization.html) 针对我们自己的应用分析了首屏，觉得挺多地方需要优化。 图片优化是个非常重要的手段


### 系统

* [synchronous-vs-asynchronous-and-blocking-vs-non-blocking](https://www.quora.com/What-is-the-difference-between-synchronous-vs-asynchronous-and-blocking-vs-non-blocking)  其实说的比较全面

* [UNIX 高手的10个习惯](http://www.ibm.com/developerworks/cn/aix/library/au-badunixhabits.html) 有几个确实说的不错，比如不太恰当的使用grep， 变量的使用等，都是比较常用的小技巧

* [Linux 入侵检查基础](https://www.91ri.org/14906.html) 常用的审计命令和排查系统异常的一些技巧，值得学习下，提高警惕。

* [blazing-performance-with-flame-graphs](http://www.slideshare.net/brendangregg/blazing-performance-with-flame-graphs) 讲解了火焰图的使用案例，教你怎么看火焰图

* [You-can-be-a-kernel-hacker](http://jvns.ca/blog/2014/09/18/you-can-be-a-kernel-hacker/) 分享了一些作者学习内核知识的经验 ，还有一些作者的演讲和学习资料

### 数据库

* [快速判断Mysql是否要建立索引](http://mp.weixin.qq.com/s?__biz=MjM5MjIxNDA4NA==&mid=401131835&idx=1&sn=37c5fd9d3d8670fb379a1e0565e50eeb&scene=23&srcid=0108bpDNZIDkIlWVJUcVT9kl#rd)

### 项目

* [channels](https://github.com/andrewgodwin/channels) django 增加websocket支持
* [django－q](https://github.com/Koed00/django-q) django 基于多进程的任务队列 主要看看实现原理参考
* [Beanstalkd](http://kr.github.io/beanstalkd/) 用来完成延迟队列 不过celery已经有了类似的功能
* [luapower](https://luapower.com/) luajit 的一个发行版 里面有很多可以学习的lua库等等，虽然很多跟or重复了
* [LuaDist](https://github.com/LuaDist/Repository) 又一个包管理的仓库 很多lua拓展 基本上都要编译 基本没人维护了 但是很多代码可以学习的
* [wrk](https://github.com/wg/wrk)  http基准测试工具

### 随写
* 看了点django 模板引擎的源码 学习到了他们对模板引擎的设计。 通过看源码才知道模板cache的功效，看源码果然有好些好处呀。
* 看了点高性能py这本书，并做了几个例子。 真的是只有自己真的实践过才能体会到这种惊喜，一个程序从100秒优化到2秒，多实践多思考才能发现更美好的东西。
