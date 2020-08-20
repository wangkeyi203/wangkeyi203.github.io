---
title: config去掉空格
date: 2020-08-20 16:32:17
tags:
- python
- configobj
---

`configobj`管理配置文件会在=两边添加空格，会导致有的服务起不来。

在configobj.py中

self._a_to_u(' = ')

改为

self._a_to_u('=')就可以解决