title: Openwrt中利用iptables实现流量监控和portal的原理
comments: true
toc: true
date: 2016-09-23 18:24:51
tags: [openwrt,iptables]
categories: [Linux]
---

<!-- more -->

> openwrt上需要开发一些功能，中间用到的一些iptables相关的知识的整理，希望对需要实现wifi portal功能或者是流量监控的朋友可以有些启发

## 流量监控

针对每个mac来统计流量的使用情况，如果需求是统计每天的总流量这种，还需要对每次取得的结果进行持久化。可以参考  [wrtbwmon](https://github.com/pyrovski/wrtbwmon) 这个脚本来做数据收集。

```
openwrt监控某个ip
添加一个新的iptables chain
iptables -N P45

往刚才建的P45和delegate_forward添加rules，ip换成你想监控的
iptables -I P45 -s 192.168.111.115 -j ACCEPT
iptables -I P45 -d 192.168.111.115 -j ACCEPT
iptables -I forwarding_rule -s 192.168.111.115 -j P45
iptables -I forwarding_rule -d 192.168.111.115 -j P45

-I 代表chain
-s source
-d destination
-j 如果符合就

# iptables -nvL forwarding_rule|grep 192.168.111.115
19657   20M P45        all  --  *      *       0.0.0.0/0            192.168.111.115
20214 2172K P45        all  --  *      *       192.168.111.115      0.0.0.0/0

-n 显示端口号
-L 规则
-v 显示计数
```

监控本机某个IP和端口，经常做本机的某些应用的流量统计

```
iptables -I INPUT -d 45.78.37.246 -p tcp --dport 9999
iptables -I OUTPUT -s 45.78.37.246 -p tcp --sport 9999

iptables -I INPUT -d 45.78.37.246
iptables -I OUTPUT -s 45.78.37.246

iptables -nvL

删除不需要的链
iptables -n -L -v --line-numbers
iptables -D INPUT 1

```

## 白名单

非转发
```
禁止某个MAC
iptables -A INPUT -m mac --mac-source 00:0F:EA:91:04:08 -j DROP

只让某个mac访问某个端口
iptables -A INPUT -p tcp --destination-port 22 -m mac --mac-source 00:0F:EA:91:04:07 -j ACCEPT
```


### WIFI Portal 认证原理 iptables部分

OpenWrt 认证使用 Wifidog类似，终端连接上来之后真对http协议的请求进行重定向到路由的portal服务器，后面就是认证的流程，当认证完成之后就可以真对某个mac地址进行放行。
```
还没认证
iptables -F forwarding_rule
iptables -t nat -F prerouting_rule
iptables -I forwarding_rule -s 192.168.111.0/24 -j DROP
iptables -t nat -I prerouting_rule -p tcp -s 192.168.111.0/24 --dport 80 -j DNAT --to 192.168.111.1:81

白名单（终端没有认证之前可以访问的网站)
iptables -I forwarding_rule -d 115.29.23.45 -j ACCEPT
iptables -t nat -I prerouting_rule -p tcp -d 115.29.23.45 --dport 80 -j ACCEPT
iptables -I forwarding_rule -d mapi.alipay.com -j ACCEPT
iptables -t nat -I prerouting_rule -p tcp -d mapi.alipay.com --dport 80 -j ACCEPT

开启某个MAC上网
iptables -I forwarding_rule -m mac --mac-source a4:5e:60:cd:b3:d9 -j ACCEPT
iptables -t nat -I prerouting_rule -p tcp -m mac --mac-source a4:5e:60:cd:b3:d9 --dport 80 -j ACCEPT
```

* why new chain?
* -d domaian, how handler? 对于白名单使用域名，iptables会自动查询dns然后转换成ip，然后加入到规则中
* 为什么还要nat呢？
    + nat是为了让80端口http，重定向到portal
* 多个dport怎么办?
    + 有个 -m --dports 选项

    ```
    iptables -A INPUT -p tcp  --match multiport --dports 110,143,993,995 -j ACCEPT
    ```

## 参考文章
* [发个openwrt下基于iptables的Luci图形ip流量监控教程](http://www.right.com.cn/forum/thread-181287-1-1.html)
* [iptables流量统计](http://zhensheng.im/2014/04/27/2212/MIAO_LE_GE_MI)
