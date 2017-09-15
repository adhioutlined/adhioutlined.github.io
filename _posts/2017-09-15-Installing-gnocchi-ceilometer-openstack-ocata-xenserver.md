---
title:  "Installing Gnocchi and Ceilometer for Openstack Ocata XenServer"
header:
  image: /assets/images/123847612834761928346124192837461283746.jpg
date:  2017-09-15
tags:
  - Openstack
  - XenServer
description: ''
categories:
  - Virtual
---
# Why Gnochi ?
Since the Newton Release, openstack change the Ceilometer backend database to Gnocchi instead of MongoDB. Ceilometer’s native storage has extremely large resource requirements. For more efficient time-series storage, a time-series database such as Gnocchi is recommended.

# What I Have ?
An running openstack Ocata environment deploy on top Centos with XenServer as Hypervisor.

# What I do ?
I deploy the Telemetry Data Collection service with Gnocchi and Ceilometer using separate node from controller. Let's say my **"Metric Node"**, my Metric Node have IP address `192.168.26.10`. Because of `gnocchi-api` service running on my Metric Node instead of Controller Node, then we had to set metric endpoints to Metric Node IP address later.

## Create OpenStack initial setup
Let's do some preparation for this works, first we create the Database for Gnocchi and Ceilometer. My database using MariaDb

```
# mysql -u root -p

MariaDB [(none)]> CREATE DATABASE gnocchidb default character set utf8;
MariaDB [(none)]> GRANT ALL ON gnocchidb.* TO 'gnocchidbuser'@'%' IDENTIFIED BY 'P@ssw0rd';
MariaDB [(none)]> GRANT ALL ON gnocchidb.* TO 'gnocchidbuser'@'localhost' IDENTIFIED BY 'P@ssw0rd';
MariaDB [(none)]> FLUSH PRIVILEGES;
MariaDB [(none)]> exit;
```

Next Create the Service and endpoints
```
# source /root/keystonerc_fulladmin

# openstack user create --domain default --password-prompt ceilometer

# openstack role add --project service --user ceilometer admin

# openstack service create --name ceilometer --description "Telemetry" metering

# openstack user create --domain default --password-prompt gnocchi

# openstack role add --project service --user gnocchi admin

# openstack service create --name gnocchi --description "Metric Service" metric

# openstack endpoint create --region RegionOne metric public http://<MY_METRIC_NODE_IP>:8041

# openstack endpoint create --region RegionOne metric internal http://<MY_METRIC_NODE_IP>:8041

# openstack endpoint create --region RegionOne metric admin http://<MY_METRIC_NODE_IP>:8041
```

## METRIC NODE works

### Install Openstack Repository
```
# yum install -y centos-release-openstack-ocata && yum makecache fast
```

### Install Gnocchi Services

I'm using gnocchi with RPM via YUM. If you want, you can installed gnocchi via `pip`, [follow the instruction](http://gnocchi.xyz/install.html) from Gnocchi official site for manual installation.

```
yum install openstack-gnocchi-api openstack-gnocchi-common \
openstack-gnocchi-indexer-sqlalchemy openstack-gnocchi-metricd openstack-gnocchi-statsd python-memcached crudini python2-gnocchiclient
```

### Configure Gnocchi
Let's use `crudini` to config gnocchi.

