title: Python logging 模块和使用经验
comments: true
toc: true
date: 2016-12-27 15:49:09
tags: [logging]
categories: [编程]
---

<!-- more -->
> 记录下常用的一些东西，每次用总是查文档有点小麻烦。 py2.7
> 日志应该是生产应用的重要生命线，谁都不应该掉以轻心

## 有益原则

### 级别分离
日志系统通常有下面几种级别，看情况是使用

* FATAL - 导致程序退出的严重系统级错误，不可恢复，当错误发生时，系统管理员需要立即介入，谨慎使用。
* ERROR - 运行时异常以及预期之外的错误，也需要立即处理，但紧急程度低于FATAL,当错误发生时，影响了程序的正确执行。需要注意的是这两种级别属于服务自己的错误，需要管理员介入，用户输入出错不属于此分类。
* WARN - 预期之外的运行时状况，表示系统可能出现问题。对于那些目前还不是错误，然而不及时处理也会变成错误的情况，也可以记为WARN，如磁盘过低。
* INFO - 有意义的事件信息，记录程序正常的运行状态，比如收到请求，成功执行。通过查看INFO,可以快速定位WARN，ERROR, FATAL。INFO不宜过多，通常情况下不超过TRACE的10%。
* DEBUG - 与程序运行时的流程相关的详细信息以及当前变量状态。
* TRACE - 更详细的跟踪信息。DEBUG和TRACE这两种规范由项目组自己定义,通过该种日志，可以查看某一个操作每一步的执行过程，可以准确定位是何种操作，何种参数，何种顺序导致了某种错误的发生

### 单独目录
日志最好放到单独的日志目录，例如 `/var/logs/` 下，按照应用分成不同的目录，或者是文件。日志不要放在应用目录下，那样不利于自动化部署和应用升级，备份等。

### 日志分类
诊断日志，统计日志，审计日志等等，不同用途等日志存储到不同的文件中，方面后面的查询，分析。

### 日志格式
不管是web日志，还是应用日志，最好有一个比较统一的格式（例如时间格式），方面日志的查询，入库，和分析。还有一些应用统一使用json的日志格式，也挺好的。

### 不好的做法
* 日志中含有用户敏感信息
* 线上程序中使用 print
* 生产环境使用 debug 级别日志 😢

### 日志切分

日志可以按照每天，每周或者是文件的大小，切分之后压缩。一方面容易按时间回溯，另一方面可以减少磁盘空间，对于很久之前的日志，可以传输到远程服务器，或者是删除。


## Python 日志

### 好习惯
* root级别的设置: 日志格式, 有利于标准化
* class 中设置logger `self.logger = logging.getLogger(type(self).__name__) `
* 模块，文件中设置 logger `logger = logging.getLogger(__name__)`
* 使用JSON YAML等格式来配置logging，感觉比使用代码或者 ini格式看起来更方面
* 错误日志是比较特殊的日志，因为它需要更多的信息，例如错误产生的上下文，还有错误堆栈等信息。可以通过 `python logging context pypi` 关键词google一些信息，或者自己设计一个 logging handler 来实现。

### 实际问题

* 简单的小应用中，单个日志文件，同时还要打印控制台

```
import logging

logging.basicConfig(level=logging.DEBUG,
                    format='%(asctime)s %(name)-12s %(levelname)-8s %(message)s',
                    datefmt='%m-%d %H:%M',
                    filename='/temp/myapp.log',
                    filemode='w')
console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
# add the handler to the root logger
logging.getLogger('').addHandler(console)
```

* 记录 Exception 的trace 信息（很有用哦)

```
try:
    open('/path/to/does/not/exist', 'rb')
except (SystemExit, KeyboardInterrupt):
    raise
except Exception, e:
    logger.error('Failed to open file', exc_info=True)
```

* ini 格式例子

这里用了第三方的一个handler，ConcurrentRotatingFileHandler， 实现多进程安全

