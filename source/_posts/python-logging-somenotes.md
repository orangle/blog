title: python logging 模块使用杂记
comments: true
toc: true
date: 2016-03-14 18:25:26
tags: [logging]
categories: [编程]
---

<!-- more -->
> 因为日志模块是线程安全的，在单进程应用中一般的问题集中在如何切分等，多进程相对来说就要复杂一些，可能需要一些自己定制的handler。


### 日志切分

logging标准库中提供的日志切分的方式有

* TimedRotatingFileHandler 基于文件大小的切分，支持多个backup
* RotatingFileHandler 基于时间间隔的切分，支持多种时间单位，多个backup。 启动时查询log文件最后修改的时间戳当做start计算时间，如果要是定义24小时切割一次，而程序12小时重启一次，那么永远都不会切割日志，这就是它不好的地方。 很多时候是希望它根据日志文件的创建时间来计算时间差。
* 很多时候需要能按整点时间轮询的日志 例如每天只是这一天24小时的记录，每小时只是当前这个小时的记录。标准库无法满足，需要自己写handler


当日志文件被移动或删除后:

* FileHandler会继续将日志输出至原有的文件描述符, 从而导致日志切分后日志丢失.
* WatchedFileHandler会检测文件是否被移动或删除, 如果有, 会新建日志文件, 并输出日志到新建的文件. 这个handerl+corntab可以实现日志的割切。



### 多进程日志

* 同步输出: 看到有些是使用文件锁来实现实现同步，也看到有些是使用多进程的queue来实现同步，或者使用通过 sockethandler 传送给单独服务
*  多进程往一个日志文件写日志（日志可能会错行，变得杂乱无序），然后用cron这种外部工具来切分日志


#### ConcurrentLogHandler

