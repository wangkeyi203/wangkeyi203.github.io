---
title: openstack模块介绍
date: 2020-09-28 15:18:18
tags:
- openstack
- 各个模块介绍
- ussuri
---

数据库
大多数OpenStack服务都使用SQL数据库来存储信息。数据库通常在控制器节点上运行。通常使用MariaDB或MySQL。

消息队列
OpenStack使用消息队列来协调服务之间的操作和状态信息。消息队列服务通常在控制器节点上运行。OpenStack支持多种消息队列服务，包括RabbitMQ，Qpid和ZeroMQ。centos8的ussuri包里和官方文档推荐使用的是RabbitMQ。因为RabbitMQ支持的发行版比较全面，我也是使用RabbitMQ来配置的。

Memcached
openstack的服务器身份认证（ Identity service）是使用 Memcached来缓存身份令牌。Memcached是部署在控制节点上，官方文档建议在生产环境，使用防火墙放行，身份验证和加密三合一来使用，我这边在适配的时候为了方便验证，是将防火墙关闭，单独使用Memcached。

Etcd
OpenStack服务可以使用Etcd（分布式可靠键值存储仓库）进行分布式键锁定，存储配置，跟踪服务活动性和其他情况。

Keystone身份验证服务
OpenStack身份服务为管理身份验证，授权和服务目录提供了单点集成。
身份服务通常是用户与之交互的第一项服务。身份验证后，最终用户可以使用其身份访问其他OpenStack服务。同样，其他OpenStack服务也利用身份服务来确保用户的身份正确，并发现其他服务在部署中的位置。身份服务还可以与某些外部用户管理系统（例如LDAP）集成。

Image service 
Image service（主要是glance-api服务）使用户能够发现，注册和检索虚拟机映像。它提供了 REST API，使用户可以查询虚拟机映像元数据并检索实际映像。用户可以通过Image service将虚拟机映像存储在各种位置。
Image service是openstack的重要组件，它接受来自磁盘或服务器映像的API请求，以及来自最终用户或OpenStack Compute组件的元数据定义。

Placement service
Placement服务（openstack-placement-api）作用是用来跟踪资源库存和使用情况。

Compute service
Compute service（openstack-nova-api openstack-nova-conductor  openstack-nova-novncproxy openstack-nova-scheduler）是openstack的重要组成部分，使用python开发。OpenStack Compute与OpenStack Identity交互以进行身份验证，与OpenStack Placement进行资源清单跟踪和选择，为磁盘和服务器映像提供OpenStack Image服务，并为用户和管理界面提供OpenStack Dashboard。
Compute service由以下几个模块组成：
nova-api service
nova-api-metadata service
nova-compute service （这个主要是在计算节点创建虚拟机用的，其他的服务大部分布置在控制节点）
nova-scheduler service
nova-conductor module
nova-novncproxy daemon
nova-spicehtml5proxy daemon

Networking service
Networking service（neutron）是openstack用来管理虚拟网络和物理网络的服务。

Dashboard
Dashboard需要Memcached服务的配合，用来管理openstack各种资源，跟其他服务的dashboard是相同的功能。