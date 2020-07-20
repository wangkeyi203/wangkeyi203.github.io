---
title: cockpit学习_1
date: 2020-07-20 18:26:39
tags:
- share
---

cockpit的配置文件在`/etc/cockpit/cockpit.conf` ，默认不存在，需要用户自己创建。

**cockpit的主要模块**

`cockpit-ws` 用来和浏览器以及各个系统组件通信的服务。

`cockpit-tls`为` cockpit-ws`的代理，证书存放于`/etc/cockpit/ws-certs.d`，可以使用 `remotectl certificate` 检查将会使用的证书。

`cockpit-bridge`用来处理从web前端到服务端的消息和命令，并代表web用户生成进程。

cockpit可以配置` /etc/systemd/system/cockpit.socket.d/listen.conf` 中配置监听的端口和ip（官方手册）。实际系统中配置监听端口和ip的配置文件为`/usr/lib/systemd/system/cockpit.socket`。可以配置同时监听多个端口，但是需要在firewall中将修改的对应端口开放。`firewall-cmd --add-port=9091/tcp`。

```
[Socket]
ListenStream=9090
ListenStream=9091
ListenStream=
```

cockpit可以同时管理和监视多个服务，但是浏览器只会连接一台机器的`cockpit-ws`服务,然后通过这个服务使用ssh协议连接其他机器。cockpit的工作原理如下图。

![avatar](https://raw.githubusercontent.com/cockpit-project/cockpit/master/doc/cockpit-transport.png)

