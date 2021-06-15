---
title: 查看dd进度
date: 2020-12-24 09:52:04
tags:
- dd
- watch
---

使用dd刻盘时查看dd进度

sudo watch -n 5 killall -USR1 dd

growisofs -dvd-compat -speed=4 -Z /dev/sr0=test.iso