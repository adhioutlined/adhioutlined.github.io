---
title:  "Recovering Openstack Galera Cluster"
header:
  image: /assets/images/4930961643_ba9b08888d_b.jpg.jpg
date:  2017-10-23
tags:
  - Openstack
  - MariaDB
  - Galera
description: ''
categories:
  - Virtual
---
It's just my note, how to recovering the MariaDB galera cluster while all node failed to create new cluster after hardware failure.

The symptoms when you start a new cluster, you will find `InnoDB: corruption in the InnoDB tablespace` notice in `mariadb.log`.

```
InnoDB: Assertion failure in thread 1129654592 in file ibuf0ibuf.c line 4231
InnoDB: Failing assertion: page_get_n_recs(page) > 1
InnoDB: We intentionally generate a memory trap.
InnoDB: Submit a detailed bug report to http://bugs.mysql.com.
InnoDB: If you get repeated assertion failures or crashes, even
InnoDB: immediately after the mysqld startup, there may be
InnoDB: corruption in the InnoDB tablespace. Please refer to
InnoDB: http://dev.mysql.com/doc/refman/5.5/en/forcing-innodb-recovery.html
InnoDB: about forcing recovery.
mysqld got signal 6 ;
This could be because you hit a bug. It is also possible that this binary
or one of the libraries it was linked against is corrupt, improperly built,
or misconfigured. This error can also be caused by malfunctioning hardware.
We will try our best to scrape up some info that will hopefully help diagnose
the problem, but since we have already crashed, something is definitely wrong
and this may fail.
...
some backtrace
...
The manual page at http://dev.mysql.com/doc/mysql/en/crashing.html contains
information that should help you find out what is causing the crash.
mysqld_safe Number of processes running now: 0
mysqld_safe mysqld restarted
```
This happen because some of your InnoDB tables are corrupt.

### Shut Down all DB nodes ###

Shut down all DB nodes, except one node to recovery progress.

### Initiate InnoDB recovery ###

Open `wsrep_cluster_address` setting to open address , so the configuration will be like this
```
# Group communication system handle
#wsrep_cluster_address="gcomm://172.17.9.23,172.17.9.24,172.17.9.22"
wsrep_cluster_address="gcomm://"
```

Turn on `innodb_recovery_force=[1-6]` into mariadb config file `/etc/my.cnf` try set the smalest value first then start the service

start maria DB service
```
root@nodedb3 # systemctl start mariadb
```

make sure MariaDB running in recovery mode.

### Check DB health ###

Check DB health using `mysqlcheck`

```
mysqlcheck -u root -p --all-databases --check
```

### Dumping & restore database ###

If the `mysqlcheck` found DB with tables error, make a DB dumping to make backup

to dump all database
```
$ mysqldump -u root -p --all-databases > alldb_backup.sql
```

to dump specific database
```
$ mysqldump -u [uname] -p[pass] [dbname] > dbname.sql
```

then drop the database

After sucessfully dropped database, restore it from the DB backup file.
```
$ mysql -u [uname] -p[pass] [db_to_restore] < [backupfile.sql]
```

### Start MariaDB normal Mode ###
Comment the `innodb_recovery_force` option in `/etc/my.cnf` and restart the mariaDB service.

### Start MariaDB in other Node ###
After MariaDB recovered from once node, start the other galera Node to recover the cluster. After Galera cluster recovered, put  `wsrep_cluster_address` setting back to origin.
