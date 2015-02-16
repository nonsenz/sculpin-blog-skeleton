---
title: install owncloud in freenas jail
category: administration
tags: 
    - freenas
    - freebsd
    - jails
    - owncloud
    - nginx
    - mariadb

main_img:
    path: freenas_owncloud.jpg
    copyright:
        name:  Glen Van Etten
        url:  https://www.flickr.com/photos/tejedoro_de_luz/
---

we will set up owncloud with mariadb and nginx in a freebsd 9 jail base jail setup described [here](/blog/2014/07/17/freenas-jail-bootstrap)

## mariadb

install mariadb via portmaster

     > portmaster databases/mariadb55-server
     
edit the rc.conf to enable autostart mariadb while booting

     > vi /etc/rc.conf

add `mysql_enable="YES"` and saveexit. to start it manually type

    > service mysql-server start
  
initializing the db data dirs with

    > cd /usr/local/   # mysql_install_db assumes its running from here
    > mysql_install_db --user=mysql

setup root user

    > mysqladmin -u root password
    
create the owncloud database and user for later usage:

    > create database owncloud;
    > create user owncloud identified by 'secret';
    > GRANT ALL PRIVILEGES ON owncloud.* TO 'owncloud'@'localhost' IDENTIFIED BY 'secret';

## nginx

install via portmaster

    > portmaster www/nginx
    
in the config dialog check webdav support (and maybe more?).
now manually start it via 

    > service nginx start

now you should be able to see the default nginx page when surfing the systems ip. next thing to do is move the original config and create a new one:

    > cd /usr/local/etc/nginx
    > mv nginx.conf nginx.conf.orig
    
insert the following code to the new config:

    user www www;
    worker_processes 1;
    worker_priority 15;

    pid /var/run/nginx.pid;
    error_log /var/log/nginx-error.log info;

    events {
        worker_connections 512;
        accept_mutex on;
        use kqueue;
    }

    http {
        include         conf.d/options;
        include         mime.types;
        default_type    application/octet-stream;
        access_log      /var/log/nginx-access.log main buffer=32k;

        include sites/*.site;
    }

next we have to create the options file `/usr/local/etc/nginx/conf.d/options` and paste:

    client_body_timeout 5s;
    client_header_timeout 5s;
    keepalive_timeout 75s;
    send_timeout 15s;
    charset utf-8;
    gzip off;
    gzip_static on;
    gzip_proxied any;
    ignore_invalid_headers on;
    keepalive_requests 50;
    keepalive_disable none;
    max_ranges 1;
    msie_padding off;
    open_file_cache max=1000 inactive=2h;
    open_file_cache_errors on;
    open_file_cache_min_uses 1;
    open_file_cache_valid 1h;
    output_buffers 1 512;
    postpone_output 1440;
    read_ahead 512K;
    recursive_error_pages on;
    reset_timedout_connection on;
    sendfile on;
    server_tokens off;
    server_name_in_redirect off;
    source_charset utf-8;
    tcp_nodelay on;
    tcp_nopush off;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
    limit_req_zone $binary_remote_addr zone=gulag:1m rate=60r/m;
    log_format main '$remote_addr $host $remote_user [$time_local] "$request" $status $body_bytes_sent "$http_referer" "$http_user_agent" $ssl_cipher $request_time';

finally create the sites dir 

    > mkdir /usr/local/etc/nginx/sites
    
## php

install it as always with portmaster

    > portmaster lang/php55

after that lets check the config. but first copy the template:

    > cp /usr/local/etc/php.ini-production /usr/local/etc/php.ini

to be save uncomment the session.save_path in the config.
now install some extensions with:

    > portmaster lang/php55-extensions databases/php55-pdo_mysql 
    
you can find ownclouds requirements and recommendations [here](https://doc.owncloud.org/server/6.0/admin_manual/installation/installation_source.html#prerequisites).
now edit the php-fpm conf at /usr/local/etc/php-fpm.conf and be set the listen params as follows :

    listen =/var/run/php-fpm.sock
    listen.owner = www
    listen.group = www
    listen.mode = 0660

savexit and make php-fpm autostart by adding php_fpm_enable="YES" to the /etc/rc.conf and start it manually with

    > service php-fpm start
    
## owncloud

create the nginx config for owncloud. check [this](https://doc.owncloud.org/server/6.0/admin_manual/installation/installation_source.html#nginx-configuration) for more info. note that this installation will not cover ssl as i only use this in my local network.

     upstream php-handler {
             server unix:/var/run/php-fpm.sock;
     }
     
     server {
             listen 80;
             server_name localhost;
     
             # Path to the root of your installation
             root /var/www/owncloud;
     
             client_max_body_size 10G; # set max upload size
             fastcgi_buffers 64 4K;
     
             rewrite ^/caldav(.*)$ /remote.php/caldav$1 redirect;
             rewrite ^/carddav(.*)$ /remote.php/carddav$1 redirect;
             rewrite ^/webdav(.*)$ /remote.php/webdav$1 redirect;
     
             index index.php;
             error_page 403 /core/templates/403.php;
             error_page 404 /core/templates/404.php;
     
             location = /robots.txt {
                 allow all;
                 log_not_found off;
                 access_log off;
             }
     
             location ~ ^/(data|config|\.ht|db_structure\.xml|README) {
                     deny all;
             }
     
             location / {
                     # The following 2 rules are only needed with webfinger
                     rewrite ^/.well-known/host-meta /public.php?service=host-meta last;
                     rewrite ^/.well-known/host-meta.json /public.php?service=host-meta-json last;
     
                     rewrite ^/.well-known/carddav /remote.php/carddav/ redirect;
                     rewrite ^/.well-known/caldav /remote.php/caldav/ redirect;
     
                     rewrite ^(/core/doc/[^\/]+/)$ $1/index.html;
     
                     try_files $uri $uri/ index.php;
             }
     
             location ~ ^(.+?\.php)(/.*)?$ {
                     try_files $1 =404;
     
                     include fastcgi_params;
                     fastcgi_param SCRIPT_FILENAME $document_root$1;
                     fastcgi_param PATH_INFO $2;
                     fastcgi_pass php-handler;
             }
             # Optional: set long EXPIRES header on static assets
             location ~* ^.+\.(jpg|jpeg|gif|bmp|ico|png|css|js|swf)$ {
                     expires 30d;
                     # Optional: Don't log access to assets
                     access_log off;
             }
     
     }

download owncloud source with wget. right now i want latest rc from 7:

    > wget http://download.owncloud.org/community/testing/owncloud-7.0.0RC2.tar.bz2

you can find the available files at <https://owncloud.org/install/>.
after that unpack the file like this an rename the folder to represent the version.

    > tar xvjf owncloud-7.0.0RC2.tar.bz2


check your browser again. there should be the owncloud install page. while setting it up open the advanced storage config and enter the owncloud user data configured earlier.

