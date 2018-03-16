title: Openresty generates QR code
comments: true
toc: true
date: 2017-03-24 13:45:11
tags: [openresty]
categories: [技术]
---

> I would like to use openresty(ngx_lua) to do QRcode generation API, then I found two libraries [lua-resty-QRcode](https://github.com/dcshi/lua-resty-QRcode) and [qrencode](https://github.com/vincascm/qrencode). After comparison，[qrencode](https://github.com/vincascm/qrencode) is simple to install and use, here is an example of hope that interested friends have inspired

Both of these libraries are dependent [libqrencode](http://fukuchi.org/works/qrencode/) and [libpng](http://www.libpng.org/pub/png/libpng.html) 

Steps for usage:
* install [libqrencode](http://fukuchi.org/works/qrencode/) and [libpng](http://www.libpng.org/pub/png/libpng.html) 
* get qrencode, and compiled into a dynamic library `qrencode.so`
* Openresty lua call dynamic library to generate qrcode 

openresty1.9.7  was installed on my Mac book.


## install libqrencode，libpng

you can install them with source code, or following methods

### Ubuntu 

```
sudo apt-get install libqrencode-dev libpng12-dev
```

### CentOS7 


```
yum install libpng-devel

wget http://ftp.riken.jp/Linux/centos/7/os/x86_64/Packages/qrencode-devel-3.4.1-3.el7.x86_64.rpm
rpm -ivh qrencode-devel-3.4.1-3.el7.x86_64.rpm 
```

### MacOS

```
brew install libqrencode
```
will auto install libpng.


## install lua library qrencode
！！For the use of openresty I rewrite the makefile, you can find  https://github.com/orangle/lua-resty-qrencode , the Makefile default for Macos, if you use other os like centos, please read Makefile and modify it.

manual version is below

```
git clone https://github.com/vincascm/qrencode.git

## on macos 
gcc -bundle -undefined dynamic_lookup -lpng -lqrencode -I/usr/local/openresty/luajit/include/luajit-2.1/ qrencode.c -o qrencode.so
cp test/test.lua ./
/usr/local/openresty/luajit/bin/luajit test.lua
```

then you can see some output on your screen.


## openresty qrcode code 

copy qrencode.so to openresty's lualib directory, or you can set lualib path in nginx conf file.

```
cp qrencode.so  /usr/local/openresty/lualib/
```

lua code in nginx conf
```
location /qrcode {
        content_by_lua_block {
            local qr = require("qrencode")
            local args = ngx.req.get_uri_args()
            local text = args.text
            
            if text == nil or text== "" then
                ngx.say('need a text param')
                ngx.exit(404)
            end
            
            ngx.say(qr {
                text=text,
                level="L",
                kanji=false,
                ansi=true,
                size=4,
                margin=2,
                symversion=0,
                dpi=78,
                casesensitive=true,
                foreground="48AF6D",
                background="3FAF6F"
            })
        }
    }
```

using curl to testing

```
curl 'http://127.0.0.1:8008/qrcode?text=http://orangleliu.info'
```
then you see a qrcode picture, test is ok. 


![openresty 二维码](http://img.blog.csdn.net/20170322131251751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvb3JhbmdsZWxpdQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)