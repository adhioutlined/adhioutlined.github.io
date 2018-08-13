---
title:  "Install PostgreSQL Nginx phpPGadmin at Centos7"
header:
  image: /assets/images/19503500053_a24778ff1e_z.jpg
date:  2018-08-10
tags:
  - Linux Basic
description: ''
categories:
  - Basic
---
My note about installing PostgreSQL, NGINX, and phpPGadmin at Centos7

## NGINX section

At this phase, we will install the NGINX web server at centos7. To installing NGINX at centos7, first we need to add epel repository.
```
yum install epel-repo
```

After epel repository installed now we can install NGINX
```
yum install nginx
```

Start NGINX
```
systemct  start nginx
```

Allow HTTP and HTTPS port in firewall

```
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

Now check access of webserver

```
http://your_ip_adress
```

## PHP section

We are add PHP function at this step,
```
yum install php php-mysql php-fpm
```

Configure php processor, edit the php-fpm configuration `/etc/php.ini`. Specially for `cgi.fix_pathinfo` we had to commenting out and set the value to `0`.

```
cgi.fix_pathinfo=0
```

open the php-fpm configuration file `/etc/php-fpm.d/www.conf` find the `listen`, `listen.owner`, `listen.group`, `user`, and `group` section, and make it look like this :
```
listen = /var/run/php-fpm/php-fpm.sock

listen.owner = nobody
listen.group = nobody

user = nginx
group = nginx

```

Start php processor, and enabled it:
```
systemctl start php-fpm
systemctl enable php-fpm
```

Next, we had to configure nginx for handling the PHP process. Create the default NGINX server block configuration file `/etc/nginx/conf.d/default.conf` , create configuration looks like :
```
server {
    listen       80;
    server_name  server_domain_name_or_IP;

    # note that these lines are originally from the "location /" block
    root   /usr/share/nginx/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

then restart NGINX, `systemctl restart nginx`

## PostgreSQL section
Now installing database, we are gonna installing PostgreSQL at this step.

```
yum install postgresql-server postgresql-contrib
```
Create a new database cluster
```
postgresql-setup initdb
```
editing `/var/lib/pgsql/data/pg_hba.conf` to make PostgreSQL to support password authentication based, make config like this :
```
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
start and enable PostgreSQL,
```
systemctl start postgresql
systemctl enable postgresql
```
Login as PostgreSQL user, and create PostgreSQL role (user)
```
sudo su postgres
createuser -P --interactive
postgres@your_machine_name:~$ createuser -P --interactive
Enter name of role to add: test
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
postgres@your_machine_name:~$
```
so the user create was `test`

## phpPGadmin section
Relax security, add firewall exception
```
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --zone=public --permanent --add-port=5432/tcp
firewall-cmd --reload
```
also relax selinux
```
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on
```
Setup PostgreSQL listening address, by editing `/var/lib/pgsql/data/postgresql.conf` file
```
listen_addresses = '*'
port = 5432
```

Install phpPGadmin,
```
yum install phpPgAdmin

```
Configure phpPGadmin, by created `/etc/nginx/conf.d/phpPgAdmin.conf ` file ,
```
# Server block for phppgadmin service
server{
        server_name     YOUR_SERVER;
        root            /usr/share/phpPgAdmin;

        access_log      /var/log/phppgadmin/access.log;
        error_log       /var/log/phppgadmin/error.log;

        location ~ \.php$ {
                try_files $uri =404;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                fastcgi_param SCRIPT_FILENAME /path/to/your/www/postgres$fastcgi_script_name;
                include /etc/nginx/fastcgi_params;
        }
}
```
Edit phpPGadmin config file `/etc/phpPgAdmin/config.inc.php`
```
$conf['servers'][0]['host'] = 'localhost';

$conf['servers'][0]['port'] = 5432;

$conf['owned_only'] = true;
```
Restart all service
```
systemctl restart postgresql.service nginx.service php-fpm.service
```
