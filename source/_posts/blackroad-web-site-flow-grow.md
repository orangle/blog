title: 从小流量到大流量架构演变的技术以及注意点
comments: true
toc: true
date: 2015-03-14 20:17:48
tags: [架构,优化]
categories: [分享]
---

<!-- more -->

> 转载：黑夜路人群技术讨论总结，由黑夜路人(公共号heiyeluren-tech)整理，[原文链接](http://mp.weixin.qq.com/s?__biz=MzA3MDA2MjE2OQ==&mid=205575944&idx=1&sn=e19cb9f0dfd8509bb93f3451f38c23fc&scene=1&from=groupmessage&isappinstalled=0#rd)

## 今日话题

> 讨论网站(如电商,游戏等,以具体业务为例)从小流量到大流量架构演变的技术以及注意点 - 耿安鹏

1. 问: 昨天reddit提到把未登录的用户作为二等公民,直接输出缓存,这是在哪一层做的呢? - twin
    - 回: http层？ - linbo
    - 回: 看报道说是丢给cdn了 - liuzhizhi

2. 问: 数据库中 读出来的 数据300w条，想做一个foreach处理。一次处理不了。有什么办法呢。 - 阿杜
    - 回: find_each(limit + offset) 类似rails的find_each - 张建
    - 回: 分批处理呗 - 徐刚
    - 回: 放到redis的set里 - 张建
    - 回: 用 scan迭代 - shawnvan

3. 小流量的站点提前做好动静分离，以便于流量增长后可以对静态代码部署CDN，对动态代码做负载。 - 守墓人-铭

4. 问: 请教个问题！如果做高并发测试的话！nginx   mysql  php  调整哪些参数有效. 这三个调整哪些参数对并发有影响！就是在配置文件中的参数 - 马振国
    - 回: 先调内核参数 - 徐刚
    - 回: 小的话随便调，大了线程数规划就要跟内存大小挂钩。php nginx 都是如此 - 守墓人-铭
    - 回: 看文档呗. 理解操作系统基本概念，tcp/ip，http协议，自然就知道什么意思了 - linbo
    - 回: 就看conf文件里的说明,或者官方的文档. 有时间就一个个试试, 能看源码那更好了. 没有一蹴而就,都是一点点积累. - twin
    - 回: 没这么复杂. 对于php来说，关键配置就几个. 比如什么 php-fpm的  max_requests. 还有 max_childs,这两个最关键. 然后是超时相关的. 一般都用静态，动态太麻烦. 一会启动进程，一会关闭进程的，不够cpu累的.动态一般是你服务器配置不咋滴，或者多个服务一块部署，资源紧张情况下。这年头，只要是个正经公司，不会没事一堆服务部署一台机器基本差不多，超时的几个参数关心下nginx 大概也是几个参数，包括cpu亲合性，针对每个核心设置不同的key。建议剩一个cpu. 剩一个核心的cpu比如你8核cpu，那你nginx就用7个核，保证操作系统别压力太大。nginx还有几个是tcp相关的配置，什么no delay之类的。还有是关于缓存等相关的。另外，建议开启sendfile后端跟php_fpm交互的化，还有fastcgi相关的配置哈哈，主要超时什么的其他基本没有，但是如果用了fastcgi_cache，或者用了proxy，或者proxy_cache，配置稍微不同，不过一般用不到。有一个unix socket域设置fpm里设置一个，设置一个socket文件保存位置。一般直接tcp交互，别用本地socket. 在nginx里 - 黑夜路人
    - 回: fpm nginx 同机，就用socket直接通信 - hilojack
    - 回: socket传输快，但在高并发下不太稳定吧. 我也不知道  这个开发给我的结果 - 马振国
    - 回: 不稳定是胡说 - 黑夜路人
    - 回: 直接用tcp就好了吧，socket只能在本机通信，将来扩展了，nginx和php一般不在一个机器上 - 风之缘

5. 行业:游戏
    - 原来公司将数据库分布式了，如果需要可以远程多调用几个数据库服务器
    - 听说墨麟是将数据库全用分布式redis实现，不用MySQL，这个伸缩性就更大。
    - 用netty，io线程占比，内存池，缓存队列都可以调解
    - 讲个别服务逻辑做成了独立的，例如聊天，场景，战斗都按功能分布式 - 程序员朋
    - 回: 用Redis取代MySQL是错误的 完全取代 算钱，统计全区信息怎么办 怎么做活动 - 如此玄妙
    - 回: 哦 统计好解决吧… 程序合并一下就好啦 - foo
    - 回: 同步到 mysql - JoJo

## 分享链接

1. [那些年，追过的开源软件和技术](http://daily.zhihu.com/story/4577334) - twin

2. [Build beautiful APIs](http://apiary.io/) - 安正超

3. [【原创】Go语言/Golang 知识简单集锦](http://blog.csdn.net/heiyeshuwu/article/details/44223223) - 黑夜路人

4. [Go 1.4+垃圾收集器计划与路线图](http://www.infoq.com/cn/news/2014/08/go-garbage-collector-plan) - 黑夜路人

5. [Go 1.5：使用 Go 来编译 Go](http://www.infoq.com/cn/news/2015/01/golang-15-bootstrapped/) - 黑夜路人

6. [每天200亿次查询– MongoDB在奇虎360](http://mongoing.com/archives/715) - 零度西瓜

7. [关于算法面试的一些思考](http://talkpower.info/blog/algorithm.html) - 黑夜路人

8. [360购物小蜜的秘密——再探推荐引擎](http://mp.weixin.qq.com/s?__biz=MzA4MDEwMTMyMA==&mid=200380841&idx=1&sn=6a55fda1a27fd493696f55920fceb586) - 零度西瓜

9. [Implementation of PHP serialization for Python](https://code.google.com/p/serek/) - 天天

10. [为什么我全力推荐Golang](http://zhuanlan.zhihu.com/tomasen/19959647) - 黑夜路人
