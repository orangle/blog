title: Python进程间异步通信之signal模块
comments: true
toc: true
date: 2016-03-06 09:54:21
tags: [系统编程]
categories: [python]
---

<!-- more -->

> 信号是Unix系统中常见的一种进程间通信方式（IPC），例如我们经常操作的 `kill -9 pid` 这里的 `-9`对应的就是 SIGKILL 信号, 9就是这个信号的编号，SIGKILL是它的名称。 由于不同版本的 *nux 的实现会有差异，具体请参照系统API，我这里是OSX，可以使用 `man signal`查看所有信号的定义。这里**简单的学习**下Python标准库中信号处理的模块 signal模块.

### 信号是啥？
由于signal是系统编程的接口，那么咱们来看看他的概念。

> 信号(signal)是Linux进程间通信的一种机制，全称为软中断信号，也被称为软中断。信号本质上是在软件层次上对硬件中断机制的一种模拟。

再来看看常用的场景

> 与其他进程间通信方式（例如管道、共享内存等）相比，信号所能传递的信息比较粗糙，只是一个整数。但正是由于传递的信息量少，信号也便于管理和使用，可以用于系统管理相关的任务，例如通知进程终结、中止或者恢复等。每种信号用一个整型常量宏表示，以SIG开头，比如SIGCHLD、SIGINT等，它们在系统头文件<signal.h>中定义。

更具体的介绍和详细的机制原理，请参考 《Unix环境高级编程》等书籍。

自己的理解就是：可以给一个正在运行的进程发送不同的信号，然后进程就能立即收到这个通知，并且做出响应的行为。

常用的几个信号


|编号|名称 |作用|
|-|-|-|
|1|SIGHUP|终端挂起或者终止进程。默认动作为终止进程|
|2|SIGINT|键盘中断 `<ctrl+c>` 经常会用到。默认动作为终止进程|
|3|SIGQUIT|键盘退出键被按下。一般用来响应 `<ctrl+d>`。 默认动作终止进程|
|9|SIGKILL|强制退出。 shell中经常使用|
|14|SIGALRM| 定时器超时，默认为终止进程|
|15|SIGTERM| 程序结束信号，程序一般会清理完状态在退出，我们一般说的优雅的退出|



### signal模块

