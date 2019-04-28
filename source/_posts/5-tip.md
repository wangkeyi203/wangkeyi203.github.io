---
title: 对deb包打patch
date: 2019-04-28 23:42:05
tags:
- tip
- 第五周
---

首先使用apt-get source xxx 下载我们想要打patch的包

再使用dpkg-source -x xxx.dsc将我们下载的源码包解压。

进入源码目录，quilt new testpatch 创建patch，假如我们需要对test.c文件进行修改，执行 quilt add test.c，然后对文件进行修改结束后，执行quilt refresh，生成patch文件，修改源码目录下的/debian/patches/series文件将我们的testpatch添加到最后一行。

回到源码的根目录，执行dpkg-buildpackage，则可以根据我们修改后的源码重新生成deb包。