```
# crudini --set /etc/gnocchi/gnocchi.conf DEFAULT verbose false
# crudini --set /etc/gnocchi/gnocchi.conf DEFAULT log_dir /var/log/gnocchi
# crudini --set /etc/gnocchi/gnocchi.conf DEFAULT transport_url "rabbit://<RABBIT_USER>:<RABBIT_PASS>@<RABBIT_HOSTNAME>"

# crudini --set /etc/gnocchi/gnocchi.conf api auth_mode keystone

# crudini --set /etc/gnocchi/gnocchi.conf archive_policy default_aggregation_methods "mean,min,max,sum,std,median,count,last,95pct"

# crudini --set /etc/gnocchi/gnocchi.conf database backend sqlalchemy
# crudini --set /etc/gnocchi/gnocchi.conf database connection "mysql+pymysql://gnocchidbuser:P@ssw0rd@DB_SERVER/gnocchidb"

# crudini --set /etc/gnocchi/gnocchi.conf indexer url "mysql+pymysql://gnocchidbuser:P@ssw0rd@DB_SERVER/gnocchidb"
# crudini --set /etc/gnocchi/gnocchi.conf indexer driver sqlalchemy

# statsuuid=$(uuidgen)
# crudini --set /etc/gnocchi/gnocchi.conf statsd resource_id $statsuuid
# crudini --set /etc/gnocchi/gnocchi.conf statsd archive_policy_name low

# crudini --set /etc/gnocchi/gnocchi.conf storage driver file
# crudini --set /etc/gnocchi/gnocchi.conf storage file_basepath /var/lib/gnocchi
# crudini --set /etc/gnocchi/gnocchi.conf storage coordination_url "file:///var/lib/gnocchi/locks"

# crudini --set /etc/gnocchi/gnocchi.conf service_credentials auth_type password
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials auth_url "http://controller:5000/v3"
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials project_domain_name default
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials user_domain_name default
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials project_name service
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials username ceilometer
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials password ceilometerpass
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials interface internalURL
# crudini --set /etc/gnocchi/gnocchi.conf service_credentials region_name RegionOne

# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken auth_uri http://controller:5000
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken auth_url http://controller:35357/v3
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken auth_type password
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken memcached_servers "controller:11211"
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken project_domain_name default
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken user_domain_name default
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken project_name service
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken username gnocchi
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken password gnocchi_password
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken interface internalURL
# crudini --set /etc/gnocchi/gnocchi.conf keystone_authtoken region_name RegionOne

# crudini --set /etc/gnocchi/api-paste.ini "pipeline:main" pipeline "gnocchi+auth"

# mkdir /var/cache/gnocchi  && chown gnocchi:gnocchi -R  /var/cache/gnocchi
# mkdir /var/lib/gnocchi  && chown gnocchi:gnocchi -R  /var/lib/gnocchi
```
Upgrade populate/provision the gnocchi index database
```
# gnocchi-upgrade
```
By default `gnocchi-api` will run at port 8000, to matching with openstack metric endpoints, we need to change `gnocchi-api` port to 8041 by edit the `/usr/bin/gnocchi-api` file. Continued with allowing port 8041 from firewall
```
# sed -i 's/8000/8041/g' /usr/bin/gnocchi-api
# firewall-cmd --zone=public --add-port=8041/tcp --permanent
# firewall-cmd --reload
```
start all gnocchi service
```
systemctl enable openstack-gnocchi-api openstack-gnocchi-metricd openstack-gnocchi-statsd
systemctl start openstack-gnocchi-api openstack-gnocchi-metricd openstack-gnocchi-statsd
```
Check gnocchi service status, and error log. Use your keystone admin credential file, don't forget to add `export OS_AUTH_TYPE = password` in your  keystone admin credential file.

### Test the gnocchi.
Use the gnocci client to test gnocchi.
```
# gnocchi status
+-----------------------------------------------------+-------+
| Field                                               | Value |
+-----------------------------------------------------+-------+
| storage/number of metric having measures to process | 0     |
| storage/total number of measures to process         | 0     |
+-----------------------------------------------------+-------+
```

Edit the granularity
```
gnocchi archive-policy create -d granularity:5m,points:12 -d granularity:1h,points:24 -d granularity:1d,points:30 low
gnocchi archive-policy create -d granularity:60s,points:60 -d granularity:1h,points:168 -d granularity:1d,points:365 medium
gnocchi archive-policy create -d granularity:1s,points:86400 -d granularity:1m,points:43200 -d granularity:1h,points:8760 high
gnocchi archive-policy-rule create -a low -m "*" default
```

### Install Ceilometer Service
Install Ceilometer service using RPM via YUM
```
yum install -y openstack-ceilometer-collector openstack-ceilometer-notification \
  openstack-ceilometer-central python-ceilometerclient python-wsme
```

### Configure Ceilometer

