---
layout: post
title:  "Openstack Nova Migrate Error SSH Key"
image: '/assets/img/blog-img/y4azyntou5ybeulcquqx.jpg'
date:  2017-01-03 00:43:31
tags:
- openstack
- nova
description: ''
categories:
- Virtual
serie: openstack
---

![alt text](/assets/img/blog-img/y4azyntou5ybeulcquqx.jpg)

```python
   ERROR oslo_messaging.rpc.server ResizeError: Resize error: not able to execute ssh command: Unexpected error while running command.
   ERROR oslo_messaging.rpc.server Command: ssh -o BatchMode=yes <compute_host_address> mkdir -p /var/lib/nova/instances/<instance-id>
```

It's possible for you to get those errors while you migrate your instance from one host (hypervisor/compute) to another host (hypervisor/compute). Those errors appears because by default nova service create user nova on system for Daemons only with nologin.

{% highlight python %}
nova:x:162:162:OpenStack Nova Daemons:/var/lib/nova:/sbin/nologin
{% endhighlight %}

Then, while ```nova migrate``` command working, source host (hypervisor/compute) will do an ssh onto destination host for some of nova works. To solve this problem, we had to copy ssh-credential across compute host.

1. Run __setenforce 0__ to put SELinux into permissive mode. 
2. Enable login abilities for the nova user:
   ```python
   # usermod -s /bin/bash nova
   ```
3. as root, give nova user password, for __ssh-copy-id__ works later
   ```python
   # passwd nova
   ```
   then switch to nova account:
   ```python
   # su nova
   ```
4. Generate ssh-key for nova user
   ```python
   $ ssh-keygen -t rsa
   ```
5. Repeat step 1-4 for all nova users in each compute nodes
6. As nova user Copy SSH-Key to every compute nodes
   ```python
   $ ssh-copy-id <compute-nodes-destinations>
   ```
7. Try to ssh, it must be automatically login without password
   ```python
   $ ssh <compute-nodes-destinations>
   ```

Try for ```nova migrate``` again, and see for any further errors.