看到stackoverflow 上很多人回答用 [ConcurrentLogHandler](https://pypi.python.org/pypi/ConcurrentLogHandler/0.9.1)来解决问题，RotatingFileHandler的多进程版本，它主要使用文件锁来保持同步。

源码[在这里](http://bazaar.launchpad.net/~lowell-alleman/python-concurrent-log-handler/master/files),  下载下来，结合多进程Gunicorn来测测，使用方法[参见文档](https://pypi.python.org/pypi/ConcurrentLogHandler/0.9.1)，非常简单方面.

我这里1000并发测试的情况下，性能影响不大，大概百分之十（这个主要取决于日志的复杂度），日志同步和轮询都没问题。 更大规模和复杂的情况还不清楚会怎样，在小程序中使用应该问题不大，要求较高的系统需要进一步测试。


### socket 日志

官方的 [loging cookbook](https://docs.python.org/2/howto/logging-cookbook.html) 中推荐在多进程中使用 `SocketHandler`来解决日志问题, 还给出来一个简单的例子 [here](https://docs.python.org/2/howto/logging-cookbook.html#sending-and-receiving-logging-events-across-a-network)。 还有很多开源的方案Syslog，Sentry，或者Github找些别人写的日志收集server，自己撸一个轮子等，选择很多，适合日志集中收集管理。


### 使用总结

* root 级别的设置只在应用中进行，不要在模块中设置
* 一个文件中有多个类，日志最好每个类一个logger，否则在生产中很难快速定位问题
*  `self.logger = logging.getLogger(type(self).__name__)` 这种方式在base class设置，这样每个子类都有不同的logger名称
* 规范format，这样容易处理日志
* logging对整体的性能有些影响的，如果想减少到更小可以 `import syslog`， 但是怎么回溯，收集日志等要好好盘算下，[use nothing but local syslog](http://www.aminus.org/blogs/index.php/2008/07/03/writing-high-efficiency-large-python-sys-1?blog=2)
* 需要记录一个异常的错误栈情况，可以使用 Logger.exception() 方法

```
#logger 是已经初始化的Logger
try:
    print err
except Exception as e:
    logger.exception("")
```


### 源码阅读

看的是 python2.6 的logging模块源码，[源码在这里](https://github.com/python/cpython/tree/2.6/Lib/logging)。日志相关的是 `pep282`， logging模块的参照是 apache `log4j` 日志系统。


logging模块一共是3个文件

*  `__init__.py`  常量定义，基础的API，logger，handler的父类，handler的适配器等等，最核心的代码
*  `config.py`  配置读取相关模块
* `handlers.py`  不同的handlers来处理日志的输出策略，如果自己写handler 这里是最好的例子



#### 通过代码来看看使用 basicConfig 发生了什么？

你可以看这个文章[dive into logging](http://atlee.ca/blog/posts/diving-into-python-logging.html), 其实看下代码就能明白。

在 `import logging` 的时候会自动生成一个 root logger, name就是root，默认级别是 warning


源码里是这样的

```

root = RootLogger(WARNING)
Logger.root = root
Logger.manager = Manager(Logger.root)
```

这个默认的logger也basicConfig需要才能使用，basicConfig其实就是设置root logger的handler。


```python
In [1]: import logging

In [2]: logger = logging.getLo
logging.getLogger       logging.getLoggerClass

In [2]: logger = logging.getLogger('lzz')

In [3]: logger.info('lzz')

In [4]: logger.root.info("lzz")

In [5]: logging.basicConfig(level=logging.INFO)

In [6]: logging.root.info("hello")
INFO:root:hello

In [7]: logger.info('lzz')
INFO:lzz:lzz
```



#### 阅读总结

* Logger 之间是一个树型结构，root Logger是根节点，父子关系等由Manager类处理
* 一般是获取一个Logger实例，设置log级别，添加一个或者多个Handler，一个或者多个Filter，Handler中设置format日志格式。
* 执行一个logger方法的过程： 例如`logger.info("hi boy")`,  首先logger检查level是否符合，如果level正确，调用 `Logger._log()`方法，根据log内容组成 `LogRecord`, 调用`Logger.handle(self, record)`方法。

`Logger.handler()` 检查logger是否disable，如果可用, 再调用 `Logger.filter(record)`, 根据设置的过滤器来过滤日志。 如果record没有过滤掉，那么继续调用 `Logger.callHandler(self, record)`, 根据配置的handlers交给handlers来处理日志。说了一通，不如直接看[官方的流程图](https://docs.python.org/2/howto/logging.html#logging-flow)


![官方的流程图](/img/logging_flow.png)

* Filter 比设置日志level有更大的灵活性, Logger和Handlers都可以设置Filter, 例如写一个过滤 hello关键字的Filter。

```python
#coding:utf-8
import logging

logger = logging.getLogger(__name__)
logging.basicConfig(level=logging.INFO)

class HelloFilter(logging.Filter):
    #overwrite filter method, 最好先看看LogRecord这个类
    def filter(self, record):
        if record.msg.find('hello') > -1:
            return 0
        return 1

logger.addFilter(HelloFilter())
logger.info("hello")
logger.info("hello boy")
logger.info("hi girl")
```

* 简单的类关系如下图

![class_pic](/img/logging_class.png)

### 阅读

* [logging api](https://docs.python.org/2/library/logging.html#module-logging) 查询手册
* [logging howto](https://docs.python.org/2/howto/logging.html) 官方教程，基础知识，全面实用
* [logging cookbook](https://docs.python.org/2/howto/logging-cookbook.html)  进阶必读，覆盖常用情景
* [python-how-to-create-rotating-logs](http://www.blog.pythonlibrary.org/2014/02/11/python-how-to-create-rotating-logs/)  简单明了说明了 两个常用的日志轮询handler怎么使用
*  [how-python-logging-module-works](http://www.shutupandship.com/2012/02/how-python-logging-module-works.html) 源码笔记，看懂logging怎样工作
* [pep282](https://www.python.org/dev/peps/pep-0282/)
