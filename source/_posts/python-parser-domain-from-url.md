title: Python从url中解析域名的几种方法
comments: true
toc: true
date: 2014-09-24 17:58:48
tags: [python]
categories:
---

<!-- more -->

>从url中找到域名,首先想到的是用正则，然后寻找相应的类库。用正则解析有很多不完备的地方，url中有域名，域名后缀一直在不断增加等。通过google查到几种方法，一种是用Python中自带的模块和正则相结合来解析域名，另一种是使第三方用写好的解析模块直接解析出域名。

## 要解析的url

```
urls = ["http://meiwen.me/src/index.html",
          "http://1000chi.com/game/index.html",
          "http://see.xidian.edu.cn/cpp/html/1429.html",
          "https://docs.python.org/2/howto/regex.html",
          """https://www.google.com.hk/search?client=aff-cs-360chromium&hs=TSj&q=url%E8%A7%A3%E6%9E%90%E5%9F%9F%E5%90%8Dre&oq=url%E8%A7%A3%E6%9E%90%E5%9F%9F%E5%90%8Dre&gs_l=serp.3...74418.86867.0.87673.28.25.2.0.0.0.541.2454.2-6j0j1j1.8.0....0...1c.1j4.53.serp..26.2.547.IuHTj4uoyHg""",
          "file:///D:/code/echarts-2.0.3/doc/example/tooltip.html",
          "http://api.mongodb.org/python/current/faq.html#is-pymongo-thread-safe",
          "https://pypi.python.org/pypi/publicsuffix/",
          "http://127.0.0.1:8000"
          ]
```

## 使用[urlparse](https://docs.python.org/2/library/urlparse.html)+正则的方式

```
import re
from urlparse import urlparse

topHostPostfix = (
    '.com','.la','.io','.co','.info','.net','.org','.me','.mobi',
    '.us','.biz','.xxx','.ca','.co.jp','.com.cn','.net.cn',
    '.org.cn','.mx','.tv','.ws','.ag','.com.ag','.net.ag',
    '.org.ag','.am','.asia','.at','.be','.com.br','.net.br',
    '.bz','.com.bz','.net.bz','.cc','.com.co','.net.co',
    '.nom.co','.de','.es','.com.es','.nom.es','.org.es',
    '.eu','.fm','.fr','.gs','.in','.co.in','.firm.in','.gen.in',
    '.ind.in','.net.in','.org.in','.it','.jobs','.jp','.ms',
    '.com.mx','.nl','.nu','.co.nz','.net.nz','.org.nz',
    '.se','.tc','.tk','.tw','.com.tw','.idv.tw','.org.tw',
    '.hk','.co.uk','.me.uk','.org.uk','.vg', ".com.hk")

regx = r'[^\.]+('+'|'.join([h.replace('.',r'\.') for h in topHostPostfix])+')$'
pattern = re.compile(regx,re.IGNORECASE)

print "--"*40
for url in urls:
    parts = urlparse(url)
    host = parts.netloc
    m = pattern.search(host)
    res =  m.group() if m else host
    print "unkonw" if not res else res
```

运行结果如下:

```
meiwen.me
1000chi.com
see.xidian.edu.cn
python.org
google.com.hk
unkonw
mongodb.org
python.org
127.0.0.1:8000
```

基本可以接受

## [urllib](https://docs.python.org/2/library/urllib.html)来解析域名

```
import urllib

print "--"*40
for url in urls:
    proto, rest = urllib.splittype(url)
    res, rest = urllib.splithost(rest)
    print "unkonw" if not res else res
```

运行结果如下：

```
meiwen.me
1000chi.com
see.xidian.edu.cn
docs.python.org
www.google.com.hk
unkonw
api.mongodb.org
pypi.python.org
127.0.0.1:8000
```

会把www.也带上，还需要进一步解析才可以

## 使用第三方模块  [tld](https://pypi.python.org/pypi/tld)

```
from tld import get_tld

print "--"*40
for url in urls:
    try:
        print  get_tld(url)
    except Exception as e:
        print "unkonw"
```

运行结果：

```
meiwen.me
1000chi.com
xidian.edu.cn
python.org
google.com.hk
unkonw
mongodb.org
python.org
unkonw
```

结果都可以接受


## 其他可以使用的解析模块：
*  [tld](https://pypi.python.org/pypi/tld)
*  [tldextract](https://pypi.python.org/pypi/tldextract)
*  [publicsuffix](https://pypi.python.org/pypi/publicsuffix/)
