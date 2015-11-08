title: Python程序性能分析
comments: true
toc: true
date: 2015-05-23 20:17:48
tags: [python,性能]
categories: [翻译]
---

<!-- more -->

> 有些脚本发现比预期要慢的多，就需要找到瓶颈，然后做相应的优化，参考[A guide to analyzing Python performance](http://www.huyng.com/posts/python-performance-analysis/),也可以说是翻译。

### 指标
*  运行时间
*  时间瓶颈
*  内存使用
*  是否有内存泄漏

### 基本
#### linux `time`
这是个shell中自带的命令，也是最简单和方面的方法，但是得到信息太少

```
[root@bogon util]# time python pvsts.py
Yesterday PV/UV

PV 46300
UV is 3899

real    2m36.591s  #花费时间
user    2m37.167s  ＃用户态时间
sys     0m2.010s   ＃内核态时间
```

如果 `sys`+`user` 比 `real` 小的多，就要考虑io等待时间是否过长了。

#### 使用Cprofile工具
用起来很简单，显示的东西也很多，但是对于`代码`来说不是很直观

```
[root@bogon util]# python -m cProfile pvsts.py
Yesterday PV/UV

PV 46300
UV is 3899
         502249600 function calls (502249597 primitive calls) in 250.221 CPU seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000  250.221  250.221 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 __future__.py:48(<module>)
        1    0.000    0.000    0.000    0.000 __future__.py:74(_Feature)
        7    0.000    0.000    0.000    0.000 __future__.py:75(__init__)
        1    0.000    0.000    0.000    0.000 __init__.py:49(normalize_encoding)
        1    0.000    0.000    0.000    0.000 __init__.py:71(search_function)
        1    0.000    0.000    0.000    0.000 base64.py:3(<module>)
```

### 测试时间工具`line_profiler`

就是这个小工具，安装很simple

```
$ pip install line_profiler
```

在想要测试的函数上添加一个 `@profile`装饰器（不用倒入任何包，工具会自动倒入）


```python
@profile
def sts_uv():
        #mac_list = []
        mac_set = set()
        with open(temp_log, 'r') as f:
                for line in f.readlines():
                        basid, mac, ip = decode_token(str(line.strip()))
                        #mac_list.append(mac)
                        mac_set.add(mac)
        #uv = len(set(mac_list))
        uv = len(mac_set)
        print "UV is {0}".format(uv)
        return uv
```

得到结果：

```bash
[root@bogon util]# kernprof -l -v pvsts.py
Yesterday PV/UV

PV 46300
UV is 3899
Wrote profile results to pvsts.py.lprof
Timer unit: 1e-06 s

Total time: 450.299 s
File: pvsts.py
Function: sts_uv at line 74

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
    74                                           @profile
    75                                           def sts_uv():
    76                                                  #mac_list = []
    77         1           10     10.0      0.0          mac_set = set()
    78         1           59     59.0      0.0         with open(temp_log, 'r') as f:
    79     42431        38556      0.9      0.0                 for line in f.readlines():
    80     42430    450188794  10610.2    100.0                         basid, mac, ip = decode_token(str(line.strip()))
    81                                                                  #mac_list.append(mac)
    82     42430        71491      1.7      0.0                          mac_set.add(mac)
    83                                                  #uv = len(set(mac_list))
    84         1            2      2.0      0.0          uv = len(mac_set)
    85         1           15     15.0      0.0         print "UV is {0}".format(uv)
    86         1            1      1.0      0.0         return uv
```

同时还是会生成一个`pvsts.py.lprof`文件

### 测试内存使用 `pip install -U memory_profiler`

安装两个工具

```
$ pip install -U memory_profiler
$ pip install psutil
```
使用上也是添加一个 '@profile' 装饰器，跟上面的一样。

测试

```bash
[root@bogon util]# python -m memory_profiler pvsts.py
Yesterday PV/UV

PV 46300
UV is 3899
Filename: pvsts.py

Line #    Mem usage    Increment   Line Contents
================================================
    74    9.676 MiB    0.000 MiB   @profile
    75                             def sts_uv():
    76                                  #mac_list = []
    77    9.676 MiB    0.000 MiB           mac_set = set()
    78    9.676 MiB    0.000 MiB        with open(temp_log, 'r') as f:
    79   15.289 MiB    5.613 MiB                for line in f.readlines():
    80   15.289 MiB    0.000 MiB                        basid, mac, ip = decode_token(str(line.strip()))
    81                                                  #mac_list.append(mac)
    82   15.289 MiB    0.000 MiB                           mac_set.add(mac)
    83                                  #uv = len(set(mac_list))
    84   14.961 MiB   -0.328 MiB           uv = len(mac_set)
    85   14.961 MiB    0.000 MiB        print "UV is {0}".format(uv)
    86   14.961 MiB    0.000 MiB        return uv
```
