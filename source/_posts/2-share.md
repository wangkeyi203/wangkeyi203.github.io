---
title: ptrace
date: 2019-03-31 22:16:34
tags:
- share
- 第二周
---

分享一个Linux的系统调用**ptrace**。

ptrace函数的参数：

```c
long ptrace(enum __ptrace_request request,
            pid_t pid,
            void *addr,
            void *data);
```

具体使用方法可以man一下，很强大，linux下调试工具gdb主要使用的就是ptrace。ptrace可以实现对另一进程的断点调试和跟踪系统调用，同样我们也可以使用这个调用来做反调试，防止我们的进程被跟踪调试，感兴趣的可以自己看一下ptrace的man手册。