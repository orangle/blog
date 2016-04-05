title: lzz's reading list 01
comments: true
toc: true
date: 2015-11-21 14:31:24
tags: [reading]
categories: [他山之石]
---

<!-- more -->
>  从上班开始一致有着阅读技术文章和技术资料的习惯，但是几次笔记本的更换，还有因为没有做过比较系统的整理，很多看过的好文章都不知道哪里去了。于是想了这么个办法，把看过的对自己有用的东西整理个列表，顺便写写自己的感受和心得。方便以后查阅，或者当做知识储备材料。

### Python
* [Python进阶必读汇总](http://www.dongwm.com/archives/pythonjin-jie-bi-du-hui-zong/)  明哥的博客总得来说质量挺高的
* [building-an-analytics-app-with-flask](http://charlesleifer.com/blog/saturday-morning-hacks-building-an-analytics-app-with-flask/) 很好的小案例 埋点统计的
* [Simple python functions that provide openssl -aes-256-cbc compatible encrypt/decrypt](http://joelinoff.com/blog/?p=885) 单纯的Pyhton aes加密解密程序很好写，但是如何兼容openssl标准以及其他基于openssl语言的程序， 重点是`IV向量`和`padding`生成的算法
* [Python中级编程](http://book.pythontips.com/en/latest/index.html)  里面提到了很多py比较高级的用法和实践，非常值得一读


### Golang

* [使用Golang快速构建web应用](https://linux.cn/article-4967-1.html#2_1353)    非常简单的入门教程，对于有些web基础的人比较合适，从静态动态请求，一般表单，到数据库连接一整套web开发的hello world。对golang web开发有感性的认识。

* [Golang官方开发wiki案例](http://codethoughts.info/go/2015/03/28/build-web-app-with-go/)   官方文档的翻译，也是web开发的入门，一个简单wiki应用。 以上2篇看完之后，基本的web开发api就熟悉了，接着就可以根据自己的需求对着文档开发了。

### Lua
最近比较热心在openresty的生态上，所以会多关注些相关的知识，ffi是luajit一个非常好用的特性

* [Lua-FFI 官方文档](http://luajit.org/ext_ffi.html) 入门必读啊
* [Lua FFI 实战](http://guiquanz.me/2013/05/19/lua-ffi-intro/)
* [Definitely an openresty guide](http://www.staticshin.com/programming/definitely-an-open-resty-guide/)  这是个入门的文章，讲解了openresty基础和一些容易混淆的概念
* [DetectingUndefinedVariables](http://lua-users.org/wiki/DetectingUndefinedVariables)  or开发中要非常注意全局变量的使用，这里介绍了luac来静态检查全局变量的办法

### Linux

* [openssl加密解密](http://cosx.me/p/957.html) 提到了命令行，lua，python等的一些操作

### 数据库

* [数据库设计的7个常见错误](http://blog.jobbole.com/93953/)  通过一个实际的案例来说明一些数据库设计中容易发生的错误，不使用关键字作为表名，字段名，对一些字段类型的错误理解，索引的常见错误，时区等等。 对于新手来说非常有用。

### 工具

* [pandoc-文档转换工具安装使用](http://guiquanz.me/2015/10/06/an-intro-to-pandoc/)
* [pandoc 的官网](http://pandoc.org/index.html)

### 项目

* [lua-pycrypto-aes](https://github.com/siddontang/lua-pycrypto-aes)  AES加密解密可以在python和lua无缝切换了

### 收获

* openresty 简单重写了3A的portal部分请求，基本都是不需要session，但是访问压力很大的部分
* 对AES加密解密在 openssl, python，luajit，lua的使用有了初步的解决方式
