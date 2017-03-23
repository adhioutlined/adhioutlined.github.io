---
title:  "When Importing Solaris SMF Fail"
header:
  image: /assets/images/utnci8mmcr0wic18psyo.jpg
date:  2017-03-23
tags:
  - OmniOS
  - Napp-it
  - zfs
  - solaris
  - storage
  - san
description: ''
categories:
  - Storage
---
It's May be happen while you importing custom SMF into your solaris, in my case I use space instead of tab while inserting value on `/etc/services` file.

This error appear when I re-import SMF manifest using `inetconv` command.

```
# inetconv -f
check_mk -> /lib/svc/manifest/network/check_mk-tcp.xml
Importing check_mk-tcp.xml ...svccfg: Temporary service "TEMP/network/check_mk/tcp" must be deleted before this manifest can be imported.
svccfg: Import of /lib/svc/manifest/network/check_mk-tcp.xml failed.  Progress:
svccfg:   Service "network/check_mk/tcp": not reached.
svccfg:     Instance "default": not reached.
svccfg: Import of /lib/svc/manifest/network/check_mk-tcp.xml failed.

inetconv: import failure (1) for /lib/svc/manifest/network/check_mk-tcp.xml
```

The first `inetconv` command runnning was look ok, no error output, but actually the service was not add into system. No appear when I check the service at `svcs -a`. This why I decide to re-import SMF manifest using `inetconv -f` command.

After fixing mistype in `/etc/services` file, this error still appear.

```
svccfg: Temporary service "TEMP/network/check_mk/tcp" must be deleted before this manifest can be imported.
```
So , to clear this error, we should remove failed temporary service using `svccfg delete -f <service-path>` command, in my case I have to remove failed temporary check_mk service

```
svccfg delete -f TEMP/network/check_mk/tcp
```

then we can re-import our custom SMF again.
