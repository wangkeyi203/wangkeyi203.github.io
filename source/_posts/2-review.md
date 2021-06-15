---
title: Linux running on the L4 microkernel
date: 2019-03-31 21:35:55
tags:
- review
- 第二周
---

news.ycombinator.com/item?id=18944165

并不算一篇文章，是L4内核一个社区的讨论，主要是讨论L4微内核的安全问题，在L4微内核上跑Linux驱动以及跑Linux应用的前景，但是L4有个弊端就是L4内核太小了，想要方便的使用，现在是需要借助其他操作系统的一些功能，最常见的就是现在的L4linux。

另外L4微内核是一种内核的代称，L4内核都具有同名内核接口，现在比较常见的有minix3和fiasco。这些讨论里也有人说苹果的t1和t2芯片上跑的也是微内核，大家也期望微内核未来能成为docker的替代品。