```
[loggers]
keys=root

[handlers]
keys=stream, rotatingFile, errorFile

[formatters]
keys=form01

[logger_root]
level=DEBUG
handlers=stream, rotatingFile, errorFile

[handler_stream]
class=StreamHandler
level=NOTSET
formatter=form01
args=(sys.stdout,)

[handler_errorFile]
class=FileHandler
level=ERROR
formatter=form01
args=('./logs/portal.log', 'a')

[handler_rotatingFile]
level=INFO
formatter=form01
class=handlers.ConcurrentRotatingFileHandler
args=('./logs/portal.log','a',50240000, 10)

[formatter_form01]
format=%(asctime)s %(name)s %(levelname)s %(message)s
datefmt=
class=logging.Formatter
```

引用
```
import logging
import logging.config
import cloghandler
logging.config.fileConfig(join(BASE_DIR, "conf/log.conf"))

logger = logging.getLogger(__name__)
```

默认会使用 root 这个logger，如果名称匹配就使用对应的logger。 一个logger也可以指定多个 handdler, 用来处理不同的日志级别等。

* JSON格式 例子

配置
```
{
    "version": 1,
    "disable_existing_loggers": false,
    "formatters": {
        "simple": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        }
    },

    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "DEBUG",
            "formatter": "simple",
            "stream": "ext://sys.stdout"
        },

        "info_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "INFO",
            "formatter": "simple",
            "filename": "info.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf8"
        },

        "error_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "ERROR",
            "formatter": "simple",
            "filename": "errors.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf8"
        }
    },

    "loggers": {
        "my_module": {
            "level": "ERROR",
            "handlers": ["console"],
            "propagate": "no"
        }
    },

    "root": {
        "level": "INFO",
        "handlers": ["console", "info_file_handler", "error_file_handler"]
    }
}
```

获取配置
```
import os
import json
import logging.config

def setup_logging(
    default_path='logging.json',
    default_level=logging.INFO,
    env_key='LOG_CFG'
):
    """Setup logging configuration

    """
    path = default_path
    value = os.getenv(env_key, None)
    if value:
        path = value
    if os.path.exists(path):
        with open(path, 'rt') as f:
            config = json.load(f)
        logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)
```
* 把日志格式化成json的工具

[python-json-logger](https://github.com/madzak/python-json-logger)，[logmatic](https://github.com/logmatic/logmatic-python/blob/master/logmatic/__init__.py) json-logger增加的一些封装

```
import logging.handlers
from pythonjsonlogger import jsonlogger
import datetime

class JsonFormatter(jsonlogger.JsonFormatter, object):
    def __init__(self,
                 fmt="%(asctime) %(name) %(processName) %(filename)  %(funcName) %(levelname) %(lineno) %(module) %(threadName) %(message)",
                 datefmt="%Y-%m-%dT%H:%M:%SZ%z",
                 style='%',
                 extra={}, *args, **kwargs):
        self._extra = extra
        jsonlogger.JsonFormatter.__init__(self, fmt=fmt, datefmt=datefmt, *args, **kwargs)

    def process_log_record(self, log_record):
        # Enforce the presence of a timestamp
        if "asctime" in log_record:
            log_record["timestamp"] = log_record["asctime"]
        else:
            log_record["timestamp"] = datetime.datetime.utcnow().strftime("%Y-%m-%dT%H:%M:%S.%fZ%z")

        if self._extra is not None:
            for key, value in self._extra.items():
                log_record[key] = value
        return super(JsonFormatter, self).process_log_record(log_record)
```


## 参考
* [Logging Cookbook](http://docs.python.org/2/howto/logging-cookbook.html#logging-cookbook) 文档一定是要看的
* [日志最佳实践](http://themoon.me/2016/01/05/日志最佳实践/)
* [good-logging-practice-in-python](https://fangpenlin.com/posts/2012/08/26/good-logging-practice-in-python/)
* [Sentry](https://sentry.io/) 日志收集，分析系统，基于Python
* [ConcurrentLogHandler](#) 解决多进程日志同步问题
