---
title:  "Nova Giveup of VHD Coalesce"
header:
  image: /assets/images/11682625885_4e0bdb28d8_z.jpg
date:  2017-09-14
tags:
  - Openstack
  - nova
  - XenServer
description: ''
categories:
  - Virtual
---
When your openstack running with Xenserver as compute HyperVisor you may find this error to instance who made often migration across the HyperVisor.

If you check the nova-compute log, you will find error like **NovaException: VHD coalesce attempts exceeded.** , this because the xapi plugin is waiting for a coalesce to happen after the snapshot - and timing out.

this is the detail of ERROR the appears at nova-compute log.
```
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [req-89edd082-6755-447a-bce8-93464d812ac1 44d172fd7b0b4cfd836329ac8cf3e7b1 3dabca2caf6f48aca388d21
17573459a - - -] [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688] _migrate_disk_resizing_up failed. Restoring orig vm due_to: VHD coalesce attempts exceeded
 (20), giving up....
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688] Traceback (most recent call last):
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]   File "/usr/lib/python2.7/site-packages/nova/vir
t/xenapi/vmops.py", line 1184, in _migrate_disk_resizing_up
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]     self._session, instance, vm_ref, label) as ro
ot_vdi_uuids:
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]   File "/usr/lib64/python2.7/contextlib.py", line
 17, in __enter__
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]     return self.gen.next()
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]   File "/usr/lib/python2.7/site-packages/nova/vir
t/xenapi/vm_utils.py", line 673, in _snapshot_attached_here_impl
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]     vdi_uuid_chain)
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]   File "/usr/lib/python2.7/site-packages/nova/vir
t/xenapi/vm_utils.py", line 2098, in _wait_for_vhd_coalesce
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]     raise exception.NovaException(msg)
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688] NovaException: VHD coalesce attempts exceeded (20
), giving up...
2017-09-13 12:58:57.807 1648 ERROR nova.virt.xenapi.vmops [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688]
2017-09-13 12:58:57.850 1648 INFO nova.compute.manager [req-89edd082-6755-447a-bce8-93464d812ac1 44d172fd7b0b4cfd836329ac8cf3e7b1 3dabca2caf6f48aca388d21175
73459a - - -] [instance: 8c8a0f6e-7108-4202-bbd6-9e3dffda4688] Setting instance back to ACTIVE after: Instance rollback performed due to: VHD coalesce attem
pts exceeded (20), giving up...
```

Then we have a few work on Dom0 that host the instance. First time we need to check the `/var/log/SMlog` of Dom0. You may find information like :

```
Sep 14 16:36:26 fatxs-1 SMGC: [19805] *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*
Sep 14 16:36:26 fatxs-1 SMGC: [19805]          ***********************
Sep 14 16:36:26 fatxs-1 SMGC: [19805]          *  E X C E P T I O N  *
Sep 14 16:36:26 fatxs-1 SMGC: [19805]          ***********************
Sep 14 16:36:26 fatxs-1 SMGC: [19805] coalesce: EXCEPTION <class 'util.CommandException'>, Invalid argument
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/cleanup.py", line 1542, in coalesce
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     self._coalesce(vdi)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/cleanup.py", line 1732, in _coalesce
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     vdi._doCoalesce()
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/cleanup.py", line 692, in _doCoalesce
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     self.parent._increaseSizeVirt(self.sizeVirt)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/cleanup.py", line 890, in _increaseSizeVirt
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     self._setSizeVirt(size)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/cleanup.py", line 905, in _setSizeVirt
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     vhdutil.setSizeVirt(self.path, size, jFile)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/vhdutil.py", line 228, in setSizeVirt
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     ioretry(cmd)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/vhdutil.py", line 102, in ioretry
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     errlist = [errno.EIO, errno.EAGAIN])
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/util.py", line 292, in ioretry
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     return f()
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/vhdutil.py", line 101, in <lambda>
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     return util.ioretry(lambda: util.pread2(cmd),
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/util.py", line 189, in pread2
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     return pread(cmdlist, quiet = quiet)
Sep 14 16:36:26 fatxs-1 SMGC: [19805]   File "/opt/xensource/sm/util.py", line 182, in pread
Sep 14 16:36:26 fatxs-1 SMGC: [19805]     raise CommandException(rc, str(cmdlist), stderr.strip())
Sep 14 16:36:26 fatxs-1 SMGC: [19805]
Sep 14 16:36:26 fatxs-1 SMGC: [19805] *~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*~*
Sep 14 16:36:26 fatxs-1 SMGC: [19805] Coalesce failed, skipping
Sep 14 16:36:26 fatxs-1 SMGC: [19805] In cleanup
Sep 14 16:36:26 fatxs-1 SMGC: [19805] Starting asynch srUpdate for SR 0995f66a-6188-ad63-4d00-117498b7fd46
```

