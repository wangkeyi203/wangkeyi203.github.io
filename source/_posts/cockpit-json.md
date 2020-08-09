---
title: cockpit_json
date: 2020-08-03 17:07:40
tags:
- cockpit
-json
---

通常配置文件分三种类型，布尔值，数值，字符串。

json格式

```json
{
  "config_name":"",
  "type":"",
  "depends":""
  "required":"",
  "conflict":""
  "value":"",
  "zh_CN":"",
  "description":"", 
  "enforce":""
}
```



//server_name是服务名称。

config_name是配置选项的名称。

type用来确定当前选项的数值类型，分布尔值bool，数值num，字符串str三种。

depends用来定位需要的前置选项。

required用来定位被需要的选项。

conflict 用来确认冲突选项。

value 存当前配置选项的值。

zh_CN 存这个选项的中文名称。

description 当前选项的描述。 

enforce 标记当前选项是否生效，给js解析的时候设置html控件用。

工作流程 python先读取配置文件，根据当前配置文件把数据填入dict，转成json后传给前端，前端根据传入的json，画出html。修改之后之后，再根据修改的结果把修改后的结果传给后面的python，再根据这个json把配置文件修改好。

