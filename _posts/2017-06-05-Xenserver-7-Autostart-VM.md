---
title:  "Xenserver 7 Autostart VM"
header:
  image: /assets/images/utnci8mmcr0wic18psyo.jpg
date:  2017-06-05
tags:
  - XenServer
description: ''
categories:
  - Virtual
---
This is guide to make Xenserver 7.x automatically start the VM (domU) every reboot. By default this feature wasn't available at XenCenter management software, but we can activate by the `xapi` command line.

First thing we had to make Xenserver pool was activating auto-start feature, we can applied this command even for standalone Xenserver. Get the Xenserver-Pool UUID

```
# xe pool-list
uuid ( RO)                : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
```
Set the Xenserver-Pool parameter to activating auto-start feature
```
# xe pool-param-set uuid=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX other-config:auto_poweron=true
```

Now We also had to activating auto-start feature for the VM we have, gather VM UUID
```
# xe vm-list
uuid ( RO)           : XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
     name-label ( RW): Control domain on host: XenSever_host_name
    power-state ( RO): running


uuid ( RO)           : 11111111-1111-1111-1111-11111111111
     name-label ( RW): VM_host_name_A
    power-state ( RO): running
```

Set the auto power on the VM

```
# xe vm-param-set uuid=11111111-1111-1111-1111-11111111111 other-config:auto_poweron=true
```
reboot the Xenserver (dom0) to test the auto-start VM.
