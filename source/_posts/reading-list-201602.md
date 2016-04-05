title: lzz's reading list 04
comments: true
toc: true
date: 2016-04-05 14:51:08
tags: [reading]
categories: [他山之石]
---

<!-- more -->
>  2月过年，回家的时间基本没有怎么阅读，所以两个月合在一起吧，顺便改了个标题名字，嘿嘿。


### 读书

*  Unix编程艺术（中文版）  读了一半
* [How to Think Like a Computer Scientist](http://interactivepython.org/runestone/static/thinkcspy/index.html)  刚开始读
* [Problem Solving with Algorithms and Data Structures](http://interactivepython.org/runestone/static/pythonds/index.html)  刚开始读
* [伯克利cs61A的教材](http://inst.eecs.berkeley.edu/~cs61a/sp12/book/index.html)    未读, 看起来内容还不错
* 《一万小时天才理论 》 之前经常看到别人引用，这次才真正看到书。 1w小时其实是作者根据别人成功所需要的练习时间推算出来的，这个并不是书的重点。 书中提出最主要的成功因素是 `精深练习`，也就是说对一门技术类似的知识，需要深度的练习，同时也是讲究方式方法的，还有一些情感，品质，环境的因素。 用自己话总结下就是：能坐住冷板凳，带有激情的研究知识的细节，建立更高层次的知识体系，实践实践实践。
* 《汇编语言》 王爽老师的，随便翻翻，没有细细研读，后面有计划认真看一看



### Python

* [Python epoll how to](http://scotdoyle.com/python-epoll-howto.html) py中怎样使用epoll来写socket程序，包括边缘触发和水平触发的例子
* [benoitc 的ppt](https://speakerdeck.com/benoitc) benoitc是Gunicorn的创建者，这里有很多他的演讲ppt，如果你想研究Gunicorn，有必要看看
* [Finally, Real-Time Django Is Here: Get Started with Django Channels](https://blog.heroku.com/archives/2016/3/17/in_deep_with_django_channels_the_future_of_real_time_apps_in_django) 介绍了Django realtime 应用的一个新的方案 django-channels, 这可能是Django的未来哦，毕竟gevent 等方案太麻烦
* [JSON-RPC2.0 规范中文版](http://wiki.geekdream.com/Specification/json-rpc_2.0.html) jsonrpc 定义非常简洁和轻量，rpc类型的服务设计需要读一读
* [Building a higher-level query API: the right way to use django's ORM](https://www.dabapps.com/blog/higher-level-query-api-django-orm/)  这是个比较早的文章，作者用一种hack的方式，对django的model进行拓展，自定义Manager等，非常有先见。 文章通过循序渐进的方式，讲述了怎样写一个健壮的model。


### Redis

* [Redis开发运维实战指南](https://www.gitbook.com/book/gnuhpc/redis-all-about/details)
* [Redis 设计与实现](http://redisbook.com/index.html)


### 安全

* [Python 渗透测试工具集合](http://www.freebuf.com/tools/94777.html)  列举了非常多的py相关的渗透工具，在需要的时候可以随时查找
* [给开发者的终极XSS防护备忘录](http://blog.knownsec.com/wp-content/uploads/2014/07/%E7%BB%99%E5%BC%80%E5%8F%91%E8%80%85%E7%9A%84%E7%BB%88%E6%9E%81XSS%E9%98%B2%E6%8A%A4%E5%A4%87%E5%BF%98%E5%BD%95.pdf)  知道创宇翻译，web开发从业者需要认真读一读，说实话有点专业，看起来也挺费功夫的。 最好去乌云上找找相关漏洞的报告来学学。


### 项目

* [AutoBahn-Python](http://autobahn.ws/python/)  实现了websocket和WAMP协议的应用服务器，基于twisted和asyncio，用来开发实时交互应用或者物联网应用等，是AutoBahn的一个子项目，其他项目包括不同场景和语言的客户端和服务器端，值得关注下。有点类似于socket.io这种东西。 Python中websocket的方案是非常多的，tornado，gevent，asyncio等等，也算是百家齐放，开源出来的demo比较多，成熟的案例见到的比较少。
* [WAMP](http://wamp-proto.org/implementations/) 这是个基于websocket，json，url的应用层协议，支持pub/sub, rpc, 浏览器方式，并且有这种语言的实现。 物联网，实时应用都有一些成功的案例。感觉非常有意思。
* [django-cacheops](https://github.com/Suor/django-cacheops) 主要用来缓存django queryset的一个框架，针对查询结果的缓存。
* [meinheld](https://github.com/mopemope/meinheld) 基于Python+C的一个高性能异步 WSGI web服务器，可以和Gunicorn结合，作为它的一种自定义worker。 性能非常高。


### 生活

#### 过年月

* 这个月赶上春节，有10来天的时间回家过年了。 基本没怎么开电脑，但是手机也是玩个不停，回到家里发现家里的阿姨们都开始用只能手机了，还自己建立了微信群，玩了好几天的抢红包，正因为有了这个微信群整个春节这些亲戚们一起的活动更多了，交流也比从前多了不少。
* 回到淮北最大的感觉就是 日子不好过，由于煤炭行业的低迷，依靠煤炭生活的日子越来越不好过了，年轻人越来越多的辞职，老工人们也过的比较清苦。
* 发现认识自己是个艰难的过程，不是自负，就是自卑呀！
* 投了几份简历都被拒绝了，面了几个公司也不咋地，发现基础还是薄弱， 很多问题曾经研究过现在没多大印象了，在不就是记得不准。

#### 3月

* 之前的碎片阅读太多，养成了很多坏习惯。 囫囵吞枣，不加思考，只收集不消化，看的多，动手少，其实花了大量的时间，确实没有多少效果。最近再看1w小时理论 这本书，所以准备改改思考，调整下生活学习的状态。 从精深练习，专心研究开始一点一点的认知和学习。
* 开始减少泛读的量，开始坚持每周几个小时的其他方面书籍阅读。 对于技术着手看Gunicorn的源码，对于其中的知识点慢慢补充。
