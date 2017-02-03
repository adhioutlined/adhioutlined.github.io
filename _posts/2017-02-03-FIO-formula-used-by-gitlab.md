---
title:  "FIO formula used by gitlab"
header:
  image: /assets/images/utnci8mmcr0wic18psyo.jpg
date:  2017-02-03
tags:
  - solaris
  - storage
  - san
description: ''
categories:
  - Storage
---
It's just my note to save this fio formula used by gitlab to benchmark storage performance of their VM.

```python
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=10G --readwrite=randread
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=10G --readwrite=randwrite
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=2048k --iodepth=64 --size=10G --readwrite=read
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=2048k --iodepth=64 --size=10G --readwrite=write
```

This formula I get from gitlab live notes during their [database recovery progress](https://docs.google.com/document/d/1GCK53YDcBWQveod9kfzW-VCxIABGiryG7_z_6jHdVik/pub) on this Febuary.

