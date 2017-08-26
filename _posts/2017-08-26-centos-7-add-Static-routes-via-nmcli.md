---
title:  "Centos 7 Add Static Routes via nmcli"
header:
  image: /assets/images/6605404301_347f303a88_b.jpg
date:  2017-08-26
tags:
  - Linux Basic
description: ''
categories:
  - Basic
---
For me configuring Network in Centos 7 are more tricky when NetworkManager is active. Often configuring hard trough the configuration file inside `/etc/sysconfig/network-scripts/` are failed, because different format used between `NetworkManager.service` & `network.service`.

For example if we need to add static route for some device, lets say `eth1` device. When `network.service` is used, we can add this line to `/etc/sysconfig/network-scripts/route-eth1` file.

```
192.168.0.0/16 via 192.168.5.254 dev eth1
```
The scripts above will failed when `NetworkManager.service` is active. So the format file `/etc/sysconfig/network-scripts/route-eth1` for `NetworkManager.service` are :
```
ADDRESS0=192.168.0.0
NETMASK0=255.255.0.0
GATEWAY0=192.168.6.254
```

We also can generate those file using `nmcli` command, first list all active connection
```
# nmcli connection show
NAME  UUID                                  TYPE            DEVICE
eth0  48069ad9-696c-44ba-b420-69b0d397a3bc  802-3-ethernet  eth0
eth1  7b99eeac-c9be-4d69-b6b8-6def1f3399be  802-3-ethernet  eth1
```
enter edit mode for eth1 device
```
# nmcli connection edit eth1

===| nmcli interactive connection editor |===

Editing existing '802-3-ethernet' connection: 'eth1'

Type 'help' or '?' for available commands.
Type 'describe [<setting>.<prop>]' for detailed property description.

You may edit the following settings: connection, 802-3-ethernet (ethernet), 802-1x, dcb, ipv4, ipv6
nmcli>
```
enter to the IPv4 configuration
```
nmcli> goto ipv4
You may edit the following properties: method, dns, dns-search, dns-options, dns-priority, addresses, gateway, routes, route-metric, ignore-auto-routes, ignore-auto-dns, dhcp-hostname, dhcp-send-hostname, never-default, may-fail, dad-timeout, dhcp-timeout, dhcp-client-id, dhcp-fqdn
nmcli ipv4>
```
Check the current IPv4 configuration for eth1 device
```
nmcli ipv4> print
['ipv4' setting values]
ipv4.method:                            auto
ipv4.dns:
ipv4.dns-search:
ipv4.dns-options:                       (default)
ipv4.dns-priority:                      0
ipv4.addresses:
ipv4.gateway:                           --
ipv4.routes:
ipv4.route-metric:                      500
ipv4.ignore-auto-routes:                no
ipv4.ignore-auto-dns:                   no
ipv4.dhcp-client-id:                    --
ipv4.dhcp-timeout:                      0
ipv4.dhcp-send-hostname:                yes
ipv4.dhcp-hostname:                     --
ipv4.dhcp-fqdn:                         --
ipv4.never-default:                     no
ipv4.may-fail:                          yes
ipv4.dad-timeout:                       -1 (default)
nmcli ipv4>
```
Add static routes
```
set routes 192.168.0.0/16 192.168.6.254
```

Check configuration again to make sure the change of static routes
```
nmcli ipv4> print
['ipv4' setting values]
ipv4.method:                            auto
ipv4.dns:
ipv4.dns-search:
ipv4.dns-options:                       (default)
ipv4.dns-priority:                      0
ipv4.addresses:
ipv4.gateway:                           --
ipv4.routes:                            { ip = 192.168.0.0/16, nh = 192.168.6.254 }
ipv4.route-metric:                      500
ipv4.ignore-auto-routes:                no
ipv4.ignore-auto-dns:                   no
ipv4.dhcp-client-id:                    --
ipv4.dhcp-timeout:                      0
ipv4.dhcp-send-hostname:                yes
ipv4.dhcp-hostname:                     --
ipv4.dhcp-fqdn:                         --
ipv4.never-default:                     no
ipv4.may-fail:                          yes
ipv4.dad-timeout:                       -1 (default)
nmcli ipv4>
```
Save and Quit from `nmcli` mode
```
nmcli ipv4> save
Connection 'eth1' (7b99eeac-c9be-4d69-b6b8-6def1f3399be) successfully updated.
nmcli ipv4> quit
#
```
Reload configuration for `eth1`  to activate the new static routes.
```
# nmcli connection down eth1;nmcli connection up eth1
```
