---
title: cockpit学习-3
date: 2020-07-22 00:02:47
tags:
- cockpit
- cockpit.js
- cockpit.DBus Client
---

cockpit有个base1插件包，里面提供供其他插件包使用的api。

分下面几个大模块

```
cockpit.js — Basic cockpit API to interact with the system
cockpit.js: DBus Client — DBus API communication
cockpit.js: File Access — Reading, writing, and watching files.
cockpit.js: HTTP Client — HTTP and REST API communication
cockpit.js: Spawning Processes — Spawning processes or scripts
cockpit.js: Metrics — Reading and streaming metric data
cockpit.js: Series Data — Representing series data
cockpit.js: Raw Channels — Raw communication channels
cockpit.js: Page Location and Jumping — Page location and navigation between components
cockpit.js: Localization — Localization and translations
cockpit.js: Errors — Problem codes and messages
cockpit.js: User Session — User information and login session state
cockpit.js: Utilities — Various utility functions
cockpit.js: Object Cache — Caching and sharing data
cockpit.js: Manifests — Manifest info
```



**cockpit.js**  

通过在html中这样调用

```
<script src="../base1/cockpit.js">
```

**cockpit.js:DBus Client** 

这些api可以直接访问dbus服务

```
client = cockpit.dbus(name, [options])
promise = client.wait([callback])
client.close([problem])
client.addEventListener("close", options => { ... })
client.addEventListener("owner", (event, owner) => { ... })
client.options
client.unique_name
client.proxy()
client.proxy()
proxy.client
//proxy = client.proxy([interface, path], [options])
proxy.path
proxy.iface
proxy.valid
proxy.data
proxy.call()
proxy.wait()
proxy.onchanged
proxy.onsignal
client.proxies()
proxies.wait()
proxies.client
proxies.iface
proxies.path_namespace
proxies.onadded
proxies.onchanged
proxies.onremoved
client.call()
invocation.then()
invocation.catch()
client.subscribe()
subscription.remove()
client.watch()
watch.then()
watch.catch()
watch.remove()
client.onnotify
client.notify()
client.onmeta
client.meta()
client.publish()
published.remove()
cockpit.variant()
cockpit.byte_array()
```

具体说明可以参考[官方文档](https://cockpit-project.org/guide/latest/cockpit-dbus.html)