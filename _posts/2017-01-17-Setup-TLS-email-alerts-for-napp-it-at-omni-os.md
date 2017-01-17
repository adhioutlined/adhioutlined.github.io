---
title:  "Setup TLS email alerts for Napp-it at OmniOS"
header:
  image: /assets/images/utnci8mmcr0wic18psyo.jpg
date:  2017-01-17
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

Encrypted TLS email requires that you install the needed OS modules for encryption, The Websocket server requires them also for wss (realtime graphics over https).

I do this things on my OmniOS 151018 and 151020 server. 

## Enable TLS

For the first thing is we have to enable TLS emails on our OmniOS, using the following setup :

```python
# perl -MCPAN -e shell
```
it will take you to the cpan shell

```python
cpan[1]> notest install Net::SSLeay
cpan[1]> notest install IO::Socket::SSL
cpan[1]> notest install Net::SMTP::TLS
cpan[1]> exit
#
```


## Setting email notification

now back to your napp-it on your web browser :

**http://[your-server-ip]:81**

pick menu **About** > **Setting** > **Email Notification** , then fill the form with your mail credentials.

```python
SMTP Mailserver: ex. smtp.gmail.com <~~ fill with your smtp server address

SMTP user: <~~~  fill with your mail username

SMTP password: <~~ your mail password

Mailto (opt commalist): <~~ fill with your destination mail address .

```

then **submit**

## Test TLS mail

**Enable TLS mail**

Go to menu **Jobs** > **TLS Email** > **enable TLS email**

**Test TLS mail**

Go to menu **Jobs** > **TLS Email** > **TLS-test** > **submit** 

to test your mail.

## Relaxed Google security 

Maybe you'll get this error if your using gmail accounts

```python
Auth failed: 534 5.7.14
```

It's because by default now google sets strictly security for every accounts, then you had to relaxed google security setting from this links

[Resolve Auth failed: 534 5.7.14](https://www.google.com/settings/u/1/security/lesssecureapps){: .btn .btn--danger}

Choose **Turn on** for **Access for less secure apps** option.

Then try to test your TLS-email again.


## Referenced

[https://www.napp-it.org/downloads/tls_en.html](https://www.napp-it.org/downloads/tls_en.html)
[http://serverfault.com/questions/635139/how-to-fix-send-mail-authorization-failed-534-5-7-14](http://serverfault.com/questions/635139/how-to-fix-send-mail-authorization-failed-534-5-7-14)

