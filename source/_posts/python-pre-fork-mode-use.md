title: Python中prefork模式的简单实现
comments: true
toc: true
date: 2016-03-06 09:54:59
tags: [perfork]
categories: [python]
---

<!-- more -->

### prefork 模式是啥？

这种模式最早看到是在Apache的介绍中，所以一般看到这个词就会想到Apache(现在也支持其他模式 worker，event等)，web服务器用的比较多nginx，gunicorn，unicorn等。 这个模式中有Master和Worker两种角色的进程，Master进程fork出Worker来处理任务（例如http请求）。 Master进程的作用是管理Worker进程，包括fork出worker进程，接受外部信号对worker生命周期进行控制，对worker进程进行健康检查等等，Worker进程的作用就是处理任务(干活)。

### 这种模式的好处？

* 充分利用多个CPU，特别是在python这种语言中，不受GIL的影响
* 一个进程出现问题，其他进程仍然可以正常服务，不会整个server挂掉

盗了一张图来是 [xiaorui.cc](http://xiaorui.cc)，Gunicorn prefork流程。

![Gunicorn prefork](/img/preforkxxx.png)


### python 中怎么实现的？

用的知识，和简单的思路。


下面是阅读Gunicorn源码之后，实现的一个简单的prefork程序。

```python
# -*- coding: utf-8 -*-
#master-slaves.py  python2.7.x
#orangleliu@gmail.com
'''
简单的模拟pre-fork模式，master进程控制多个子进程

这里实现这么几个信号
INT ctrl+c 退出
TTIN 增加一个worker
TTOU 减少一个worker
'''

import os
import sys
import signal
import time
import random


class Worker(object):
    '''
    子进程要实现一些特定的信号来响应外界和父进程的操作
    '''

    def run(self):
        while True:
            time.sleep(3)


class Master(object):

    WORKERS = {}
    SIG_QUEUE = []
    SIGNALS = [getattr(signal, "SIG%s" % x)
                for x in "INT TTIN TTOU".split()]
    SIG_NAMES = dict(
        (getattr(signal, name), name[3:].lower()) for name in dir(signal)
        if name[:3] == "SIG" and name[3] != "_"
    )

    def __init__(self, worker_nums=2):
        self.worker_nums = worker_nums
        self.master_name = "Master"
        self.reexec_pid = 0

    def start(self):
        print "start master"

        self.pid = os.getpid()
        self.init_signals()

    def init_signals(self):
        [signal.signal(s, self.signal) for s in self.SIGNALS]
        signal.signal(signal.SIGCHLD, self.handle_chld)

    def signal(self, sig, frame):
        '''
        普通的信号发生的时候，往信号队列增加一个信号
        '''
        if len(self.SIG_QUEUE) < 5:
            self.SIG_QUEUE.append(sig)

    def run(self):
        self.start()

        try:
            self.manage_workers()
            while True:
                # 如果不增加sleep 整个master进程就会进入几乎100 cpu的状态
                # 使用sleep的好处就是master的cpu消耗小很多，对于来自系统的给master的信号可以即使反馈
                time.sleep(1)

                sig = self.SIG_QUEUE.pop(0) if len(self.SIG_QUEUE) else None
                if sig is None:
                    self.manage_workers()
                    continue

                if sig not in self.SIG_NAMES:
                    print "unknow signals:%s"%sig
                    continue

                signame = self.SIG_NAMES.get(sig)
                handler = getattr(self, "handle_%s"%signame, None)
                if not handler:
                    print "Unhandler signal: %s"%signame
                    continue

                handler()
        except StopIteration:
            self.halt()
        except KeyboardInterrupt:
            self.halt()
        except SystemExit:
            pass
        except Exception as e:
            print e
            self.stop()
            sys.exit(-1)

    def handle_chld(self, sig, frame):
        '''
        对于子进程退出SIGCHLD信号处理，防止产生大量僵尸进程
        '''
        self.reap_workers()

    def handle_int(self):
        '''
        ctrl+c 关闭master进程，先关闭子进程，然后抛出异常，自己退出
        '''
        self.stop()
        raise StopIteration

    def handle_ttin(self):
        '''
        增加一个子进程
        '''
        print "add a worker"
        self.worker_nums += 1
        self.manage_workers()

    def handle_ttou(self):
        '''
        减少一个子进程
        '''
        print "deincrease a worker"
        if self.worker_nums <= 1:
            return
        self.worker_nums -= 1
        self.manage_workers()

    def stop(self):
        '''
        停止子进程 这里都当做SIGTERM来处理
        '''
        print 'stop workers'
        sig = signal.SIGTERM
        self.kill_workers(sig)
        self.kill_workers(signal.SIGKILL)

    def halt(self, exit_status=0):
        '''
        master 进程自杀
        '''
        print "master exit"
        self.stop()
        sys.exit(exit_status)

    def reap_workers(self):
        '''
        这里的检测也是为了避免僵尸进程，否则大量资源无法释放
        参考：http://www.cnblogs.com/mickole/p/3187770.html
        '''
        try:
            while True:
                #os.waitpid 收集僵尸子进程的信息,并把它彻底销毁后返回
                #这里的 -1 代表所有子进程
                #os.WNOHANG 如果没有子进程信息就立刻返回
                wpid, status = os.waitpid(-1, os.WNOHANG)
                if not wpid:
                    break
                else:
                    exitcode = status >> 8
                    worker = self.WORKERS.pop(wpid, None)
                    if not worker:
                        continue
        except OSError as e:
            #errno.ECHILD 是没有子进程错误
            if e.error != errno.ECHILD:
                raise


    def manage_workers(self):
        '''
        workers 的健康检查，数量是否对齐等
        '''
        if len(self.WORKERS.keys()) < self.worker_nums:
            self.spawn_workers()

        workers = self.WORKERS.items()
        while len(workers) > self.worker_nums:
            (pid, _) = workers.pop(0)
            self.kill_worker(pid, signal.SIGTERM)

    def spawn_worker(self):
        worker = Worker()
        pid = os.fork()

        #master进程处理
        if pid != 0:
            self.WORKERS[pid] = worker
            return pid

        #worker进程处理
        worker_pid = os.getpid()
        try:
            worker.run()
            sys.exit(0)
        except SystemExit:
            raise
        except Exception as e:
            print "work error %s"%str(e)
            sys.exit(-1)

    def spawn_workers(self):
        for i in range(self.worker_nums - len(self.WORKERS.keys())):
            self.spawn_worker()
            #为什么要那么端时间的休眠
            time.sleep(0.1*random.random())

    def kill_workers(self, sig):
        worker_pids = list(self.WORKERS.keys())
        for pid in worker_pids:
            self.kill_worker(pid, sig)

    def kill_worker(self, pid, sig):
        try:
            os.kill(pid, sig)
        except OSError as e:
            print "kill worker error: %s"%str(e)


if __name__ == "__main__":
    Master().run()
```

### 有用阅读

* [python使用master worker管理模型开发服务端](http://xiaorui.cc/2015/07/13/python%E4%BD%BF%E7%94%A8master-worker%E7%AE%A1%E7%90%86%E6%A8%A1%E5%9E%8B%E5%BC%80%E5%8F%91%E6%9C%8D%E5%8A%A1%E7%AB%AF/)


