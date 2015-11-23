title: Docker初次接触
comments: true
toc: true
date: 2014-08-18 17:04:00
tags: [docker]
categories: [翻译]
---

<!-- more -->

最近看了不少docker介绍性文章，也听了不少公开课，于是今天去官网逛了逛，发现了一个[交互式的小教程](https://www.docker.com/tryit/)于是决定跟着学习下。只是把觉得重点的知识记录下来，不是很系统的学习和笔记。

### 理论部分
*  **Docker 引擎**包含了两个部分，一个守护进程作为服务器端来管理所有的容器。一个客户端，可以远程来控制服务端。
*  Docker有公共的云端仓库 Docker Hub Registry，里面有可以使用的镜像
*  你可以认为容器**containers**就是沙箱**box**中的一个进程。这个盒子中包括了所有一个进程需要的东西，文件系统，系统库，shell等等，只是默认情况下他们是没有运行的。
*  我们可以在操纵和改变容器，然后通过命令保存成新的镜像
*  在需要使用容器id的地方，我们可以只输入前几个字符


### 操作部分
docker help 可以查看能使用的命令和简单描述

#### 查看版本

    you@tutorial:~$ docker version
    Docker Emulator version 0.1.3
    Emulating:
    Client version: 0.5.3
    Server version: 0.5.3
    Go version: go1.1

#### 从公共云仓库中查找一个镜像 tutorial

    you@tutorial:~$ docker search tutorial
    Found 1 results matching your query ("tutorial")
    NAME                      DESCRIPTION
    learn/tutorial            An image for the interactive tutorial

#### 从仓库中拉取一个镜像，注意要写这个镜像的全名

    you@tutorial:~$ docker pull learn/tutorial
    Pulling repository learn/tutorial from https://index.docker.io/v1
    Pulling image 8dbd9e392a964056420e5d58ca5cc376ef18e2de93b5cc90e868a1bbc8318c1c (precise) from ubuntu
    Pulling image b750fe79269d2ec9a3c593ef05b4332b1d1a02a62b4accb2c21d589ff2f5f2dc (12.10) from ubuntu
    Pulling image 27cf784147099545 () from tutorial

#### 启动一个docker并且运行命令

    you@tutorial:~$ docker run learn/tutorial echo "hello boy"
    hello boy

tips：这里是启动了一个容器并且运行了一个命令，当命令运行完的时候容器就停止了。可以通过docker的ps命令查看当前正在运行的容器。

    you@tutorial:~$ docker ps
    ID                  IMAGE               COMMAND               CREATED             STATUS              PORTS

#### 安装一个软件ping

    you@tutorial:~$ docker run learn/tutorial apt-get install -y ping
    Reading package lists...
    Building dependency tree...
    The following NEW packages will be installed:
      iputils-ping
    0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
    Need to get 56.1 kB of archives.
    After this operation, 143 kB of additional disk space will be used.
    Get:1 http://archive.ubuntu.com/ubuntu/ precise/main iputils-ping amd64 3:20101006-1ubuntu1 [56.1 kB]
    debconf: delaying package configuration, since apt-utils is not installed
    Fetched 56.1 kB in 1s (50.3 kB/s)
    Selecting previously unselected package iputils-ping.
    (Reading database ... 7545 files and directories currently installed.)
    Unpacking iputils-ping (from .../iputils-ping_3%3a20101006-1ubuntu1_amd64.deb) ...
    Setting up iputils-ping (3:20101006-1ubuntu1)

#### 查看改变之后的容器，然后保存成learn/ping

    you@tutorial:~$ docker ps -l
    ID                  IMAGE               COMMAND                CREATED             STATUS              PORTS
    6982a9948422        ubuntu:12.04        apt-get install ping   1 minute ago        Exit 0
    you@tutorial:~$ docker commit 6982 learn/ping
    effb66b31edb

#### 使用新的镜像ping  google

    you@tutorial:~$ docker run learn/ping ping www.google.com
    PING www.google.com (74.125.239.129) 56(84) bytes of data.
    64 bytes from nuq05s02-in-f20.1e100.net (74.125.239.148): icmp_req=1 ttl=55 time=2.23 ms
    64 bytes from nuq05s02-in-f20.1e100.net (74.125.239.148): icmp_req=2 ttl=55 time=2.30 ms
    64 bytes from nuq05s02-in-f20.1e100.net (74.125.239.148): icmp_req=3 ttl=55 time=2.27 ms
    64 bytes from nuq05s02-in-f20.1e100.net (74.125.239.148): icmp_req=4 ttl=55 time=2.30 ms

#### 查看正在运行的容器的状态

    you@tutorial:~$ docker ps
    ID                  IMAGE               COMMAND               CREATED             STATUS              PORTS
    efefdc74a1d5        learn/ping:latest   ping www.google.com   37 seconds ago      Up 36 seconds
    you@tutorial:~$ docker inspect efef
    [2013/07/30 01:52:26 GET /v1.3/containers/efef/json
    {
      "ID": "efefdc74a1d5900d7d7a74740e5261c09f5f42b6dae58ded6a1fde1cde7f4ac5",
      "Created": "2013-07-30T00:54:12.417119736Z",
      "Path": "ping",
      "Args": [
          "www.google.com"
      ],
      "Config": {
          "Hostname": "efefdc74a1d5",
          "User": "",
          "Memory": 0,
          "MemorySwap": 0,
          "CpuShares": 0,
          "AttachStdin": false,
          "AttachStdout": true,
          "AttachStderr": true,
          "PortSpecs": null,
          "Tty": false,
          "OpenStdin": false,
          "StdinOnce": false,
          "Env": null,
          "Cmd": [
              "ping",
              "www.google.com"
          ],
          "Dns": null,
          "Image": "learn/ping",
          "Volumes": null,
          "VolumesFrom": "",
          "Entrypoint": null
      },
      "State": {
          "Running": true,
          "Pid": 22249,
          "ExitCode": 0,
          "StartedAt": "2013-07-30T00:54:12.424817715Z",
          "Ghost": false
      },
      "Image": "a1dbb48ce764c6651f5af98b46ed052a5f751233d731b645a6c57f91a4cb7158",
      "NetworkSettings": {
          "IPAddress": "172.16.42.6",
          "IPPrefixLen": 24,
          "Gateway": "172.16.42.1",
          "Bridge": "docker0",
          "PortMapping": {
              "Tcp": {},
              "Udp": {}
          }
      },
      "SysInitPath": "/usr/bin/docker",
      "ResolvConfPath": "/etc/resolv.conf",
      "Volumes": {},
      "VolumesRW": {}

#### 查看本地镜像

    you@tutorial:~$ docker images
    ubuntu                latest              8dbd9e392a96        4 months ago        131.5 MB (virtual 131.5 MB)
    learn/tutorial        latest              8dbd9e392a96        2 months ago        131.5 MB (virtual 131.5 MB)
    learn/ping            latest              effb66b31edb        10 minutes ago      11.57 MB (virtual 143.1 MB)

#### 把镜像推送到云端

    docker inspect efe

tips： 这个是推送到docker的云仓库的，会有你自己的独立命名控件，账号[注册地址](https://hub.docker.com/)


这个十分钟交互式的短教程可以让我们对docker有个感性的认识，最基本的使用和激发兴趣。
