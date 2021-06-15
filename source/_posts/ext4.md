---
title: ext4
date: 2020-12-10 17:03:22
tags:
- ext4
- metadata_csum
---

使用metadata_csum

mkfs.ext4 -O metadata_csum,64bit /dev/sda

有的内核不支持，报错，可以去掉metadata_csum

```
e2fsck -f /dev/sda
tune2fs -O -metadata_csum /dev/sda
e2fsck -f /dev/sda
```