在[文档](https://docs.python.org/2/library/signal.html)的开头，讲述了Python signal对于系统的封装和一些使用常识, 使用之前应当认真阅读一下。



#### 常用的API

* `signal.signal(signalnum, handler)` 针对不同的信号需要定义对应的处理函数，当运行中的程序接受到对应的信号时候，会调用对应的handler。 handler函数应当有2个参数，一个是 signalnum, 另一是 `stack frame`(None 或者是 [frame对象](https://docs.python.org/2/reference/datamodel.html#frame-objects))

例如写一个小程序，来处理 `ctrl+c`事件和 `SIGHUP`，也就是1和2信号。

```python
#coding:utf-8
#orangleliu py2.7
#recv_signal.py

import signal
import time
import sys
import os

def handle_int(sig, frame):
    print "get signal: %s, I will quit"%sig
    sys.exit(0)

def handle_hup(sig, frame):
    print "get signal: %s"%sig


if __name__ == "__main__":
    signal.signal(2, handle_int)
    signal.signal(1, handle_hup)
    print "My pid is %s"%os.getpid()
    while True:
        time.sleep(3)
```

我们来测试下，首先启动程序（根据打印的pid），在另外的窗口输入 `kill -1 21838` 和 `kill -HUP 21838`, 最后使用 `ctrl+c`关闭程序。 程序的输出如下：

```
# python recv_signal.py
My pid is 21838
get signal: 1
get signal: 1
^Cget signal: 2, I will quit
```

* `signal.getsignal(signalnum)` 根据signalnum返回信号对应的handler，可能是一个可以调用的Python对象，或者是 `signal.SIG_IGN`（表示被忽略）, `signal.SIG_DFL`（默认行为已经被使用） 或 `None`（Python的handler还没被定义）。


获取signal中定义的信号num和名称，还有它的handler是什么

```python
#coding:utf-8
#orangleliu py2.7
#getsignal_handler.py

import signal

def handle_hup(sig, frame):
    print "get signal: %s"%sig

signal.signal(1, handle_hup)

if __name__ == "__main__":

    ign = signal.SIG_IGN
    dfl = signal.SIG_DFL
    print "SIG_IGN", ign
    print "SIG_DFL", dfl
    print "*"*40

    for name in dir(signal):
        if name[:3] == "SIG" and name[3] != "_":
            signum = getattr(signal, name)
            gsig = signal.getsignal(signum)

            print name, signum, gsig
```

运行的结果：可以看到大部分信号都是都有默认的行为。

```
SIG_IGN 1
SIG_DFL 0
****************************************
SIGABRT 6 0
SIGALRM 14 0
SIGBUS 10 0
SIGCHLD 20 0
SIGCONT 19 0
SIGEMT 7 0
SIGFPE 8 0
SIGHUP 1 <function handle_hup at 0x109371c80>
SIGILL 4 0
SIGINFO 29 0
SIGINT 2 <built-in function default_int_handler>
SIGIO 23 0
SIGIOT 6 0
SIGKILL 9 None
SIGPIPE 13 1
SIGPROF 27 0
SIGQUIT 3 0
SIGSEGV 11 0
SIGSTOP 17 None
SIGSYS 12 0
SIGTERM 15 0
SIGTRAP 5 0
SIGTSTP 18 0
SIGTTIN 21 0
SIGTTOU 22 0
SIGURG 16 0
SIGUSR1 30 0
SIGUSR2 31 0
SIGVTALRM 26 0
SIGWINCH 28 0
SIGXCPU 24 0
SIGXFSZ 25 1
```

* `多线程使用信号`
多线程环境下使用信号，只有main thread可以设置signal的handler，也有它能接收到signal.  下面用一个例子看看效果。

```python
#coding:utf-8
#orangleliu py2.7
#thread_signal.py

import signal
import threading
import os
import time

def usr1_handler(num, frame):
    print "received signal %s %s"%(num, threading.currentThread())

signal.signal(signal.SIGUSR1, usr1_handler)

def thread_get_signal():
    #如果在子线程中设置signal的handler 会报错
    #ValueError: signal only works in main thread
    #signal.signal(signal.SIGUSR2, usr1_handler)

    print "waiting for signal in", threading.currentThread()
    #sleep 进程直到接收到信号
    signal.pause()
    print "waiting done"

receiver = threading.Thread(target=thread_get_signal, name="receiver")
receiver.start()
time.sleep(0.1)

def send_signal():
    print "sending signal in ", threading.currentThread()
    os.kill(os.getpid(), signal.SIGUSR1)

sender = threading.Thread(target=send_signal, name="sender")
sender.start()
sender.join()

print 'pid', os.getpid()
#这里是为了让程序结束，唤醒pause
signal.alarm(2)
receiver.join()
```

测试结果

```html
# python thread_signal.py
waiting for signal in <Thread(receiver, started 123145306509312)>
sending signal in  <Thread(sender, started 123145310715904)>
received signal 30 <_MainThread(MainThread, started 140735138967552)>
pid 23188
[1]    23188 alarm      python thread_signal.py
```

* `多进程使用信号`

主要是模拟下pre-fork模式，代码比较长，另写一篇来说明 [Python 中pre-fork模式的简单实现]


### 参考

* [python docs - signal](https://docs.python.org/2/library/signal.html)
* [pymotw - signal](https://pymotw.com/2/signal/)
* [Python - Signal handling and identifying stack frame](http://itsjustsosimple.blogspot.com/2014/01/python-signal-handling-and-identifying.html)


