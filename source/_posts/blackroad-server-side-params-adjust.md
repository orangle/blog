title: 服务器调优之提高连接数
comments: true
toc: true
date: 2015-04-04 20:17:48
tags: [优化,并发]
categories: [分享]
---

<!-- more -->

> 转载：黑夜路人群技术讨论总结，由黑夜路人(公共号heiyeluren-tech)整理，[原文链接](http://mp.weixin.qq.com/s?__biz=MzA3MDA2MjE2OQ==&mid=206067272&idx=1&sn=cb0509f80fdc12509d10638d81ed37d3&scene=5#rd)

## 今日话题

> 服务器的参数如何配置优化来提高连接数? - 菜包子

1.  需要调整好多选项  包括最大打开文件句柄数量，包括core文件大小  还有一些tcp相关的配置，比如简单的防ddos攻击的一些选项  最关键就是打开文件描述符大小 - 黑夜路人

2. 前几天生产特别慢，上调了ulimit参数，就ok了 - 召阳

3. 调整的参数
    + net.core.somaxconn
    + backlog
    + net.ipv4.tcp_fin_timeout
    + net.ipv4.tcp_tw_recycle
    + net.ipv4.tcp_max_syn_backlog
    + net.ipv4.tcp_tw_reuse
    + net.ipv4.ip_local_port_range
    + net.ipv4.tcp_max_tw_buckets
    + 还有keepalive的选项  - sky

4. 问: 最关键就是打开文件描述符大小?這怎麼設最恰當 - shuying
    + 回: 设大点 省得不够用 - sky
    + 回: 越大越好 可以设置到128万 - 黑夜路人

5. 我在三个地方看到keepalive的概念，做主备切换的keepalive，tcp协议的keepalive，http的keepalive，是不同的三个概念。 - 项超
    + 回: 这里主要说TCPkeep啊live - 如此玄妙
    + 回: 做ha的是keepalived 其实就是tcp层面和http层面的两个 - sky

6. tcp和http的连接数维持还是不太一样得吧. tcp的话不太熟，http的话一般调sysctl，这个一般直接上网搜有调优说明 - H uangsir

7. GitHub c1000kpracticeguide - 如此玄妙
    + 附: A C1000k demo with detailed description https://github.com/xiaojiaqi/C1000kPracticeGuide

8. 个人感觉不应该一味的提高连接数，还要看后台服务能支持的个数，连接数太高了可能把服务搞挂了。一般连接数都有nginx来控制。不同的服务有不同的连接数需求，比如需要长链接的消息类应用需要的连接数多，连接时间长，而http类的服务要求连接多，连接时间短，时间长了宁可抛弃掉也不能长时间占用着 - 上吊de鱼

9. IM都是大并发 高链接的. http类不能长连接 更多是因为现在服务器不支持. 大多数. HTTP保持长连并无不可. 但是HTTP设计成无状态就是为了扩展 - 如此玄妙

10. 长连接短链接本身并无区别，区别在于适用场景 - platoli
    + 回: 短链接和长连接在逻辑上很大区别
    + 短链接每次处理要做一次身份识别 长连接只要做一次
    + 如何做好这个身份识别就和架构有关系了 - 如此玄妙

11. fd 开启对长连接或者连接数多的非常关键。但是每个连接，内核里都需要占用一定的内存。 - 黑夜路人

12. 一个连接内核里需要占用4K内存（空闲状态）
    + 100w连接需要4g
    + 活动状态，需要占用约8k内存一个连接，100w连接就是8g内存。
    + 内存太小，这个活基本没法干。
    + golang 开发的程序，用户态还得占用一些内存。
    + 一个连接在golang，还需要大约128字节内存占用，结构体之类。
    + 另外，每个协程需要4k内存。
    + 全双工协程需要8k内存。
    + 基本golang里都是一个协程hold一个connection
    + 100W活动连接总内存：
    + 8G（linux连接维护，读写各4k，活动状态） + 1.2G（golang维护连接结构体） + 8G（golang全双工协程内存） = 17.2G
    + 这里是你啥都还没干，业务自己还要申请内存。
    + 那是你用户自己程序里的，不管。只是说系统级的。
    + 所以，能够写100w连接单机的程序，也不容易哇，哈。
    + 消息push类服务还是比较常见的，恩。
    + 我算一下真实场景下。
    + 100W连接真实场景总内存：
    + 4.8G（linux连接维护，读写各4k，活动状态，一般活动连接为10%） + 1.2G（golang维护连接结构体） + 4G（golang非双工协程内存） = 10G
    + 我估计最少10g内存，哈哈。啥都没干呢。
    + 10g内存没了。
    + 嘿嘿。
    + 所以，100w连接的服务器，怎么也得64G内存配置起。
    + 拿c++写应该能够节约不少内存。
    + 不过连接数都这么大了，就别抠唆这点内存了。
    + 说明业务发展的好
    + -- 黑夜路人

13. 之前试过http长连接，算上业务使用的内存，128w连接，吃掉29g的内存
不过用http做长连接的业内似乎比较少吧！ - 项超
    + 问: keepalive 么？ - 黑夜路人
    + 回: 是的 因为是和浏览器交互，所以只能http，websocket后来也没尝试。 - 项超
    + 回: 哈哈，这个方案不推荐。累死了。- 黑夜路人

14. [Linux下高并发socket最大连接数所受的各种限制](http://blog.sae.sina.com.cn/archives/1988) - 星星bigxing

15.  [Linux TCP的相关参数梳理总结](http://m.weibo.cn/3851645388/C9ROf30tz?jumpfrom=weibocom) - Garfielt-刘卫涛

16. [构建C1000K的服务器(1)  – 基础]( http://tc.uc.cn/?v=1&src=l4uLj8XQ0IiIiNGWm5qeiIrRkZqL0J2TkJjQno2cl5aJmozQyMvP0ZeLkpM%3D&restype=1&ucshare=1&ucshareplatform=4&country=cn&os=adr&pf=jdaEnfXr%2BcSL152d7OPsuw%3D%3D) - Garfielt-刘卫涛

17. 问: 一直想问这种维持10W、100W 连接的机器是专门只做请求处理跳转操作的吗？ - Micarol 
    + 回: @Micarol  对的。这个地方不能加任何业务逻辑。 - 我的书包不见了

## 分享链接

1. [ Github遭受大规模DDOS攻击已超80小时](http://m.csdn.net/article/2015-03-30/2824335?reload=1)  - 幻觉大的很

2. [58同城推荐系统架构设计与实现](http://mp.weixin.qq.com/s?__biz=MjM5NTg2NTU0Ng==&mid=204322474&idx=4&sn=c9ed078cd8c69032cc48971fae99f198) - 黑夜路人

3. [无需越狱实现iPhone一键拨号](http://jingyan.baidu.com/article/215817f7e1ffbb1eda1423ff.html) - Soony

4.  [web攻击日志分析之新手指南](http://drops.wooyun.org/%E8%BF%90%E7%BB%B4%E5%AE%89%E5%85%A8/5411) - secdragon

5.  [通过「足记」紧急技术支持过程看设计和运营一款优秀的移动社交App应该注意哪些技术问题](http://mp.weixin.qq.com/s?__biz=MjM5NDcyNzkwMw==&mid=203515922&idx=1&sn=77a3ea14304daa6e24b62f548f396deb) - xingxing

6. [足记火了，难掩创业者技术之殇](http://luochao.baijia.baidu.com/article/51078) - Xiangz

7.  [go 语言可以嵌入JS了](https://godoc.org/github.com/ry/v8worker) - 菜包子

8.  [Best PHP Framework for 2015 – SitePoint Survey Results](http://www.sitepoint.com/best-php-framework-2015-sitepoint-survey-results/) - 姚文强
