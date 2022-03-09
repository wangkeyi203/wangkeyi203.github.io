---
title: dbus配置说明
date: 2022-02-23 15:54:39
tags:
- dbus服务自启动
---

以c的dbus demo代码为例子，做一个自启动的dbus服务配置

首先c demo的源码目录下有一个 test1.conf文

```
<!-- Bus that listens on a debug pipe and doesn't create any restrictions -->

<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>
  <type>system</type> 

  <listen>unix:tmpdir=/tmp</listen>

  <standard_system_servicedirs />

  <policy context="default">
    <!-- Allow everything to be sent -->
    <allow send_destination="*" eavesdrop="true"/>
    <!-- Allow everything to be received -->
    <allow eavesdrop="true"/>
    <!-- Allow anyone to own anything -->
    <allow own="*"/>
    <allow user="*"/>
  </policy>

</busconfig>
```

先将这个文件放到 /etc/dbus-1/system.d/ 下，几个关键的配置说明

<type>system</type>    dbus 服务的bus类型，分为system和session，我们要垮用户通信，就需要用system总线

<listen>unix:tmpdir=/tmp</listen>  设置监听地址

<standard_system_servicedirs />  dbus service文件目录，在/usr/share/dbus-1/system-services下

 <policy context="default"> 配置dbus的一些策略

然后需要在/usr/share/dbus-1/system-services/下创建一个 com.dbus.receiver_app.service

```
cat /usr/share/dbus-1/system-services/com.dbus.receiver_app.service
[D-BUS Service]
Name=com.dbus.receiver_app
Exec=/usr/bin/demo receive
User=root
```

这个文件是用来给dbus服务程序做按需启动的，当客户端发起请求时，会根据 test1.conf中的 <standard_system_servicedirs /> 在/usr/share/dbus-1/system-services/找到对应的service文件，来启动dbus的服务程序

Name=com.dbus.receiver_app 这个跟代码里服务程序的bus name对应起来

Exec=/usr/bin/demo receive 这个为启动服务程序的方式

User=root    启动服务程序的用户，因为我们是为了替代sudo nopasswd权限滥用问题，服务端就用root就可以

然后把c的源码编译一下，直接make一下，然后将编译好的demo复制一份到/usr/bin/下

再用普通用户执行 

```
wky@wky-pc:~/C_dbus$ ./demo send METHOD INT32 99
[59355] Got Method Return INT32: 99
```

可以获取到服务端返回的返回值

```
wky@wky-pc:~/C_dbus$ ps aux |grep demo
root       9584  0.0  0.0   5008  1100 ?        S    14:21   0:00 /usr/bin/demo receive
wky       59389  0.0  0.0   9152   888 pts/2    S+   17:24   0:00 grep demo
```

使用ps查看一下，也可以看到服务端已经以root权限启动了
