---
title:  "Add sudoers passwordless centos7"
header:
  image: /assets/images/utnci8mmcr0wic18psyo.jpg
date:  2017-03-01
tags:
  - Linux Basic
description: ''
categories:
  - Basic
---
It's just my note to create passwordless udoers user in centos7

## Add User

```python
# adduser username
```
## Create Password

```python
# passwd username

Set password prompts:
Changing password for user username.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```
## Use `usermod` to add new user to `wheel` group
```python
# usermod -aG wheel username
```
## Create sudoers file for user
```python
# echo 'username  ALL=(ALL)  NOPASSWD: ALL' > /etc/sudoers.d/username
```
