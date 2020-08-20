---
title: vsftpd配置
date: 2020-07-29 12:48:38
tags: 
- vsftpd
---

| 属性                    | 属性值     | 含义                                                         |
| ----------------------- | ---------- | ------------------------------------------------------------ |
| anonymous_enable        | `YES`/`NO` | 是否允许匿名用户（anonymous）登录 `FTP`，如果该设置被注释，则默认允许 |
| local_enable            | `YES`/`NO` | 是否允许本地系统用户登录                                     |
| write_enable            | `YES`/`NO` | 是否开启任何形式的 `FTP` 写入命令，上传文件                  |
| local_umask             | xxx        | 本地用户的 `umask` 设置，如果注释该设置则默认为 `077`，但一般都设置成 `022` |
| anon_upload_enable      | `YES`/`NO` | 是否允许匿名用户上传文件，如果要设置为允许，则需要先开启 `write_enable`，否则无效，此外对应目录还要具有写权限 |
| anon_mkdir_write_enable | `YES`/`NO` | 是否允许匿名用户创建新目录                                   |
| dirmessage_enable       | `YES`/`NO` | 当进入某个目录时，发送信息提示给远程用户：                   |
| xferlog_enable          | `YES`/`NO` | 是否开启 上传/下载 的日志记录                                |
| connect_from_port_20    | `YES`/`NO` | 是否使用 `20` 端口来连接 `FTP`                               |
| chown_uploads           | `YES`/`NO` | 匿名上传的文件是否由某一指定用户 `chown_username` 所有       |
| chown_username          | 有效用户名 | 匿名上传的文件由该设定用户所有                               |
| xferlog_file            | 有效路径   | 设置日志文件的保存位置，默认为 `/var/log/xferlog`            |
| xferlog_std_format      | `YES`/`NO` | 是否使用标准的 `ftpd xferlog`日志格式，该格式日志默认保存在 `/var/log/xferlog` |
| idle_session_timeout    | 数值       | 设置空闲连接的超时时间，单位 秒                              |
| data_connection_timeout | 数值       | 设置等待数据传输的最大时间，单位 秒（`data_connection_timeout` 与 `idle_session_timeout` 在同一时间只有一个有效） |
| nopriv_user             | 有效用户名 | 指定一个非特权用户，用于运行 `vsftpd`                        |
| async_abor_enable       | `YES`/`NO` | 是否支持异步 `ABOR` 请求                                     |
| ascii_upload_enable     | `YES`/`NO` | 是否开启 `ASCII` 模式进行文件上传，一般不开启                |
| ascii_download_enable   | `YES`/`NO` | 是否开启 `ASCII` 模式进行文件下载，一般不开启                |
| ftpd_banner             | …          | 自定义登录标语                                               |
| deny_email_enable       | `YES`/`NO` | 如果匿名登录，则会要求输入 email 地址，如果不希望一些 email 地址具有登录权限，则可以开启此项，并在 `banned_email_file` 指定的文件中写入对应的 email 地址 |
| banned_email_file       | 有效文件   | 当开启 `deny_email_enable` 时，需要通过此项指定一个保存登录无效 email 的文件 |
| chroot_local_user       | `YES`/`NO` | 是否将所有用户限制在主目录，当为 `NO` 时， `FTP` 用户可以切换到其他目录 |
| chroot_list_enable      | `YES`/`NO` | 是否启用限制用户的名单列表                                   |
| chroot_list_file        | 有效文件   | 用户列表，其作用与 `chroot_local_user` 和 `chroot_local_user` 的组合有关，详见下表 |
| allow_writeable_chroot  | `YES`/`NO` | 是否允许用户对 ftp 根目录具有写权限，如果设置成不允许而目录实际上却具备写权限，则会报错 |
| ls_recurse_enable       | `YES`/`NO` | 是否允许 `ls -R` 指令来递归查询，递归查询比较耗资源          |
| listen                  | `YES`/`NO` | 如果为 `YES`，`vsftpd` 将以独立模式运行并监听 `IPv4` 的套接字，处理相关连接请求（该指令不能与 `listen_ipv6` 一起使用） |
| listen_ipv6             | `YES`/`NO` | 是否允许监听 `IPv6` 套接字                                   |
| pam_service_name        | …          | 设置 `PAM` 外挂模块提供的认证服务所使用的配置文件名 ，即 `/etc/pam.d/vsftpd` 文件，此文件中 `file=/etc/vsftpd/ftpusers` 字段，说明了 `PAM` 模块能抵挡的帐号内容来自文件 `/etc/vsftpd/ftpusers` 中 |
| userlist_enable         | `YES`/`NO` | 是否启用 `user_list` 文件来控制用户登录                      |
| userlist_deny           | `YES`/`NO` | 是否拒绝 `user_list` 中的用户登录，此属性设置需在 `userlist_enable = YES` 时才有效 |
| tcp_wrappers            | `YES`/`NO` | 是否使用 `tcp_wrappers` 作为主机访问控制方式                 |
| max_clients             | 数值       | 同一时间允许的最大连接数                                     |
| max_per_ip              | 数值       | 同一个IP客户端连接的最大值                                   |
| local_root              | 有效目录   | 系统用户登录后的根目录                                       |
| anon_root               | 有效目录   | 匿名用户登录后的根目录                                       |
| user_config_dir         | 有效目录   | 用户单独配置文件存放目录，该目录下用户的文件名就是对应用户名 |

`chroot_local_user` 和 `chroot_local_user` 组合功能如下：

|                        | chroot_local_user=YES                                        | chroot_local_user=NO                                         |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| chroot_list_enable=YES | 1.所有用户都被限制在其主目录下 2.使用 `chroot_list_file` 指定的用户列表 `/etc/vsftpd/chroot_list`，这些用户作为“例外”，不受限制 | 1.所有用户都不被限制其主目录下 2.使用 `chroot_list_file` 指定的用户列表 `/etc/vsftpd/chroot_list`，这些用户作为“例外”，受到限制 |
| chroot_list_enable=NO  | 1.所有用户都被限制在其主目录下 2.不使用 `chroot_list_file` 指定的用户列表 `/etc/vsftpd/chroot_list`，没有任何“例外”用户 | 1.所有用户都不被限制其主目录下 2.不使用 `chroot_list_file` 指定的用户列表 `/etc/vsftpd/chroot_list`，没有任何“例外”用户 |