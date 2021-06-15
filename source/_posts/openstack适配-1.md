---
title: openstack适配（一）
date: 2020-09-25 15:20:18
tags:
- nfs
- openstack
---

Nfs911版本适配OpenStack Ussuri。

为了配置方便，先关闭防火墙和selinux。

```
systemctl stop firewalld && systemctl disable firewalld
setenforce 0
```

修改/etc/selinux/config中 

SELINUX=disabled

关闭swap分区

```
swapoff -a
sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

设置内核

```
modprobe bridge
modprobe br_netfilter
cat > /etc/sysconfig/modules/neutron.modules <<EOF
#!/bin/bash
modprobe -- bridge
modprobe -- br_netfilter
EOF
chmod 755 /etc/sysconfig/modules/neutron.modules && bash /etc/sysconfig/modules/neutron.modules
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables=1" >> /etc/sysctl.conf
sysctl -p
```

设置时间同步

```
yum install -y chrony && yum -y autoremove
systemctl start chronyd.service && systemctl enable chronyd.service

```

控制节点设置hostname

```
hostnamectl set-hostname controller
```

计算节点设置hostname

```
hostnamectl set-hostname compute1
```

将机器ip分别添加到/etc/hosts中（ip根据自己机器修改）

```
echo "192.168.50.1 controller" >> /etc/hosts
echo "192.168.50.2 compute1" >> /etc/hosts
```

因为911版本的包缺少openstack的仓库，我们需要在centos8.2上获取这些包。

centos-release-ceph-nautilus-1.2-2.el8.noarch.rpm  
centos-release-openstack-ussuri-1-3.el8.noarch.rpm  
centos-release-storage-common-2-2.el8.noarch.rpm
centos-release-messaging-1-2.el8.noarch.rpm  
centos-release-rabbitmq-38-1-2.el8.noarch.rpm

将这些包安装到系统中，遇到依赖问题，就先安装依赖的包就可以。
然后创建个shell脚本test.sh

#!/bin/bash
dir=/etc/yum.repos.d

for file in $dir/*
do
    sed -i 's/$releasever/8/g' $file
    echo $file
done
执行  sh test.sh 
 yum makecache
yum install -y python3-openstackclient

先配置控制节点
安装配置Mariadb
yum install -y mariadb mariadb-server python2-PyMySQL （python2-PyMySQL要从centos8.2上拿）
tee /etc/my.cnf.d/openstack.cnf <<-'EOF'
[mysqld]
bind-address = 192.168.50.1
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
EOF
systemctl enable mariadb.service && systemctl start mariadb.service
echo -e "\nY\n123456\n123456\nY\nn\nY\nY\n" | mysql_secure_installation

安装配置RabbitMQ
systemctl enable rabbitmq-server.service && systemctl start rabbitmq-server.service
rabbitmqctl add_user openstack 123456
rabbitmqctl set_permissions openstack ".*" ".*" ".*"

安装配置Memcached
yum install -y memcached python3-memcached
sed -i "s/-l 127.0.0.1,::1/-l 127.0.0.1,::1,controller/g" /etc/sysconfig/memcached
systemctl enable memcached.service && systemctl start memcached.service

安装配置Etcd
yum install -y etcd
rm -f /etc/etcd/etcd.conf

tee /etc/etcd/etcd.conf <<-'EOF'
#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="192.168.50.1:2380"
ETCD_LISTEN_CLIENT_URLS="192.168.50.1:2379"
ETCD_NAME="controller"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="192.168.50.1:2380"
ETCD_ADVERTISE_CLIENT_URLS="192.168.50.1:2379"
ETCD_INITIAL_CLUSTER="controller=192.168.50.1:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
systemctl enable etcd && systemctl start etcd

安装配置Identity service
mysql -uroot -p123456 -e "CREATE DATABASE keystone"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY '123456'"

先从centos8 下载python3-dns这个包并安装
yum install -y openstack-keystone httpd python3-mod_wsgi

sed -i "556c connection = mysql+pymysql://keystone:123456@controller/keystone" /etc/keystone/keystone.conf
sed -i "2418c provider = fernet" /etc/keystone/keystone.conf
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
keystone-manage bootstrap --bootstrap-password 123456 --bootstrap-admin-url http://controller:5000/v3/  --bootstrap-internal-url http://controller:5000/v3/  --bootstrap-public-url http://controller:5000/v3/  --bootstrap-region-id RegionOne
echo "ServerName controller" >> /etc/httpd/conf/httpd.conf
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
启动服务
systemctl enable httpd.service && systemctl start httpd.service

echo "export OS_USERNAME=admin" >> /etc/profile
echo "export OS_PASSWORD=123456" >> /etc/profile
echo "export OS_PROJECT_NAME=admin" >> /etc/profile
echo "export OS_USER_DOMAIN_NAME=Default" >> /etc/profile
echo "export OS_PROJECT_DOMAIN_NAME=Default" >> /etc/profile
echo "export OS_AUTH_URL=http://controller:5000/v3" >> /etc/profile
echo "export OS_IDENTITY_API_VERSION=3" >> /etc/profile
source /etc/profile
openstack project create --domain default --description "Service Project" service

安装配置Image service
mysql -uroot -p123456 -e "CREATE DATABASE glance"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 glance
openstack role add --project service --user glance admin
openstack service create --name glance --description "OpenStack Image" image
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292

yum install -y openstack-glance
sed -i "2062c connection = mysql+pymysql://glance:123456@controller/glance" /etc/glance/glance-api.conf
sed -i "5034c www_authenticate_uri  = http://controller:5000" /etc/glance/glance-api.conf
sed -i "5035c auth_url = http://controller:5000" /etc/glance/glance-api.conf
sed -i "5036c memcached_servers = controller:11211" /etc/glance/glance-api.conf
sed -i "5037c auth_type = password" /etc/glance/glance-api.conf
sed -i "5038c project_domain_name = Default" /etc/glance/glance-api.conf
sed -i "5039c user_domain_name = Default" /etc/glance/glance-api.conf
sed -i "5040c project_name = service" /etc/glance/glance-api.conf
sed -i "5041c username = glance" /etc/glance/glance-api.conf
sed -i "5042c password = 123456" /etc/glance/glance-api.conf
sed -i "5678c flavor = keystone" /etc/glance/glance-api.conf
sed -i "3413c stores = file,http" /etc/glance/glance-api.conf
sed -i "3414c default_store = file" /etc/glance/glance-api.conf
sed -i "3415c filesystem_store_datadir = /var/lib/glance/images/" /etc/glance/glance-api.conf

su -s /bin/sh -c "glance-manage db_sync" glance
systemctl enable openstack-glance-api.service && systemctl start openstack-glance-api.service

安装配置Placement service
mysql -uroot -p123456 -e "CREATE DATABASE placement"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 placement
openstack role add --project service --user placement admin
openstack service create --name placement --description "Placement API" placement
openstack endpoint create --region RegionOne placement public http://controller:8778
openstack endpoint create --region RegionOne placement internal http://controller:8778
openstack endpoint create --region RegionOne placement admin http://controller:8778

yum install -y openstack-placement-api

sed -i "507c connection = mysql+pymysql://placement:123456@controller/placement" /etc/placement/placement.conf
sed -i "192c auth_strategy = keystone" /etc/placement/placement.conf
sed -i "241c auth_url = http://controller:5000/v3" /etc/placement/placement.conf
sed -i "242c memcached_servers = controller:11211" /etc/placement/placement.conf
sed -i "243c auth_type = password" /etc/placement/placement.conf
sed -i "244c project_domain_name = Default" /etc/placement/placement.conf
sed -i "245c user_domain_name = Default" /etc/placement/placement.conf
sed -i "246c project_name = service" /etc/placement/placement.conf
sed -i "247c username = placement" /etc/placement/placement.conf
sed -i "248c password = 123456" /etc/placement/placement.conf
su -s /bin/sh -c "placement-manage db sync" placement
systemctl restart httpd

安装配置Compute service
mysql -uroot -p123456 -e "CREATE DATABASE nova_api"
mysql -uroot -p123456 -e "CREATE DATABASE nova"
mysql -uroot -p123456 -e "CREATE DATABASE nova_cell0"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 nova
openstack role add --project service --user nova admin
openstack service create --name nova --description "OpenStack Compute" compute
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
yum install -y openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler

sed -i "2c enabled_apis = osapi_compute,metadata" /etc/nova/nova.conf
sed -i "3c transport_url = rabbit://openstack:123456@controller:5672/" /etc/nova/nova.conf
sed -i "4c my_ip = 10.0.0.11" /etc/nova/nova.conf
sed -i "1079c connection = mysql+pymysql://nova:123456@controller/nova_api" /etc/nova/nova.conf
sed -i "1622c connection = mysql+pymysql://nova:123456@controller/nova" /etc/nova/nova.conf
sed -i "872c auth_strategy = keystone" /etc/nova/nova.conf
sed -i "2561c www_authenticate_uri = http://controller:5000/" /etc/nova/nova.conf
sed -i "2562c auth_url = http://controller:5000/" /etc/nova/nova.conf
sed -i "2563c memcached_servers = controller:11211" /etc/nova/nova.conf
sed -i "2564c auth_type = password" /etc/nova/nova.conf
sed -i "2565c project_domain_name = Default" /etc/nova/nova.conf
sed -i "2566c user_domain_name = Default" /etc/nova/nova.conf
sed -i "2567c project_name = service" /etc/nova/nova.conf
sed -i "2568c username = nova" /etc/nova/nova.conf
sed -i "2569c password = 123456" /etc/nova/nova.conf
sed -i "5171c enabled = true" /etc/nova/nova.conf
sed -i '5172c server_listen = $my_ip' /etc/nova/nova.conf
sed -i '5173c server_proxyclient_address = $my_ip' /etc/nova/nova.conf
sed -i "1937c api_servers = http://controller:9292" /etc/nova/nova.conf
sed -i "3571c lock_path = /var/lib/nova/tmp" /etc/nova/nova.conf
sed -i "4093c region_name = RegionOne" /etc/nova/nova.conf
sed -i "4094c project_domain_name = Default" /etc/nova/nova.conf
sed -i "4095c project_name = service" /etc/nova/nova.conf
sed -i "4096c auth_type = password" /etc/nova/nova.conf
sed -i "4097c user_domain_name = Default" /etc/nova/nova.conf
sed -i "4098c auth_url = http://controller:5000/v3" /etc/nova/nova.conf
sed -i "4099c username = placement" /etc/nova/nova.conf
sed -i "4100c password = 123456" /etc/nova/nova.conf
sed -i "4509c discover_hosts_in_cells_interval = 300" /etc/nova/nova.conf

su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
systemctl enable openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service && systemctl start openstack-nova-api.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service

