---
title: dbus_c
date: 2022-01-24 18:11:29
tags:
- c
- dbus
---

## C dbus开发

### 发送消息

dbus错误结构体

```
DBusError error;
dbus_error_init(&error); 
```

连接到dbus总线

```
DBusConnection* connection = dbus_bus_get(DBUS_BUS_SYSTEM, &error);
```

如果要用system bus，第一个参数就是DBUS_BUS_SYSTEM，如果要用session bus，那么第一个参数就是DBUS_BUS_SESSION

注册连接名称

```
int ret = dbus_bus_request_name(connection, sender.bus_name, DBUS_NAME_FLAG_REPLACE_EXISTING, &error);
```

第二个参数为 DBUS_APPLICATION sender；

接着创建完善和发送dbus消息，这个流程是发送信号的

```
DBusMessage* message = dbus_message_new_signal(receiver.object_path, receiver.interface_name, receiver.member_name);
dbus_message_iter_init_append(message, &iter);
dbus_connection_send(connection, message, &serial)
```

如果是要发送函数调用，发送流程如下

```
DBusMessage* message = dbus_message_new_signal(receiver.object_path, receiver.interface_name, receiver.member_name);
dbus_message_iter_init_append(message, &iter);
dbus_connection_send_with_reply(connection, message, &pending, DBUS_TIMEOUT_USE_DEFAULT)
```

阻塞等待函数返回结果

```
dbus_pending_call_block(pending);              
message = dbus_pending_call_steal_reply(pending);
dbus_pending_call_unref(pending);
```

提取结果

```
dbus_message_iter_init(message, &iter)
ret = dbus_message_iter_get_arg_type(&iter);
```

### 接受消息 

dbus错误结构体

```
DBusError error;
dbus_error_init(&error); 
```

连接到dbus总线

```
DBusConnection* connection = dbus_bus_get(DBUS_BUS_SYSTEM, &error);
```

注册连接名称

```
int ret = dbus_bus_request_name(connection, sender.bus_name, DBUS_NAME_FLAG_REPLACE_EXISTING, &error);
```

添加消息筛选

```
dbus_bus_add_match(connection, rule, &error);
```

rule为char*

循环接收和处理消息

```
dbus_connection_read_write(connection, 0);
message = dbus_connection_pop_message(connection);
```

分析接收到的消息

```
dbus_message_is_signal(message, self.interface_name, DBUS_MEMBER_SIGNAL)
dbus_message_is_method_call(message, self.interface_name, DBUS_MEMBER_METHOD)
```

根据这两个函数的返回值判断接收到的是一个信号还是一个函数调用请求
