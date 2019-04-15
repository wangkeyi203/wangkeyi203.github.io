---
title: 内核定时器
date: 2019-04-14 23:55:48
tags:
- share
- 第四周
---

内核定时器可以用来在未来某一时间执行被调用程序，类似于软件中断。
结构体
```
struct timer_list {

   struct list_head entry;



   unsigned long expires;

   void (*function)(unsigned long);

   unsigned long data;



   struct tvec_base *base;

   /* ... */

};
```

