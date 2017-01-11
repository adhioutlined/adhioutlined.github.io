---
title:  "Maybe Libvirt have Deadlock while Openstack Nova-Live-Migration"
header:
  image: /assets/images/4831557124_e0e6b30b82_b.jpg
date:  2017-01-10
tags:
  - openstack
  - nova
description: ''
categories:
  - Virtual
---

Just in case you got a Libvirt Deadlock while doing instance Live-migration on Openstack with libvirt. Maybe you'll find some libvirt authentication error at nova-compute log.

### Firewall Area
Make sure TCP ports of libvirt allowed by firewall, if no rules defined for libvirt in iptables then add this rule in ```/etc/sysconfig/iptables```

```python
-A INPUT -p tcp -m multiport --ports 16509 -m comment --comment "libvirt" -j ACCEPT
-A INPUT -p tcp -m multiport --ports 49152:49216 -m comment --comment "migration" -j ACCEPT
-A INPUT -p tcp -m multiport --ports 5901:5910 -m comment --comment "vnc-migration" -j ACCEPT
```

First rule allow libvirt to listen on TCP port **16509**, 

Second rule was for allowing KVM communicating on TCP port within range from **49152** to **49261**, 

Last rule for allowing VNC console communicating on TCP port within range from **5901** to **5910**.

### Libvirt Configuration

Enable libvirt listen flag at ```/etc/sysconfig/libvirtd``` file.

```python
LIBVIRTD_ARGS=”–listen”
```

Configure ``` /etc/libvirt/libvirtd.conf``` to make hypervisor listen TCP communication with none authentication.

```python
listen_tls = 0
listen_tcp = 1
auth_tcp = “none”
```

Because we use NONE authentication , it's strongly recomended to use SSH keys for authentication.

### NOVA configuration

Add this option in nova configuration file ```/etc/nova/nova.conf```

```python
[DEFAULT]
.....

live_migration_flag=VIR_MIGRATE_UNDEFINE_SOURCE,VIR_MIGRATE_PEER2PEER,VIR_MIGRATE_LIVE
live_migration_uri=qemu+tcp://%s/system
```


