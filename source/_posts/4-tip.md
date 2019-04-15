---
title: tar的追加与取出文件
date: 2019-04-14 23:55:38
tags:
- tip
- 第四周
---

追加用“-r"选项，取出用“-x”选项：
如：有一个包 file.tar.bz2 ,需要把file2添加进去，就用命令：
**tar -rvf file.tar.bz2 file2**
取出file2则用：
**tar -xvf file.tar.bz2 file2**
