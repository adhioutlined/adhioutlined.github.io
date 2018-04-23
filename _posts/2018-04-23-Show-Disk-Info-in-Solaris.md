---
title:  "Show Disk Info in Solaris"
header:
  image: /assets/images/12798055643_ed4046568d_k-e1474062189329.jpg
date:  2018-04-23
tags:
  - Solaris
description: ''
categories:
  - Storage
---
It's just my note, about How to Show Disk Info in Solaris. Using the `iostat` command we can gather about disk information.
In this case I need to see the HDD Vendor and Type/Series, so I'll use `-E`.
```
# iostat -E
sd0       Soft Errors: 0 Hard Errors: 0 Transport Errors: 0
Vendor: ATA      Product: ST4000NM0035-1V4 Revision: TN02 Serial No: ZC114E8X
Size: 4000.79GB <4000787030016 bytes>
Media Error: 0 Device Not Ready: 0 No Device: 0 Recoverable: 0
Illegal Request: 64965 Predictive Failure Analysis: 0
sd1       Soft Errors: 0 Hard Errors: 0 Transport Errors: 0
Vendor: ATA      Product: ST4000NM0033-9ZM Revision: SN06 Serial No: S1Z2N7ZF
Size: 4000.79GB <4000787030016 bytes>
Media Error: 0 Device Not Ready: 0 No Device: 0 Recoverable: 0
Illegal Request: 64969 Predictive Failure Analysis: 0
sd2       Soft Errors: 0 Hard Errors: 0 Transport Errors: 0
Vendor: ATA      Product: ST4000NM0035-1V4 Revision: TN02 Serial No: ZC114E2N
Size: 4000.79GB <4000787030016 bytes>
Media Error: 0 Device Not Ready: 0 No Device: 0 Recoverable: 0
Illegal Request: 64965 Predictive Failure Analysis: 0
sd3       Soft Errors: 1 Hard Errors: 0 Transport Errors: 40
Vendor: ATA      Product: ST4000NM0035-1V4 Revision: TN02 Serial No: ZC114JNZ
Size: 4000.79GB <4000787030016 bytes>
Media Error: 0 Device Not Ready: 0 No Device: 0 Recoverable: 0
Illegal Request: 1799 Predictive Failure Analysis: 0
```
