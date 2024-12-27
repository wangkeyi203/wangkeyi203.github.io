---
title: audispd-plugins
date: 2023-01-03 18:21:04
tags:
---

audispd-plugins功能调研

audisp-remote.conf 

remote_server =    //插件将向其发送日志信息的远程服务器主机名或者地址。

port = 60       //远程服务器接受信息端口。

##local_port =       //本地计算机发送信息时的端口，如果不指定，则使用任何可用的端口。

transport = tcp    //此参数告诉远程日志记录应用程序如何将事件发送到远程系统。现在唯一有效的值是*tcp*。如果设置为*tcp*，远程日志记录应用程序将只与远程系统建立正常的明文连接。

queue_file = /var/spool/audit/remote.log    //队列文件

mode = immediate             //此参数告诉远程日志记录应用程序使用什么策略将记录获取到远程系统。有效值为*immediate*和 *forward*。如果设置为*immediate*，远程日志记录应用程序将尝试在获取事件后立即发送它们。*forward*意味着它会将事件存储到磁盘，然后尝试发送记录。如果无法建立连接，它将对记录进行排队，直到它可以连接到远程系统。

queue_depth = 10240             //此选项是一个无符号整数，用于确定在将其视为发送失败之前可以将多少条记录缓冲到磁盘或内存中。此参数影响*模式选项的**转发*模式和临时网络中断的内部排队。

format = managed           //此参数告诉远程日志记录应用程序将使用哪种数据格式用于通过网络发送的消息。默认时managed会增加开销，如果改成ascii会减少开销

network_retry_time = 1        //检测到网络错误时重试之间的时间（以秒为单位）。

max_tries_per_record = 3        //尝试传递每条消息的最大次数。最小值为 1，因为即使是完全成功的交付也需要至少尝试一次。

max_time_per_record = 5         //尝试传递每条消息所花费的最长时间（以秒为单位）。

heartbeat_timeout = 0               //此参数确定客户端应以秒为单位向远程服务器发送心跳事件的频率。这用于让客户端和服务器都知道每一端都处于活动状态并且没有以没有不干净地关闭连接的方式终止。该值必须与服务器的*tcp_client_max_idle*设置相协调。默认值为 0，即禁用发送心跳。

network_failure_action = stop       //此参数告诉系统在将审计事件发送到远程系统时检测到错误时要采取的操作。有效值为 *ignore*、*syslog*、*exec*、*suspend*、*single*、*halt*和*stop*。

disk_low_action = ignore        //同样，如果远程端发出磁盘低错误信号，此参数会告诉系统采取什么操作。默认是忽略它。

disk_full_action = warn_once    //此参数告诉系统如果远程端发出磁盘已满错误信号要采取的操作。

disk_error_action = warn_once         //这个参数告诉系统如果远程端发出磁盘错误信号要采取什么行动。

remote_ending_action = reconnect        //这个参数告诉系统如果远程端发出磁盘错误信号要采取什么行动。此操作有一个附加选项， *重新连接*，它告诉远程插件在收到下一条审计记录后尝试重新连接到服务器。如果不成功，审计记录可能会丢失。

generic_error_action = syslog      //如果远程端发出我们无法识别的错误信号，此参数会告诉系统采取什么操作。

generic_warning_action = syslog       //这个参数告诉系统如果远程端发出我们不认识的警告信号要采取什么行动。

queue_error_action = stop       //此参数告诉系统在使用本地记录队列时出现问题时要采取的操作。

overflow_action = syslog        //这个参数告诉系统如果内部事件队列溢出要采取什么行动。有效值为*ignore*、*syslog*、*suspend*、 *single*和*halt*。

startup_failure_action = warn_once_continue      //此参数告诉系统在启动期间连接到远程系统时出现错误时要采取的操作。有效值为 ignore、syslog、exec、warn_once 和 warn_once_continue。

##krb5_principal =
##krb5_client_name = auditd
##krb5_key_file = /etc/audisp/audisp-remote.key