To fixing corrupted/failed Coalesce of VHD you should repair the VHD using `vhd-util` tools.

Check the VHD condition
```
# vhd-util check -n /var/run/sr-mount/0995f66a-6188-ad63-4d00-117498b7fd46/91499ab6-63e9-47cf-b73a-b7fc2b09dcc7.vhd
primary footer invalid: invalid cookie
/var/run/sr-mount/0995f66a-6188-ad63-4d00-117498b7fd46/91499ab6-63e9-47cf-b73a-b7fc2b09dcc7.vhd appears invalid; dumping metadata
VHD Footer Summary:
-------------------
Cookie              : conectix
Features            : (0x00000002) <RESV>
File format version : Major: 1, Minor: 0
Data offset         : 512
Timestamp           : Thu Sep 14 15:48:19 2017
Creator Application : 'tap'
Creator version     : Major: 1, Minor: 3
Creator OS          : Unknown!
Original disk size  : 3072 MB (3221225472 Bytes)
Current disk size   : 10240 MB (10737418240 Bytes)
Geometry            : Cyl: 20805, Hds: 16, Sctrs: 63
                    : = 10239 MB (10737377280 Bytes)
Disk type           : Differencing hard disk
Checksum            : 0xffffed5b|0xffffed5b (Good!)
UUID                : 55fd3e99-5cf8-4dfd-97f5-96851bf7f51b
Saved state         : No
Hidden              : 0

VHD Header Summary:
-------------------
Cookie              : cxsparse
Data offset (unusd) : 18446744073709
Table offset        : 1536
Header version      : 0x00010000
Max BAT size        : 5120
Block size          : 2097152 (2 MB)
Parent name         : 16f02a7d-4e4b-4298-a44a-9cc155cbd2b1.vhd
Parent UUID         : 2b0565ca-4f72-4471-a0e9-c7f2b7460491
Parent timestamp    : Thu Sep 14 15:48:19 2017
Checksum            : 0xffffd93a|0xffffd93a (Good!)

VHD Parent Locators:
--------------------
locator:            : 0
       code         : PLAT_CODE_MACX
       data_space   : 512
       data_length  : 49
       data_offset  : 23552
       decoded name : ./16f02a7d-4e4b-4298-a44a-9cc155cbd2b1.vhd

locator:            : 1
       code         : PLAT_CODE_W2KU
       data_space   : 512
       data_length  : 84
       data_offset  : 24064
       decoded name : ./16f02a7d-4e4b-4298-a44a-9cc155cbd2b1.vhd

locator:            : 2
       code         : PLAT_CODE_W2RU
       data_space   : 512
       data_length  : 84
       data_offset  : 24576
       decoded name : ./16f02a7d-4e4b-4298-a44a-9cc155cbd2b1.vhd

VHD Batmap Summary:
-------------------
Batmap offset       : 22528
Batmap size (secs)  : 2
Batmap version      : 0x00010002
Checksum            : 0xfffff312|0xfffff312 (Good!)
```

Repair the VHD

```
vhd-util repair –n /var/run/sr-mount/<sr-uuid>/<vdi-uud>.vhd
```

Example :

```
# vmuuid=$(xe vm-list name-label=<INSTANCE NAME> --minimal)
# vbduuid=$(xe vbd-list vm-uuid=$vmuuid --minimal)
# xe vdi-list vbd-uuids=$vbduuid

uuid ( RO)                : 91499ab6-63e9-47cf-b73a-b7fc2b09dcc7
          name-label ( RW): instance-000000c4
    name-description ( RW): root
             sr-uuid ( RO): 0995f66a-6188-ad63-4d00-117498b7fd46
        virtual-size ( RO): 10737418240
            sharable ( RO): false
           read-only ( RO): false

vhd-util repair –n /var/run/sr-mount/0995f66a-6188-ad63-4d00-117498b7fd46/91499ab6-63e9-47cf-b73a-b7fc2b09dcc7.vhd
```

Then Check the VHD condition using .
```
vhd-util check –n /var/run/sr-mount/<sr-uuid>/<vdi-uud>.vhd
```

Example :
```
[root@fatxs-1 images]# vhd-util check -n /var/run/sr-mount/0995f66a-6188-ad63-4d00-117498b7fd46/91499ab6-63e9-47cf-b73a-b7fc2b09dcc7.vhd
/var/run/sr-mount/0995f66a-6188-ad63-4d00-117498b7fd46/91499ab6-63e9-47cf-b73a-b7fc2b09dcc7.vhd is valid
```

If the vhd-util check result OK/Good you can try to migrate instance again.

Reference :

- http://lists.openstack.org/pipermail/openstack/2013-March/041259.html
- https://support.citrix.com/article/CTX201296
