title: Python和shell中Base64编码使用那些事
comments: true
toc: true
date: 2016-04-17 09:40:30
tags: [shell]
categories: [密码学]
---

<!-- more -->
> 做开发第一个接触的编码方式就是Base64，当时是用url来传输一些参数，传输的两端会用Base64来编码和解码，保证数据不被url转义破坏。



下面是 [维基百科](https://zh.wikipedia.org/wiki/Base64) Base64  中的介绍，其实自己实现起来也不是很麻烦。

> **Base64** 是一种基于64个可打印字符来表示二进制数据的表示方法。由于2的6次方等于64，所以每6个比特为一个单元，对应某个可打印字符。三个字节有24个比特，对应于4个Base64单元，即3个字节需要用4个可打印字符来表示。它可用来作为电子邮件的传输编码。在Base64中的可打印字符包括字母A-Z、a-z、数字0-9，这样共有62个字符，此外两个可打印符号在不同的系统中而不同。



### Shell中使用

一般Linux系统中都会有个base64的命令，openssl命令也有base的选项。

#### base64 命令

字符串操作，更多参考 `man base64`


```bash
# echo "you are so cool"|base64
eW91IGFyZSBzbyBjb29sCg==

# echo "eW91IGFyZSBzbyBjb29sCg=="|base64 -d
you are so cool

#中文

# echo "你真帅"|base64
5L2g55yf5biFCg==

# echo "5L2g55yf5biFCg=="|base64 -d
你真帅
```

#### openssl命令

```bash
$ openssl enc -base64 <<< "good boy"
Z29vZCBib3kK


$ openssl enc -base64 -d <<< "Z29vZCBib3kK"
good boy
```

### Python 使用base64

标准库中提供了base16，base32, base64 好几个接口，最常用的是 `base64.b64encode` 和 `base64.b64decode` ，还有针对于url的改进方式 `base64.urlsafe_b64encode` 和 `base64.urlsafe_b64decode`, 这一对方法和标准的base64不同的是针对url改善了64个字符中最后2个，具体的看这里的[说明](https://docs.python.org/2/library/base64.html#base64.urlsafe_b64decode), 结合wiki中的 `base64索引表` 就明白了。


#### 编码 和 解码

```bash
$ python -c "import base64; print base64.b64encode('you are so cool')"
eW91IGFyZSBzbyBjb29s

$ python -c "import base64; print base64.b64decode('eW91IGFyZSBzbyBjb29s')"
you are so cool
```



注意无法对unicode直接base64编码, 所以请注意字符编码问题。

```bash
# python -c "import base64; print base64.b64encode('酷')"
6YW3

# python -c "import base64; print base64.b64encode(u'酷')"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/System/Library/Frameworks/Python.framework/Versions/2.7/lib/python2.7/base64.py", line 53, in b64encode
    encoded = binascii.b2a_base64(s)[:-1]
UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-2: ordinal not in range(128)
```



#### 常用代码片段

经常使用的方式是对编码之后字符串结尾的 `=`做剔除 。对于 `urlsafe_b64encode` 和 `b64encode` 都适用，针对不同编程语言相互转化的情况请具体参照语言的实现方式。



```python
import base64

def base64encode(raw):
    return base64.urlsafe_b64encode(raw).strip("=")

def base64decode(data):
    return base64.urlsafe_b64decode(data + "=" * (-len(data)%4))
```



### 阅读

* [how can i decode a base64 string from command line](http://askubuntu.com/questions/178521/how-can-i-decode-a-base64-string-from-the-command-line)
* [base64 中文Wiki](https://zh.wikipedia.org/wiki/Base64)
* [python2 base64文档](https://docs.python.org/2/library/base64.html)
* [PyMOTW base64](https://pymotw.com/2/base64/)
* [how to base64 url decode in python](http://stackoverflow.com/questions/3302946/how-to-base64-url-decode-in-python)

