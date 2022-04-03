---
title: 通过FRP和LAN服务器实现Minecraft联机
date: 2020-07-03 10:06:54
tags:
    - 游戏
    - 捣腾
    - 杂项
---

# 前言

当时成功联机然后延迟还很低的时候确实震撼我妈。

最近碰上有朋友指名要用MC局域网联机，对面在看什么Sakura FRP的教程的时候我寻思，我最近不是在捣腾FRP吗，就试着操作了一下。

技术方面肯定有CS经验的人比我懂的不知道多到哪去了，我就不班门弄斧了。主要还是讲一下如何实施，记录一下。
<!-- more -->
# 准备

首先你要准备这么几样东西：

* 朋友
* 一台有公网IP的服务器
* 正版Minecraft账号
* 最好是有一点linux经验
* 主机端的网络不能太差

# 整活

## 服务器端

在[这里](https://github.com/fatedier/frp/releases)找找适合你服务器的frp版本然后下载并安装，我的服务器是x86架构64位的Ubuntu，所以在SSH执行：

`wget https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz`

之后解压。

`tar -zxvf frp_0.33.0_linux_amd64.tar.gz`

等会还要回来开放端口的所以先说主机端。

## 主机端

谁想当主机谁就是主机端，最好是网络条件更好的人当主机端。

主机端也要下载frp，在上面的那个链接里面找到【frp_0.33.0_windows_amd64.zip】下载并解压。

打开Minecraft创建地图，并在ESC选单里打开【对局域网开放】

聊天窗里会自动发送一条格式为【本游戏已在端口XXXXX上开启】的信息，记下端口。

打开解压好的frp目录，编辑frpc.ini。

打开后的frpc配置文件里有如下内容：

```
[common]
server_addr = 127.0.0.1
server_port = 7000

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 6000
```

修改`[common]`中的`server_addr`为你自己的公网服务器IP。
并在`[ssh]`下空一行，键入如下内容：

```
[Minecraft]
type = tcp
local_ip = 127.0.0.1
local_port = [你刚才记下的端口]
remote_port = [选一个你喜欢的端口]
```

然后回到服务器，开放那个你喜欢的端口，具体开放端口方式可以参考你租的服务器的文档。

在服务端运行frps。

```
cd frp_0.33.0_linux_amd64.tar.gz
nohup ./frps -c ./frps.ini
```

回到主机端，打开控制台，进入你的frp目录，运行frpc。我这里frp解压到了D盘，所以运行如下指令。

```
d:
cd frp_0.33.0_windows_amd64
frpc.exe
```

确定控制台中显示了`start proxy success`的信息之后保持控制台开启。

## 客户端

其他的玩家通过【你的公网服务器IP:你喜欢的端口】就可以进入你的局域网Minecraft服务器了。

# 后言

这种方法可以算是有公网服务器的人最简单（？）的开服方式了，当然缺点就是只有你在玩的时候打开了LAN服务器和frpc别人才可以进。如果理想的话能用个树莓派挂个服务器当然是最方便了。用同样的方式做一次内网穿透，其他人就可以在你电脑没有hosting服务器的时候也可以玩了。而且如果mod多的话用PC开LAN服务器还是有点卡的。
