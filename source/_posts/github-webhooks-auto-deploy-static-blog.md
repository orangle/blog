title: 利用github webhook 结合openresty自动更新静态博客
comments: true
toc: true
date: 2016-09-19 13:26:04
tags: [github, git]
categories: [技术分享]
---

<!-- more -->
> 使用hexo在github pages上弄了一个静态博客，后来觉得访问有点慢，于是放到自己vps上。

对于静态博客的部署非常简单，本来就是html，js，css等静态文件，只要nginx上配置下目录就可以正常访问了。 麻烦的是博客更新的时候，还要去vps上操作更新git pull操作，如果每次在本地commit之后，github仓库能够自动更新到vps上多好啊，于是就用到了webhook的功能。（这里静态文件的生成还是在本地，只是把生成好的静态文件push到github了，所以自动部署没有构建的环节)

## 部署静态博客
代码clone到vps上的目录，然后配置nginx

```
 server {
        listen      80;
        server_name  xxx.info ;

        location / {
            alias  /www/orangle.github.io/;
            index  index.html index.htm;
        }
}
```

注意下权限问题 ，配置完成`reload nginx` 就能访问了。



## 自动更新shell脚本

从github拉取代码，然后强制更新所有内容。

git_update.sh

```

#! /bin/bash

blog_dir=/path/to/git/repository
git=/usr/bin/git
branch=master

cd $blog_dir
$git reset --hard origin/$branch
$git clean -f
$git pull

dtime=`date`
echo "success $dtime"  > d.txt
```

##  配置webhook

在github上配置这个项目的webhook，然后在openresty中写一个http接口来出来每次github发来的 `push事件`。

[webhook的设置文档](https://developer.github.com/webhooks/) ，这里设置成  http://xxx.info/hook



修改nginx配置
```
   server {
        listen       80;
        server_name  xxx.info www.xxx.info;
        lua_code_cache on;

        location / {
            alias  /www/orangle.github.io/;
            index  index.html index.htm;
        }

        location /hook {
            content_by_lua_file /etc/nginx/hook.lua;
        }
   }
```



lua脚本
```
local signature = ngx.req.get_headers()["X-Hub-Signature"]
local key = "xxxxxx" --github上配置的相同
if signature == nil then
    return ngx.exit(404)
end

-- 校验header
ngx.req.read_body()
local t = {}
for k, v in string.gmatch(signature, "(%w+)=(%w+)") do
    t[k] = v
end


local str = require "resty.string"
local digest = ngx.hmac_sha1(key, ngx.req.get_body_data())
if not str.to_hex(digest) == t["sha1"] then
    return ngx.exit(404)
end

-- 执行更新
os.execute("bash /www/blog_update.sh");
ngx.say("OK")
ngx.exit(200)
```

[参考文章](http://blog.liaol.net/2015/06/use-github-webhooks-to-deploy-hexo/)
