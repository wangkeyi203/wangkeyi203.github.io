---
title: cockpit学习_2
date: 2020-07-21 09:26:25
tags:
- cockpit
---

**cockpit管理多台主机**

首先在A机器安装cockpit-dashboard这个插件包。然后web重新登陆，就可以看到dashboard的选项。

在B机器安装cockpit包，然后设置开机启动cockpit服务

```
systemctl enable --now cockpit.socket
```

如果要做免密登陆，要在A机器执行下面命令

```
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa.pub B机IP
```

然后在web界面的dashboard页面里选择添加机器，输入B机IP，就能在A的web页面管理B机器。若没有配置密钥，则需要输入B机器的账号和密码。

**cockpit管理容器**

cockpit管理容器使用的是podman，所以需要先安装`cockpit-podman`这个插件包，然后刷新web页面，就可以看到Podman Containers的选项。

在这个标签页里，可以直接搜索镜像，拉取下来之后可以直接启动。

**cockpit管理虚拟机**

需要先安装`cockpit-machines`这个插件包，然后刷新web页面，就可以看到虚拟机的标签页。

**cockpit插件**

cockpit使用XDG规范来存放插件包，默认情况下，cockpit按以下顺序在这几个目录里查找插件包，遇到同名的则使用找到的第一个。

```
~/.local/share/cockpit/

/usr/local/share/cockpit/

/usr/share/cockpit/
```

可以使用下面这条命令查看系统中安装的cockpit插件包

```
$ cockpit-bridge --packages
```

一个插件包包含以下几个文件

```
/usr/share/cockpit/my-package/
        manifest.json
        file.html
        some.js
```

manfiest.json中一般包含以下几个字段

```
content-security-policy
name
priority
requires
version
preload
dashboard
menu
tools
```

 其中menu是由以下json对象注册的

```
label
order
path
docs
keywords
```

keywords又是由以下json注册的

```
matches
goto
weight
translate
```

一个manifest.json文件例子

```json
{
  "version": 0,
  "require": {
      "cockpit": "120"
  },
  "tools": {
     "mytool": {
        "label": "My Tool",
        "path": "tool.html"
     }
  }
}
```

为了压缩空间，插件包支持压缩格式，加入一个文件名为`test.de.js`,如果cockpit没有在找到该文件，将会以以下的扩展名寻找这个文件。

```
mypackage/test.de.js
mypackage/test.de.min.js
mypackage/test.de.js.gz
mypackage/test.de.min.js.gz
mypackage/test.js
mypackage/test.min.js
mypackage/test.js.gz
mypackage/test.min.js.gz
```