```
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT verbose true
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT meter_dispatchers gnocchi
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT event_dispatchers gnocchi
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT transport_url "rabbit://<RABBIT_USER>:<RABBIT_PASS>@<RABBIT_HOSTNAME/IP>"

# crudini --set /etc/ceilometer/ceilometer.conf dispatcher_gnocchi filter_service_activity False
# crudini --set /etc/ceilometer/ceilometer.conf dispatcher_gnocchi archive_policy low

# crudini --set /etc/ceilometer/ceilometer.conf service_credentials auth_type password
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials auth_url "http://controller:5000/v3"
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials project_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials user_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials project_name service
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials username ceilometer
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials password ceilometer_pass
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials interface internalURL
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials region_name RegionOne
```
Create Ceilometer resources in Gnocchi
```
# ceilometer-upgrade --skip-metering-database
```
Enable and start the Telemetry service
```
# systemctl enable openstack-ceilometer-notification.service \
  openstack-ceilometer-central.service \
  openstack-ceilometer-collector.service
# systemctl start openstack-ceilometer-notification.service \
  openstack-ceilometer-central.service \
  openstack-ceilometer-collector.service
```
### Install Ceilometer Compute Agent in Compute Node
Install Ceilometer compute agent using RPM via YUM

```
# yum install -y openstack-ceilometer-compute crudini
```
### Configure Ceilometer Compute Agent
```
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT hypervisor_inspector xenapi
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT verbose true
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT transport_url "rabbit://<RABBIT_USER>:<RABBIT_PASS>@<RABBIT_HOSTNAME/IP>"
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT auth_strategy keystone
# crudini --set /etc/ceilometer/ceilometer.conf DEFAULT log_dir /var/log/ceilometer

# crudini --set /etc/ceilometer/ceilometer.conf compute instance_discovery_method naive

# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken auth_uri "http://controller:5000"
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken auth_url "http://controller:35357"
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken memcached_servers "controller:11211"
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken auth_type password
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken project_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken user_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken project_name service
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken username ceilometer
# crudini --set /etc/ceilometer/ceilometer.conf keystone_authtoken password ceilometer_pass

# crudini --set /etc/ceilometer/ceilometer.conf service_credentials auth_type password
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials auth_url "http://controller:5000/v3"
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials project_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials user_domain_name default
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials project_name service
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials username ceilometer
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials password ceilometer_pass
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials interface internalURL
# crudini --set /etc/ceilometer/ceilometer.conf service_credentials region_name RegionOne

# crudini --set /etc/ceilometer/ceilometer.conf xenapi connection_url "http://<XENSERVER_IP_ADDRESS"
# crudini --set /etc/ceilometer/ceilometer.conf xenapi connection_username root
# crudini --set /etc/ceilometer/ceilometer.conf xenapi connection_password <XENSERVER_ROOT_PASSWORD>
```

### Configure Compute to use Telemetry
Add this option to `/etc/nova/nova.conf` file in your compute node.

```
# crudini --set /etc/nova/nova.conf DEFAULT instance_usage_audit True
# crudini --set /etc/nova/nova.conf DEFAULT instance_usage_audit_period hour
# crudini --set /etc/nova/nova.conf DEFAULT notify_on_state_change vm_and_task_state

# crudini --set /etc/nova/nova.conf oslo_messaging_notifications driver messagingv2
```

Enable and restart service
```
# systemctl enable openstack-ceilometer-compute.service
# systemctl start openstack-ceilometer-compute.service

# systemctl restart openstack-nova-compute.service
```
### Check the Metric
After about 15 minutes, check the Gnocchi status, and resource lists

