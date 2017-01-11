---
title:  "libvirtError: unsupported configuration: CPU specification not supported by hypervisor"
header:
  image: /assets/images/2624988907_39e6f5e4d4_b.jpg
date:  2016-12-19
tags:
  - kvm
  - qemu
  - openstack
description: ''
categories:
  - Virtual
---
If you're in staging or testing phase of openstack development, it's possible for you using an Virtual Machine as Compute Node / Hypervisor. That's mean you will create and running an VM over an VM.

Typically we only define ```virt_type = qemu``` inside of ```[libvirt]``` section of ```nova.conf``` file. Because we use a VM as a Hypervisor, it is possible to raise an error like this :

```libvirtError: unsupported configuration: CPU specification not supported by hypervisor```

By default nova will define libvirt cpu-mode using "host-model". So if you are running your compute node/hypervisor using an Virtual Machine, you should specify libvirt cpu-mode using an ***"none"*** option. Add this line inside of ```[libvirt]``` section of ```nova.conf``` file.

So your ```[libvirt]``` section of ```nova.conf``` will be look like this :

{% highlight python %}
[libvirt]
virt_type = qemu
cpu_mode = "none"
{% endhighlight %}