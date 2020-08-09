---
title: cockpit数据格式
date: 2020-08-05 18:11:18
tags:
- cockpit数据格式
---

今天写处理配置文件的逻辑的时，发现之前的有些问题，就给改了下

往前端传的数据，是一个list，里面元素是

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

然后每个服务还需要选项属性文件，用来记录每项配置的属性

  anonymous_enable,bool,null,null,null,null,是否允许匿名登录,111111,null
  local_enable
  write_enable
  local_umask
  anon_upload_enable

分别对应字典里的config_name,type,depends,required,conflict,value,zh_CN,description,enforce 

主要是用来判断选项的type，depends，required，conflict，再就是用来填选项显示的中文名和描述。

脚本先从选项属性文件里读出每条选项的属性，处理成list，在从服务配置文件里处理出来选项的当前状态，然后填到字典里，再转成json传给前端。这样以后增加服务，就给添加一个服务的选项属性文件。





```
{
    "packageName":"软件包名",
    "version":"软件包版本，暂只做页面显示用",
    "name":"软件包可读性较好的名字。可省略，如省略使用软件包名",
    "config":{
        "path":"配置文件的路径",
        "type":"配置文件的类型,xml,ini,property,json....",
        "handler":"处理此配置文件的脚本二binary程序，用于将配置文件转成json格式填充进来，以及将web中用户填写的内容写到配置文件",
        "version":"保留字段，配置文件的版本，暂时不使用。",
        "oldConfigPath":"保留字段，用于保存上一次配置的路径",
        "data":[
            {
                "name":"配置的key",
                "displayName":"用户显示给用户看的名字",
                "description":"描述",
                "type":"配置value的类型，INT，STRING，BOOL，RADIO，MULTISELECT,SELECT...",
                "value":"属性的值",
                "default":"默认值",
                "depends":{
                    "name":"依赖配置名",
                    "value":"依赖属性的值为这些值时才显示"
                },
                "child":"",
                "conflict":"与哪些属性互斥",
                "optional":0
            }
        ]
    }
}
```

