---
title: "Avoid Openstack Neutron using Ryu"
header:
  teaser: /assets/images/8698869017_339ed43c69_o.jpg
date:  2016-12-14 00:43:31
categories:
  - Virtual
tags:
  - openstack
  - neutron
  - cloud computing
---

![alt text](/assets/images/8698869017_339ed43c69_o.jpg)

I'm so excited by the latest release of OpenStack, the Newton release. Recently I workon Liberty release connected to XenServer as a hypervisor, and I skipped tried Mitaka release.

No Mitaka no problem, because we are in Newton now. Since the Kilo release many Neutron features require special tweaks to work on XenServer. As far as I know Security Groups default features still can't running well.

I hope in the Newton release the Neutron features problems that work with XenServer can be resolved. Today a problem arises when I'm setup ```neutron_openvswitch_agent``` in my Compute Node, ```neutron_openvswitch_agent``` can't running normally and there is an error as follows:

{% highlight python %}
2016-12-14 17:09:03.040 5375 ERROR ryu.lib.hub [-] hub: uncaught exception: Traceback (most recent call last):
  File "/usr/lib/python2.7/site-packages/ryu/lib/hub.py", line 54, in _launch
    return func(*args, **kwargs)
  File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/openflow/native/ovs_ryuapp.py", line 37, in agent_main_wrapper
    ovs_agent.main(bridge_classes)
  File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/ovs_neutron_agent.py", line 2170, in main
    agent = OVSNeutronAgent(bridge_classes, cfg.CONF)
  File "/usr/lib/python2.7/site-packages/neutron/plugins/ml2/drivers/openvswitch/agent/ovs_neutron_agent.py", line 139, in __init__
    self.ovs = ovs_lib.BaseOVS()
  File "/usr/lib/python2.7/site-packages/neutron/agent/common/ovs_lib.py", line 107, in __init__
    self.ovsdb = ovsdb.API.get(self)
  File "/usr/lib/python2.7/site-packages/neutron/agent/ovsdb/api.py", line 89, in get
    return iface(context)
  File "/usr/lib/python2.7/site-packages/neutron/agent/ovsdb/impl_idl.py", line 196, in __init__
    OvsdbIdl.ovsdb_connection.start()
  File "/usr/lib/python2.7/site-packages/neutron/agent/ovsdb/native/connection.py", line 89, in start
    helper = do_get_schema_helper()
  File "/usr/lib/python2.7/site-packages/retrying.py", line 68, in wrapped_f
    return Retrying(*dargs, **dkw).call(f, *args, **kw)
  File "/usr/lib/python2.7/site-packages/retrying.py", line 229, in call
    raise attempt.get()
  File "/usr/lib/python2.7/site-packages/retrying.py", line 261, in get
    six.reraise(self.value[0], self.value[1], self.value[2])
  File "/usr/lib/python2.7/site-packages/retrying.py", line 217, in call
    attempt = Attempt(fn(*args, **kwargs), attempt_number, False)
  File "/usr/lib/python2.7/site-packages/neutron/agent/ovsdb/native/connection.py", line 88, in do_get_schema_helper
    self.schema_name)
  File "/usr/lib/python2.7/site-packages/neutron/agent/ovsdb/native/idlutils.py", line 109, in get_schema_helper
    'err': os.strerror(err)})
Exception: Could not retrieve schema from tcp:127.0.0.1:6640: Connection refused
{% endhighlight %}

I use ```neutron_openvswitch_agent``` configuration exactly as it can work well in the release liberty.

After a closer look at the error log there, I realized that by default ```neutron_openvswitch_agent``` in newton call ```ryu.base.app_manager``` release. There seems to be a major change in the neutron, and the possibility of this changes since the release Mitaka.

By default neutron now use the native interface of OVSDB and OpenFlow. By using of native interfaces of OVSDB and OpenFlow allows the neutron call ```ryu.base.app_manager``` during operation. My guess openvswitch in XenServer does not use this Native interface. So I tried to force ```neutron_openvswitch_agent``` not use the native interface by adding the following settings in the file ```/etc/neutron/plugins/ml2/openvswitch_agent.ini```

{% highlight python %}
[ovs]
....
of_interface = ovs-ofctl
ovsdb_interface = vsctl
{% endhighlight %}