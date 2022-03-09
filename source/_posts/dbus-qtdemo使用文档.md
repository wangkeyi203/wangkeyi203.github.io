---
title: dbus_qtdemo使用文档
date: 2022-02-24 16:44:53
tags:
- dbus
- qt
---

## dbus替代sudo NOPASSWD的qt方案示例说明

撰写人：系统安全组

此文档提供一种使用dbus的机制来替代系统某些脚本权限过高的问题，我们系统中对一 些管理脚本赋予了sudo的NOPASSWD权限，如果脚本被修改执行，那么脚本可以直接 获取root权限，这个存在很大的安全隐患。 现在使用dbus写了一个demo，demo分为服务端和客戶端，客戶端相当于之前系统中的 管理脚本，只有普通用戶权限，将需要root权限的操作放到server端里去执行。

## 服务端

### 实现简介

qt demo的模拟场景一个是需要root权限执行 autitcl命令时，有客户端发送method call 来请求，由服务端来执行具体的操作，以及一个发送signal的演示

### 代码编译

需要安装的开发包libdbus-1-dev及需要移动一下头文件:

$ sudo apt install libdbus-1-dev -y
$ sudo cp /usr/lib/x86_64-linux-gnu/dbus-1.0/include/dbus/dbus-arch-deps.h /usr/include/dbus-1.0/dbus/

服务端编译

服务端代码是由c写的，进入c_service/编译service代码

```
wky@wky-pc:~/qt$ cd c_service/
wky@wky-pc:~/qt/c_service$ make
```

移动编译好的demo程序到/usr/bin/: 

wky@wky-pc: sudo cp demo /usr/bin/

### 服务端配置

#### 1conf配置

因为使用的是system bus，我们还需要给配置一下接口的conf文件，创建一个/etc/dbus-1/system.d/test1.conf内容如下:

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

此文件中几个关键的配置说明:

 system dbus 服务的bus类型，分为system和session，我们要垮用戶通信，就需要用 system总线 unix:tmpdir=/tmp 设置监听地址

 dbus service文件目录，在/usr/share/dbus-1/system-services下 配置dbus的一些策略

主要是注册com.dbus.receiver_app，send_interface和send_destination为了方便 调试我这里都是用*。

#### 2 服务自启动配置

为了使服务端随着系统启动就进行自启动，需要配置/usr/share/dbus-1/system-services/com.example.SampleService.service文件，示例如下: 在/usr/share/ dbus-1/system-services/下创建一个com.dbus.receiver_app.service

```
wky@wky-pc:~$ cat /usr/share/dbus-1/system-services/com.dbus.receiver_app.service
[D-BUS Service]
Name=com.dbus.receiver_app
Exec=/usr/bin/demo receive
User=root
```

这个文件是用来给dbus服务程序做按需启动的，当客戶端发起请求时，会根据 test1.conf中的 在/usr/share/dbus-1/system-services/找到对应的service文件，来 启动dbus的服务程序。

 Name=com.dbus.receiver_app 这个跟代码里服务程序的bus name对应起来。 Exec=/usr/bin/demo receive 这个为启动服务程序的方式。

 User=root 启动服务程序的用戶，因为我们是为了替代sudo nopasswd权限滥用问 题，服务端就用root就可以。

### 客户端

编译客户端，进入到src目录中编译qt代码

wky@wky-pc:~/qt$ cd src/

wky@wky-pc:~/qt/src$ make

注：新的qt工程如果要用dbus接口，可能需要在Makefile中的INCPATH参数后面补充上 -I/usr/lib/x86_64-linux-gnu/qt5/mkspecs/linux-g++ -I/usr/include/dbus-1.0 -I/usr/lib/x86_64-linux-gnu/dbus-1.0/include -ldbus-1

然后执行 ./dbus 启动qt客户端

客户端一共两个按钮，signal按钮是向服务端发送一个信号，信号的信息填写到  路径 的textedit框中就可以发送给服务端

如果是手动执行的服务端程序就可以看到服务端打印的信息。

主要功能是第一个按钮 method call，是模拟执行下面这条命令

auditctl -w /tmp/111 -p wax -k testaudit

这条命令普通用户权限是无法执行的，只有root用户有权限

路径 中填入 /tmp/111

权限 中填入 wax

key值 中填入 testaudit

点击method call ，效果如下，如果服务端运行成功，则返回一个sucess

![success](/Users/wky/Desktop/work/wangkeyi203.github.io/source/_posts/dbus-qtdemo使用文档/success.png)

此时用root用户执行下面命令可以看到规则已经添加

```
root@wky-pc:/home/wky/qt/src# auditctl -l
-w /tmp/111 -p wxa -k testaudit
```

这时候如果再点击method call ，因为规则已经存在，服务端执行命令失败，则返回一个false

![false](/Users/wky/Desktop/work/wangkeyi203.github.io/source/_posts/dbus-qtdemo使用文档/false.png)

可以使用root用户执行 auditctl -D 清空规则，再次执行就可以成功

