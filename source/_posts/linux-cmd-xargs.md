title: Linux命令之xargs
comments: true
toc: true
date: 2016-05-19 17:46:03
tags: [cmd]
categories: [linux]
---

<!-- more -->
> 平时只是在用find命令时会使用 `xargs`命令，一直没怎么系统的学习过，之前一次面试中被问到了，有点哑口无言，现在来学一学。

## 介绍

[维基百科](https://zh.wikipedia.org/wiki/Xargs)是这么介绍的,

> xargs是一条Unix和类Unix操作系统的常用命令。它的作用是将参数列表转换成小块分段传递给其他命令，以避免参数列表过长的问题。
例如删除某个目录下的文件，可以这么做 rm `find /path -type f`, 如果文件过多，就可能出现 `参数列表过长`的错误，导致执行失败。
这个时候使用 xargs 就能比较好的解决问题 `find /path -type f -print0 | xargs -0 rm`。


xargs是通过标准输入或者是管道中的一段字符串来传递命令的`参数列表`, 中间会有个解析的参数的过程，然后调用相应的命令并执行，详细的参数解释请 `man xargs`

## 用法

### -0

xargs参数列表是通过空格，制表符，还有换行符来区分的，例如下面的一个命令

```
find /tmp -name core -type f -print | xargs /bin/rm -f
```

一般情况下会正常的执行，但是文件名中如果有 空格或者是换行符，就会执行失败了

```
find /tmp -name core -type f -print0 | xargs -0 /bin/rm -f
```

这时候加上 `-0` 就能正常执行了。


### -I

替换字符，把输入进来的参数替换成一个自己定义的字符，类似给输入的参数起了一个别名。

```
find . -name '*.py' -print0 | xargs -0 -I fname echo fname python
```

上面的命令会输出 `.py` 文件的名称，然后附加 " python"，也就是 `./test.py` 变成了 `./test.py python`


查看 `/` 目录下的文件

```
echo "ls -ls -h"|xargs -I cmd sh -c cmd" /"
```


### --show-limits

显示当前操作系统，命令行长度的限制

```
[lzz@orangleliu test]$ xargs --show-limits
Your environment variables take up 2161 bytes
POSIX upper limit on argument length (this system): 2617231
POSIX smallest allowable upper limit on argument length (all systems): 4096
Maximum length of command we could actually use: 2615070
Size of command buffer we are actually using: 131072

Execution of xargs will continue now, and it will try to read its input and run commands; if this is not what you wanted to happen, please type the end-of-file keystroke.
Warning: /bin/echo will be run at least once.  If you do not want that to happen, then press the interrupt keystroke.
```

### 例子

删除当前目录下 `.c`文件

```
find . -name "*.c" -print0 | xargs -0 rm -rf
```

查找当前目录下包含 `utf-8`的 `.py` 文件，显示行号

```
find . -name '*.py' -print0 | xargs -0 grep -n 'utf-8'
```

查找当前目录下 `.bak` 文件，并移动到 `~/old.files` 文件夹下面

```
find . -name "*.bak" -print0 | xargs -0 -I file mv file ~/old.files
```


实际使用中，复杂的操作比较少，暂且记录那么多。
