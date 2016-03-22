title: tornado源码阅读
comments: true
toc: true
date: 2016-03-17 13:45:11
tags: [tornado 源码]
categories: [编程]
---

<!-- more -->

> 如果提到高性能的Python web框架，很多人的第一反应可能就是[Tornado](https://github.com/tornadoweb/tornado)，最早是[FriendFeed](http://friendfeed.com/)开源的。虽然写了挺久的Python，可是还没有在公司的项目中用到它，真是遗憾呀。这次准备好好看看它的源码实现，记录一些阅读心得。

## Tornado的特点

* 使用 `non-blocking network I/O`, 例如 epoll，kqueue等
* 维持成千上万的连接
* 可以处理长轮询，websocket等长连接场景

## 概览

基本调试学习环境, 使用的经验少，结合文档和案例来熟悉使用。 [阅读注释源码](https://github.com/orangle/tornado-source-reading)

* python2.7+
* tornado4.3+
* OSX

tornado项目的中包含了 demos，docs，maint, tornado几个目录，Python源码在tornado目录中，主要是看这一块，这里有2个目录和一些py文件，
* `platform`目录是不同平台下IO模型的实现
* `test`目录下是各个功能模块的测试用例
* 剩下的 `.py` 文件就是这个框架的各个功能实现了

### IOLoop

`ioloop.py` 这是tornado最核心的部分了，由于有了这个无阻塞的`I/O`事件循环机制，才能维持成千上万的connect。基于IOLoop不仅可以做HTTP server也可以做TCP，UDP服务端的开发。



### IOStream


### HTTPServer



## 阅读
* [Understanding the code inside Tornado, the asynchronous web server](http://golubenco.org/understanding-the-code-inside-tornado-the-asynchronous-web-server-powering-friendfeed.html) 09年对Tornado源码阅读的文章，大致的骨架基本没变



