title: 【Py之旅】Python Shell  MD5使用的那些事
comments: true
toc: true
date: 2015-11-27 18:40:17
tags: [MD5]
categories: [密码学]
---

<!-- more -->
> MD5 应该是用的非常多的算法，就自己使用经验说说吧。

## 场景
算法层面不多说了，[维基百科](https://en.wikipedia.org/wiki/MD5)，还有很多文章都有说明。

主要用过的场景

*  密码存储，现在基本没怎么有使用的了，毕竟破解容易了很多
*  API校验，现在使用的也蛮多的，API双方都有一个私有key，把数据和key放到一起生成token，两边校验（注意的一点是对于unicode编码，一定要encode）
*  文件校验，这个用的还挺多，大家不要总是忘了这一步，`Xcode植入后门就是教训`

## 用法
尽量的列出使用过的方式，多个方式可以相互印证

### 字符串MD5

#### Shell 方式
一般都会有 `md5sum` 命令(Centos)

```
# echo -n "sweet girl"|md5sum
74417797d6a7200192978659effa5e2d -
```

或者使用openssl命令

```
# echo -n "sweet girl"|openssl md5
(stdin)= 74417797d6a7200192978659effa5e2d
```

#### Python
代码比较短，这里写成命令方式

```
# python -c "import hashlib; print hashlib.md5('sweet girl').hexdigest()"
74417797d6a7200192978659effa5e2d
```

#### Lua

Lua中没有标准的实现，不过C拓展和纯Lua版本也容易找到，`LuaJIT` 推荐使用春哥的 [lua-resty-string](https://github.com/openresty/lua-resty-string) 模块，基于FFI，很快


### 文件MD5
文件校验使用的比较多的还是文件下载的校验

#### Shell 方式

```
# md5sum printf.lua
629fc9a9c1b1debd24e162d817f4e4a7 printf.lua
```

openssl

```
# openssl md5 printf.lua
MD5(printf.lua)= 629fc9a9c1b1debd24e162d817f4e4a7
```

#### Python
需要一块一块的读取

```
#coding:utf-8
#@orangleliu
#fname: md5file.py

import hashlib
def md5_for_file(path, block_size=256*128, hr=True):
    md5 = hashlib.md5()
    with open(path,'rb') as f:
        for chunk in iter(lambda: f.read(block_size), b''):
             md5.update(chunk)
    if hr:
        return md5.hexdigest()
    return md5.digest()

if __name__ == "__main__":
    print md5_for_file("printf.lua")
```

使用
```
# python md5file.py printf.lua
629fc9a9c1b1debd24e162d817f4e4a7
```



