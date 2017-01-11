---
title: "Openvswitch Cheat Sheet"
date: 2016-12-14
tags:
  - openvswitch
  - Cheat Sheet
categories:
  - Virtual
---

## Look up the table
```python
ovs-vsctl list bridge ovs-br
```

## About Bridge and Port

OpenVswitch | About Bridge and Port
----------- | ---------------------
Add Bridge | ```ovs-vsctl add-br ovs-br```
Corresponds to the interface on ovs-br | ```ovs-vsctl add-port ovs-br eth0```
(1) + (2) can be written | ```ovs−vsctl add−br ovs-br -- add−port ovs-br eth0```
Remove Bridge | ```ovs-vsctl del-br ovs-br``` # If it does not exist, there will be error log <br /> ```ovs-vsctl --if-exists del-br ovs-br```
Change the ofport (openflow port number) to 100 | ```ovs-vsctl add-port ovs-br eth0 -- set Interface eth0 ofport_request=100```
Set the port to internal | ```ovs-vsctl set Interface eth0 type=internal```

## About the Controller

OpenVswitch | About the Controller
----------- | --------------------
Set the Controller | ```ovs-vsctl set-controller ovs-br tcp:1.2.3.4:6633```
Set the multi controller | ```ovs-vsctl set-controller ovs-br tcp:1.2.3.4:6633 tcp:5.6.7.8:6633```
Query the Controller settings | ```ovs-vsctl show```
If you have successfully connected to the controller appears is_connected:true , otherwise not connected | ```ovs-vsctl get-controller ovs-br```
Remove the Controller | ```ovs-vsctl del-controller ovs-br```


## About STP (Spanning Tree Protocol)

OpenVswitch | About STP 
----------- | ---------
Enable STP | ``` ovs-vsctl set bridge ovs-br stp_enable=true```
Turn off STP | ``` ovs-vsctl set bridge ovs-br stp_enable=false```
Query STP settings | ``` ovs-vsctl get bridge ovs-br stp_enable```
Set Priority | ``` ovs−vsctl set bridge br0 other_config:stp-priority=0x7800```
Set Cost | ``` ovs−vsctl set port eth0 other_config:stp-path-cost=10```
Remove the STP settings | ``` ovs−vsctl clear bridge ovs-br other_config```

## About Openflow Version

OpenVswitch | About Openflow Version
----------- | ----------------------
OpenFlow Version 1.3 is supported | ```ovs-vsctl set bridge ovs-br protocols=OpenFlow13```
Support OpenFlow Version 1.3 1.2 | ```ovs-vsctl set bridge ovs-br protocols=OpenFlow12,OpenFlow13```
Remove the OpenFlow support settings | ```ovs-vsctl clear bridge ovs-br protocols```

## VLAN

OpenVswitch | About VLAN
----------- | ----------
Set the VLAN tag | ``` ovs-vsctl add-port ovs-br vlan3 tag=3 -- set interface vlan3 type=internal```
Remove the VLAN | ``` ovs-vsctl del-port ovs-br vlan3```
Query the VLAN | ```ovs-vsctl show``` <br /> ```ifconfig vlan3```
Set the Vlan trunk | ```ovs-vsctl add-port ovs-br eth0 trunk=3,4,5,6```
Set the add port to access port, vlan id 9 | ``` ovs-vsctl set port eth0 tag=9```
Ovs-ofctl add-flow Set vlan 100 | ``` ovs-ofctl add-flow ovs-br in_port=1,dl_vlan=0xffff,actions=mod_vlan_vid:100,output:3``` <br /> ```ovs-ofctl add-flow ovs-br in_port=1,dl_vlan=0xffff,actions=push_vlan:0x8100,set_field:100-\>vlan_vid,output:3```

Ovs-ofctl add-flow Remove the vlan tag | ``` ovs-ofctl add-flow ovs1 in_port=3,dl_vlan=100,actions=strip_vlan,output:1```
Two_vlan example | ``` ovs-ofctl add-flow pop-vlan``` <br /> ``` ovs-ofctl add-flow ovs-br in_port=3,dl_vlan=0xffff,actions=pop_vlan,output:1```

## About GRE tunnels

OpenVswitch | About GRE
----------- | ---------
Set the GRE tunnel | ```ovs−vsctl add−port ovs-br ovs-gre -- set interface ovs-gre type=gre options:remote_ip=1.2.3.4```
Check the GRE tunnel | ``` ovs-vsctl show```

## About Dump flows

OpenVswitch | About Dump flows
----------- | ----------------
Dumps OpenFlow flows do not contain hidden flows (common) | ``` ovs-ofctl dump-flows ovs-br```
Dumps OpenFlow flows contain hidden flows | ``` ovs-appctl bridge/dump-flows ovs-br```
Dump specific bridge of the datapath flows regardless of any type | ``` ovs-appctl dpif/dump-flows ovs-br```
Dump in the Linux kernel in the datapath flow table (commonly used) | ```ovs-dpctl dump-flows [dp]```
Top like behavior for ovs-dpctl dump-flows | ```ovs-dpctl-top```

## XenServer starts OpenvSwitch mode

OpenVswitch | XenServer
----------- | ---------
Check whether it is on or not | ``` service openvswitch status```
Openv | ``` xe-switch-network-backend openvswitch```
shut down | ``` xe-switch-network-backend bridge```


## About Log

OpenVswitch | About Log
----------- | ---------
Query log level list | ``` ovs-appctl vlog/list```
Set the log level (to stp set dbg level file as an example) | ``` ovs-appctl vlog/set stp:file:dbg``` <br /> ``` ovs-appctl vlog/set {module name}:{console, syslog, file}:{off, emer, err, warn, info, dbg}```

## About Fallback

