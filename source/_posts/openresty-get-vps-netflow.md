title: OpenResty查询Linux VPS主机流量
comments: true
toc: true
date: 2016-09-23 18:52:36
tags: [vps,openresty]
categories: [Linux,折腾]
---

<!-- more -->

> 事情是这样的，我买了vps，每个月流量不是很多，我就想看看用了多少流量。但是我又不想去主机的后台查，我还不想用那些监控软件，优点小题大做了，于是我就想弄个脚本，然后openresty读出来，我没事看眼。

## 获取流量数据
vps的操作系统是Centos6，用shell命令或者是读取某个文件获取

可以读下面两个文件，venet0是网卡名
```
[root@CT1391 ~]# cat /sys/class/net/venet0/statistics/rx_bytes
2300558468
[root@CT1391 ~]# cat /sys/class/net/venet0/statistics/tx_bytes
2111210364
```

在lua中我们就可以直接读取文件并得到网卡的流量值了，剩下的就是格式化并输出就好了。

## lua脚本
lua读取网卡数据并显示，还需要一个每月清零的动作（一般都是重启网卡，没有找到其他法子，麻烦点的方法就是每个月初记录下来，自己计算当月)。

nginx配置部分
```
server {
    listen       80;
    server_name  localhost;
    lua_code_cache off;

		location /status {
			default_type text/html;
			charset utf-8;
			content_by_lua_file /etc/nginx/lua/tstatus.lua;
		}
}
```

部署的时候记得把 `lua_code_cache` 开启了。

tstatus.lua

```lua
local io = require "io"
local math = require "math"

-- 改成自己的网卡和总量
local fname = "eth2"
local month_flow = "250G"

local function get_value(v)
	local file = io.open("/sys/class/net/"..fname.."/statistics/"..v.."_bytes")
	local value = file:read()
	file:close()
	return value
end

local function round(num, dip)
	return tonumber(string.format("%."..(dip or 0).."f", num))
end

local function flow_format(v)
	local v = tonumber(v)
	if v < 1024 then
		return v.."byte"
	elseif v < 1024*1024 then
		return round(v/1024.0, 2).."Kb"
	elseif v < 1024*1024*1024 then
		return round(v/1024.0/1024.0, 2).."M"
	elseif v < 1024*1024*1024*1024 then
		return round(v/1024.0/1024.0/1024.0, 2).."G"
	end
end

local rx = get_value("rx")
local tx = get_value("tx")
local total = rx + tx

ngx.say("您本月可用的总流量是"..month_flow.."<br>")
ngx.say("RX:"..rx.." -> "..flow_format(rx).."<br>")
ngx.say("TX:"..tx.." -> "..flow_format(tx).."<br>")
ngx.say("Total:"..total.." -> "..flow_format(total))
ngx.exit(200)
```

## 测试

用curl测试，在浏览器中也是可以正常使用的哈，如果想要看起来高大上，可以弄一个饼图或者是仪表盘，然后用json获取流量值就行了。

```
[root@orangleliu lzz]# curl http://192.168.59.104/status
您本月可用的总流量是250G<br>
RX:3412800496 -> 3.18G<br>
TX:61802720 -> 58.94M<br>
Total:3474603216 -> 3.24G
```

先这么着，下个月1号先把累计的流量记录下来，弄个持久化和计算当月的流程吧。。
