title: Sweet Nginx-根据User Agent或者url特征阻止请求
comments: true
toc: true
date: 2016-09-25 18:53:17
tags: [nginx]
categories: [技术分享, openresty]
---

<!-- more -->
> 有时候为了解决一些安全问题需要禁止某些特定的请求，例如根据user angent来阻止垃圾请求，百度爬虫这种，或者是其他的恶意请求，还有例如根据url的特征阻止某些请求，类似的有sql注入等。如果需求比较简单，直接使用nginx配置就能解决，如果需要比较专业的应用防火墙请使用专业的waf，像libmodsec 或者 naxsi。


下面主要以禁止UA作为例子

## 使用if
nginx中的if也是做流程判断，if增强了nginx的灵活性。if会有一些坑, 请参考 [If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/)

下面是配置片段
```
server {
...
location ~ /ifua {

        # 设定一个标志
        set $block_user_agents 0;
        # 对ua进行判断
        if ($http_user_agent ~ "curl") {
           set $block_user_agents 1;
        }
        if ($http_user_agent ~ "Go-Ahead-Got-It") {
           set $block_user_agents 1;
        }
        if ($http_user_agent ~ "TurnitinBot") {
           set $block_user_agents 1;
        }
        if ($http_user_agent ~ "GrabNet") {
           set $block_user_agents 1;
        }
        # 对判断结果进行处理
        # 每个请求都会对所有规则进行判断，如果每次判断直接return 403更好
        if ($block_user_agents = 1) {
            return 403;
        }

        # 或者这么写
        if ($http_user_agent ~* (QQMusic|Agoo-sdk-2.0|anclient|QQGame|kwai-android)){
            return 403;
        }

        # SQL注入的规则可以这么写
        set $block_sql_injections 0;
        if ($query_string ~ "union.*select.*\(") {
            set $block_sql_injections 1;
        }
        if ($query_string ~ "union.*all.*select.*") {
            set $block_sql_injections 1;
        }
        if ($query_string ~ "concat.*\(") {
            set $block_sql_injections 1;
        }
        if ($block_sql_injections = 1) {
            return 403;
        }

        return 200;
    }
...
}


```


使用curl测试下, 测试使用的nginx是 1.9.x版本
```
liuzhizhi@lzz-rmbp|nginx # curl -H "User-Agent: chrome" 'http://127.0.0.1/ifua' -i
HTTP/1.1 200 OK
...


liuzhizhi@lzz-rmbp|nginx # curl -H "User-Agent: curl" 'http://127.0.0.1/ifua' -i
HTTP/1.1 403 Forbidden
...

liuzhizhi@lzz-rmbp|nginx # curl -H "User-Agent: QQMusic" 'http://127.0.0.1/ifua' -i
HTTP/1.1 403 Forbidden
...

liuzhizhi@lzz-rmbp|nginx # curl -i -H "User-Agent: chrome" 'http://127.0.0.1/ifua?name=concat('
HTTP/1.1 403 Forbidden
...

liuzhizhi@lzz-rmbp|nginx # curl -i -H "User-Agent: chrome" 'http://127.0.0.1/ifua?name=lzz'
HTTP/1.1 200 OK
...
```


## 使用map
map是一个映射的过程，符合某些条件就会映射为某一个值，感觉用map更加整洁一些。

我们可以把ua的规则放在一个单独的文件 useragent.rules, 在配置中就可以使用 `$badagent` 这个变量的值来判断ua是否匹配了。
```
map $http_user_agent $badagent {
        default              0;
        ~curl                1;
        ~Go-Ahead-Got-It     1;
        ~TurnitinBot         1;
        ~GrabNet             1;
        ~*(QQMusic|Agoo-sdk-2.0|anclient|QQGame|kwai-android) 1;
}
```

nginx配置，注意include的位置。
```
http{
...
    # map只能在http块中声明
    include useragent.rules;

    server {
        ...
        location ~ /mapua {
            if ($badagent){
                return 403;
            }
        return 200;
        }
    }
}

```

使用curl 带上不同的ua测试
```
liuzhizhi@lzz-rmbp|nginx # curl -i -H "User-Agent: chrome" 'http://127.0.0.1/mapua'
HTTP/1.1 200 OK
...

liuzhizhi@lzz-rmbp|nginx # curl -i -H "User-Agent: curl" 'http://127.0.0.1/mapua'
HTTP/1.1 403 Forbidden
...

liuzhizhi@lzz-rmbp|nginx # curl -i -H "User-Agent: QQMusic" 'http://127.0.0.1/mapua'
HTTP/1.1 403 Forbidden
...
```

用map这种方式看起来更整洁，个人比较喜欢。


## 小结

nginx论坛上曾有人讨论过这两种方式的性能如何 [Re: performance hit in using too many if's](https://forum.nginx.org/read.php?2,269808,269811#msg-269811), 意思是说使用map更高效一些，我自己测试下来发现几乎差不多。

不光是ua和query string这两个变量，对于nginx其他的变量也可以那么操作，例如ip地址，特定的header都可以用类似的方式灵活的阻止访问，保护我们的服务器。

## 参考

* [If Is Evil](https://www.nginx.com/resources/wiki/start/topics/depth/ifisevil/) 最早是春哥提出来的
* [nginx map文档](http://nginx.org/en/docs/http/ngx_http_map_module.html)
* [nginx: How To Block Exploits, SQL Injections, File Injections, Spam, User Agents, Etc](https://www.howtoforge.com/nginx-how-to-block-exploits-sql-injections-file-injections-spam-user-agents-etc)
* [How to block specific user agents on nginx web server](http://ask.xmodulo.com/block-specific-user-agents-nginx-web-server.html)