OpenVswitch | About Fallback
----------- | --------------
Controller connection: false, will be automatically transferred into the legacy switch mode | ``` ovs-vsctl set-fail-mode ovs-br standalone```
Regardless of the Controller connection status why, must be carried out through OpenFlow network behavior (default) | ``` ovs-vsctl set-fail-mode ovs-br secure```
Remove | ``` ovs-vsctl del-fail-mode ovs-br```
Inquire | ``` ovs-vsctl get-fail-mode ovs-br```

## About sFlow

OpenVswitch | About sFlow
----------- | -----------
Inquire | ``` ovs-vsctl list sflow```
New | ``` set sFlow```
delete | ``` ovs-vsctl -- clear Bridge ovs-br sflow```

## About NetFlow

OpenVswitch | About NetFlow
----------- | -------------
Inquire | ``` ovs-vsctl list netflow```
New | ``` Set NetFlow```
Delete | ``` ovs-vsctl -- clear Bridge ovs-br netflow```

## Set the Out-of-band and in-band

OpenVswitch | Set the Out-of-band and in-band
----------- | -------------------------------
Inquire | ``` ovs-vsctl get controller ovs-br connection-mode```
Out-of-band | ``` ovs-vsctl set controller ovs-br connection-mode=out-of-band```
In-band (default) | ``` ovs-vsctl set controller ovs-br connection-mode=in-band```
Remove the hidden flow | ``` ovs-vsctl set bridge br0 other-config:disable-in-band=true```

## About ssl

OpenVswitch | About SSL
----------- | ---------
Inquire | ``` ovs-vsctl get-ssl```
set up | ``` ovs-vsctl set-ssl sc-privkey.pem sc-cert.pem cacert.pem```
delete | ``` ovs-vsctl del-ssl```

## About SPAN

OpenVswitch | About SPAN
----------- | ----------
Detailed settings | ```ovs-vsctl add-br ovs-br``` <br /> ```ovs-vsctl add-port ovs-br eth0``` <br /> ```ovs-vsctl add-port ovs-br eth1``` <br /> ```ovs-vsctl add-port ovs-br tap0 \``` <br /> ```- --id = @ p ​​get port tap0 \``` <br /> ```- - id = @m create mirror name = m0 select-all = true output-port = @ p ​​\``` <br /> ```- set bridge ovs-br mirrors = @ m```
Add | ``` ovs-br on add-port {eth0, eth1} mirror to tap0```
delete | ``` ovs-vsctl clear bridge ovs-br mirrors # About Table```
Check the Table | ``` ovs-ofctl dump-tables ovs-br```

## About VXLAN

[Reference rascov - Bridge Remote Mininets using VXLAN](http://blog.mcchan.io/bridge-remote-networks-using-vxlan)

OpenVswitch | About VxLAN
----------- | -----------
Establish the VXLAN Network ID (VNI) and the specified OpenFlow port number, eg: VNI ​​= 5566, OF_PORT = 9 | ```ovs-vsctl set interface vxlan type=vxlan option:remote_ip=xxxx option:key=5566 ofport_request=9```
VNI flow by flow | ``` ovs-vsctl set interface vxlan type=vxlan option:remote_ip=140.113.215.200 option:key=flow ofport_request=9```
Set the VXLAN tunnel id | ``` ovs-ofctl add-flow ovs-br in_port=1,actions=set_field:5566->tun_id,output:2``` <br /> ``` ovs-ofctl add-flow s1 in_port=2,tun_id=5566,actions=output:1```

## About OVSDB Manager

[Reference OVSDB Integration: Mininet OVSDB Tutorial](https://wiki.opendaylight.org/view/OVSDB_Integration:Mininet_OVSDB_Tutorial)

OpenVswitch | About OVSDB
----------- | -----------
Active Listener settings | ``` ovs-vsctl set-manager tcp:1.2.3.4:6640```
Passive Listener settings | ``` ovs-vsctl set-manager ptcp:6640```

## OpenFlow Trace

OpenVswitch | About OpenFlow Trace
----------- | --------------------
Generate pakcet trace | ``` ovs-appctl ofproto/trace ovs-br in_port=1,dl_src=00:00:00:00:00:01,dl_dst=00:00:00:00:00:02 -generate```

## Other

OpenVswitch | Others
----------- | ------
Query the OpenvSwitch version | ``` ovs-ofctl -V```
Query the history of the next instruction | ``` ovsdb-tool show-log [-mmm]```

## Reference

* [Ovs-vsctl](http://openvswitch.org/cgi-bin/ovsman.cgi?page=utilities%2Fovs-vsctl.8)
* [OpenvSwitch FAQ](http://git.openvswitch.org/cgi-bin/gitweb.cgi?p=openvswitch;a=blob_plain;f=FAQ;hb=HEAD)
* [OpenvSwitch Debugging](http://openvswitch.org/slides/OVS-Debugging-110414.pdf)
* [Network flow monitoring with Open vSwitch](http://www.areteix.net/blog/2013/08/network-flow-monitoring-with-open-vswitch/)
* [Pica8 OpenvSwitch configuration](http://www.pica8.com/document/pica8-OVS-MPLS-configuration-guide.pdf)
* [Hwchiu - Multipath routing with Group table at mininet](http://hwchiu.logdown.com/posts/207387-multipath-routing-with-group-table-at-mininet)
* [Rascov - Bridge Remote Mininets using VXLAN](http://blog.mcchan.io/bridge-remote-networks-using-vxlan)
* [OpenFlow Practice Based on Open vSwitch (Chen Shaq)](http://www.sdnap.com/sdn-technology/5114.html)
* [OpenVswitch Advanced Tutioral](http://vlabs.cfapps.io/openvswitch/openvswitch_tutorial.html)