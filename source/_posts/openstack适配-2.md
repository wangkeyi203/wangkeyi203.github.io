---
title: openstack适配_2
date: 2020-09-28 16:59:00
tags:
- openstack适配
---

控制节点安装配置Networking service
mysql -uroot -p123456 -e "CREATE DATABASE neutron"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '123456'"
mysql -uroot -p123456 -e "GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '123456'"
openstack user create --domain default --password 123456 neutron
openstack role add --project service --user neutron admin
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
yum install openstack-neutron openstack-neutron-ml2 -y

cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
egrep -v "^$|^#" /etc/neutron/neutron.conf.bak >/etc/neutron/neutron.conf

修改/etc/neutron/neutron.conf
[DEFAULT]
bind_host = 192.168.50.88
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
auth_strategy = keystone
transport_url = rabbit://openstack:123456@controller:5672
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[database]
connection = mysql+pymysql://neutron:123456@controller/neutron

[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = neutron
password = 123456

[nova]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 123456

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp



修改/etc/nova/nova.conf
[neutron]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123456
service_metadata_proxy = true
metadata_proxy_shared_secret = devops

cp /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugins/ml2/ml2_conf.ini.bak
egrep -v "^$|^#" /etc/neutron/plugins/ml2/ml2_conf.ini.bak >/etc/neutron/plugins/ml2/ml2_conf.ini
然后修改该文件
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true

执行 su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
systemctl restart openstack-nova-api.service
systemctl status openstack-nova-api.service
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
systemctl enable neutron-server.service
systemctl restart neutron-server.service
systemctl status neutron-server.service

计算节点安装Networking service
yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables -y
cp /etc/neutron/neutron.conf /etc/neutron/neutron.conf.bak
egrep -v "^$|^#" /etc/neutron/neutron.conf.bak >/etc/neutron/neutron.conf

修改/etc/neutron/neutron.conf
[DEFAULT]
auth_strategy = keystone
transport_url = rabbit://rabbitmq:123456@controller:5672
[database]
connection = mysql+pymysql://neutron:123456@controller/neutron
[keystone_authtoken]
www_authenticate_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = neutron
password = 123456

[oslo_concurrency]
lock_path = /var/lib/neutron/tmp

修改/etc/nova/nova.conf

[neutron]
auth_url = http://controller:5000
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123456

systemctl restart openstack-nova-compute.service
修改/etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = true

修改/etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:enp0s8

[vxlan]
enable_vxlan = true
local_ip = 192.168.50.148
l2_population = true

[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

修改/etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = linuxbridge

修改/etc/neutron/dhcp_agent.ini

[DEFAULT]
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true

修改/etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = devops

然后执行
systemctl enable neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
systemctl restart neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent
systemctl status neutron-linuxbridge-agent neutron-dhcp-agent neutron-metadata-agent neutron-l3-agent

控制节点 安装Dashboard
yum install -y openstack-dashboard
sed -i '118c OPENSTACK_HOST = "controller"' /etc/openstack-dashboard/local_settings
sed -i "39c ALLOWED_HOSTS = ['*']" /etc/openstack-dashboard/local_settings
sed -i "104c SESSION_ENGINE = 'django.contrib.sessions.backends.cache'" /etc/openstack-dashboard/local_settings
sed -i "94c CACHES = {" /etc/openstack-dashboard/local_settings
sed -i "95c 'default': {" /etc/openstack-dashboard/local_settings
sed -i "96c 'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache'," /etc/openstack-dashboard/local_settings
sed -i "97c 'LOCATION': 'controller:11211'," /etc/openstack-dashboard/local_settings
sed -i "98c }" /etc/openstack-dashboard/local_settings
sed -i "99c }" /etc/openstack-dashboard/local_settings
sed -i '119c OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST' /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_API_VERSIONS = {' >> /etc/openstack-dashboard/local_settings
echo '    "identity": 3,' >> /etc/openstack-dashboard/local_settings
echo '    "image": 2,' >> /etc/openstack-dashboard/local_settings
echo '    "volume": 3' >> /etc/openstack-dashboard/local_settings
echo '}' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"' >> /etc/openstack-dashboard/local_settings
echo 'OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"' >> /etc/openstack-dashboard/local_settings
sed -i '123c TIME_ZONE = "Asia/Shanghai"' /etc/openstack-dashboard/local_settings
echo "WEBROOT = '/dashboard/'" >> /etc/openstack-dashboard/local_settings
echo 'WSGIApplicationGroup %{GLOBAL}' >> /etc/httpd/conf.d/openstack-dashboard.conf
systemctl restart httpd.service memcached.service