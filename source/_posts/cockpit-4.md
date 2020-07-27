---
title: cockpit-4
date: 2020-07-24 17:43:15
tags:
- cockpit
- cockpit文件操作
---

cockpit可以让用户对完整的读写和查看常规文件，但是对大型文件的读写支持的不是特别好。

```
file = cockpit.file(path,
                    { syntax: syntax_object,
                      binary: boolean,
                      max_read_size: int,
                      superuser: string,
                      host: string
                    })

promise = file.read()
promise
    .then((content, tag) => { ... })
    .catch(error => { ... })

promise = file.replace(content, [ expected_tag ])
promise
    .then(new_tag => { ... })
    .catch(error => { ... })

promise = file.modify(callback, [ initial_content, initial_tag ]
promise
    .then((new_content, new_tag) => { ... })
    .catch(error => { ... })

file.watch((content, tag, [error]) => { })

file.close()
```

简单的读取功能,text1为textarea控件id

```
cockpit.file("/root/test").read()
    .then((content, tag) => {
            text1.value = content;
    })
    .catch(error => {
            result.innerHTML ="0";
    });
```

简单的写入

```
cockpit.file("/root/test").replace(content)
    .then(tag => {
    })
    .catch(error => {
       result.innerHTML ="0";
    });
```

然后调用close()将文件关闭。