```
# gnocchi resource list --type instance -c id -c display_name
+--------------------------------------+--------------+
| id                                   | display_name |
+--------------------------------------+--------------+
| d203e7f8-186e-44d7-9f47-74c696124a4e | prom-cl1     |
| 7893356c-283c-4b2e-a890-b3ffccd35949 | mon-srg      |
| fac48538-0f58-4de1-85c6-fdadfe8d565c | tehanget     |
+--------------------------------------+--------------+

# gnocchi resource show d203e7f8-186e-44d7-9f47-74c696124a4e
+-----------------------+---------------------------------------------------------------------+
| Field                 | Value                                                               |
+-----------------------+---------------------------------------------------------------------+
| created_by_project_id | 24bf793d5e5244f8ab497cad9816b369                                    |
| created_by_user_id    | 3fcd0d94cd1345ffb2e937f2615cad3a                                    |
| creator               | 3fcd0d94cd1345ffb2e937f2615cad3a:24bf793d5e5244f8ab497cad9816b369   |
| ended_at              | None                                                                |
| id                    | d203e7f8-186e-44d7-9f47-74c696124a4e                                |
| metrics               | compute.instance.booting.time: e567caee-d9c1-4c07-901f-8c119fdbaec1 |
|                       | cpu.delta: 13f9d6af-0441-482a-8e6d-625974e81ddd                     |
|                       | cpu: 49978e5d-4775-4d70-bc4f-41e374f15a9d                           |
|                       | cpu_l3_cache: 47c663c9-78bb-4ebf-8083-9fd8c4ac7bc5                  |
|                       | cpu_util: 7eb17084-1553-4137-acb1-83f51abc348c                      |
|                       | disk.allocation: 2ee9ca0a-83ee-47d3-831c-34c620fa4a20               |
|                       | disk.capacity: a022cc29-0312-424d-8598-713ff966b15d                 |
|                       | disk.ephemeral.size: 70b91e59-389f-47f3-9f2e-a22720598903           |
|                       | disk.iops: 963775ee-76fa-42f3-b2c8-2aa664c8e9f3                     |
|                       | disk.latency: 1520eac0-826e-4109-bae4-156d58429299                  |
|                       | disk.read.bytes.rate: cab39fcf-a553-468b-8bc5-872a3b1df139          |
|                       | disk.read.bytes: 2f236639-4ca4-46f0-8a0a-a370c22ca69b               |
|                       | disk.read.requests.rate: 72dc103f-430f-44c7-9c0a-5fb5de6fead2       |
|                       | disk.read.requests: ef5c3b8f-d565-4b4f-bf56-e3c2dd7992a4            |
|                       | disk.root.size: d51a72db-9467-4714-8f13-38a27441dd5f                |
|                       | disk.usage: 29ae8307-a963-4bda-93fa-1386bdfed3bd                    |
|                       | disk.write.bytes.rate: 250b61ac-f4d6-4c47-a538-3ad857646b1b         |
|                       | disk.write.bytes: a0f1a3fe-231d-4d6f-985e-af819d9e3f75              |
|                       | disk.write.requests.rate: 0f1377d7-3127-4984-8f0c-04aa400f4b9d      |
|                       | disk.write.requests: 054462f9-c33c-4bf6-823e-ea32ebd89c72           |
|                       | memory.bandwidth.local: ca831817-3f87-49c0-887f-d16730d385b8        |
|                       | memory.bandwidth.total: 78a959e7-f736-48b2-92be-5c22037ef24e        |
|                       | memory.resident: 87d30032-6e04-4d1b-81f4-3c70accc69dc               |
|                       | memory.usage: 617d02a6-d246-4ccd-964c-7a623cd5fc56                  |
|                       | memory: 8bb03507-433a-4ba9-9d71-e918aef19128                        |
|                       | perf.cache.misses: d9a9742c-a2be-4144-baa8-3bf988c53c22             |
|                       | perf.cache.references: 76da5362-1192-4d6a-955f-2859fffc0a3d         |
|                       | perf.cpu.cycles: fb05ed27-bfb7-40c0-92dc-0bb605dbbb96               |
|                       | perf.instructions: 6c57a4b4-fec7-4fa8-a55b-8635fa3324e9             |
|                       | vcpus: c6d21dd8-3706-4682-b0aa-e8d8877a1576                         |
| original_resource_id  | d203e7f8-186e-44d7-9f47-74c696124a4e                                |
| project_id            | 44a24d52922e4cba9cadc8569728e104                                    |
| revision_end          | None                                                                |
| revision_start        | 2017-09-12T06:42:42.134352+00:00                                    |
| started_at            | 2017-09-12T06:42:42.134336+00:00                                    |
| type                  | instance                                                            |
| user_id               | 2e1f670e576545289be19fc134d0864a                                    |
+-----------------------+---------------------------------------------------------------------+

# gnocchi measures show 7eb17084-1553-4137-acb1-83f51abc348c
+---------------------------+-------------+-----------------+
| timestamp                 | granularity |           value |
+---------------------------+-------------+-----------------+
| 2017-09-12T00:00:00+00:00 |     86400.0 |  0.128578109739 |
| 2017-09-13T00:00:00+00:00 |     86400.0 |  0.233725771396 |
| 2017-09-14T00:00:00+00:00 |     86400.0 |  0.144799765935 |
| 2017-09-15T00:00:00+00:00 |     86400.0 |  0.158299882857 |
| 2017-09-14T05:00:00+00:00 |      3600.0 | 0.0475997506631 |
| 2017-09-14T06:00:00+00:00 |      3600.0 |  0.146442977348 |
| 2017-09-14T07:00:00+00:00 |      3600.0 |  0.177794484747 |
| 2017-09-14T08:00:00+00:00 |      3600.0 |  0.138790727457 |
| 2017-09-14T09:00:00+00:00 |      3600.0 | 0.0883682995967 |
| 2017-09-14T10:00:00+00:00 |      3600.0 |  0.240357051249 |
| 2017-09-14T11:00:00+00:00 |      3600.0 |   0.11409045801 |
| 2017-09-14T12:00:00+00:00 |      3600.0 |  0.140119885934 |
| 2017-09-14T13:00:00+00:00 |      3600.0 | 0.0868569108813 |
| 2017-09-14T14:00:00+00:00 |      3600.0 |  0.278745149747 |
| 2017-09-14T15:00:00+00:00 |      3600.0 | 0.0511041535598 |
| 2017-09-14T16:00:00+00:00 |      3600.0 |  0.161444203817 |
| 2017-09-14T17:00:00+00:00 |      3600.0 |  0.203060101315 |
| 2017-09-14T18:00:00+00:00 |      3600.0 |  0.138982608526 |
| 2017-09-14T19:00:00+00:00 |      3600.0 |  0.168237239268 |
| 2017-09-14T20:00:00+00:00 |      3600.0 |  0.103726969367 |
| 2017-09-14T21:00:00+00:00 |      3600.0 | 0.0965127501937 |
| 2017-09-14T22:00:00+00:00 |      3600.0 |  0.226772870519 |
| 2017-09-14T23:00:00+00:00 |      3600.0 |  0.117725197439 |
| 2017-09-15T00:00:00+00:00 |      3600.0 |  0.101538751915 |
| 2017-09-15T01:00:00+00:00 |      3600.0 | 0.0795756161097 |
| 2017-09-15T02:00:00+00:00 |      3600.0 |  0.291508250666 |
| 2017-09-15T03:00:00+00:00 |      3600.0 | 0.0540401818095 |
| 2017-09-15T04:00:00+00:00 |      3600.0 |  0.477910075642 |
| 2017-09-15T03:20:00+00:00 |       300.0 | 0.0218942048377 |
| 2017-09-15T03:25:00+00:00 |       300.0 | 0.0273383891908 |
| 2017-09-15T03:30:00+00:00 |       300.0 | 0.0199829868507 |
| 2017-09-15T03:35:00+00:00 |       300.0 |  0.401234114543 |
| 2017-09-15T03:40:00+00:00 |       300.0 | 0.0241629430093 |
| 2017-09-15T03:45:00+00:00 |       300.0 | 0.0182056217454 |
| 2017-09-15T03:50:00+00:00 |       300.0 | 0.0291028991342 |
| 2017-09-15T03:55:00+00:00 |       300.0 | 0.0197634843062 |
| 2017-09-15T04:00:00+00:00 |       300.0 | 0.0275397702353 |
| 2017-09-15T04:05:00+00:00 |       300.0 | 0.0343208026607 |
| 2017-09-15T04:10:00+00:00 |       300.0 | 0.0224961535423 |
| 2017-09-15T04:15:00+00:00 |       300.0 |   1.82728357613 |
+---------------------------+-------------+-----------------+

```


Reference :

- https://docs.openstack.org/project-install-guide/telemetry/ocata/get_started.html
- http://www.cnblogs.com/multi-task/p/5553830.html
- https://github.com/tigerlinux/tigerlinux-extra-recipes/blob/master/recipes/openstack/ceilometer-with-gnocchi-backend/RECIPE-ceilometer-with-gnocchi-backend.md
