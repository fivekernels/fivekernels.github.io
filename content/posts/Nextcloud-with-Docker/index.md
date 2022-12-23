---
title: Nextcloud With Docker
date: 2022-12-20T11:05:07+08:00
lastmod: 2022-12-20T11:05:07+08:00
tags: 
- nextcloud
- docker
categories: 
- nextcloud
draft: true
---

&emsp;&emsp;本文介绍了在树莓派(aarch64)上使用docker的方式配置nextcloud:fpm、mariadb、nginx、redis以及onlyoffice。

&emsp;&emsp;Nextcloud官方手册中有数据库等各种镜像版本推荐<https://docs.nextcloud.com/server/latest/admin_manual/installation/system_requirements.html>，最好不要使用过于新的版本以免出现未被解决的兼容性问题。

**Nextcloud版本介绍：**

- nextcloud:latest：默认最新版本，新手友好。使用debian容器搭建，自带apache web服务，特点就是简单方便，一键部署就可以使用，缺点就是包略大。
- nextcloud:fpm：在debian容器的基础上不带apache web服务，需要配套web程序才能使用，如NGINX。
- nextcloud:\<version\>-alpine：使用alpine作为容器，空间占据少，适合小主机使用。

## docker-compose

### docker-compose.yml

```yml {hl_lines=[35, 36]}
version: '3'

services:
    nextcloud:
        hostname: nextcloud
        image: nextcloud:fpm
        container_name: nextcloud_nextcloud
        restart: unless-stopped
        volumes:
            - ./PersistentData/nextcloud:/var/www/html
        env_file:
            - ./nextcloud.env
        depends_on:
            - mariadb
            - redis

    mariadb:
        hostname: mariadb
        image: mariadb
        container_name: nextcloud_mariadb
        restart: unless-stopped
        command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --transaction-isolation=READ-COMMITTED --binlog-format=ROW  --innodb_read_only_compressed=OFF
        volumes:
            - ./PersistentData/mariadb:/var/lib/mysql
        env_file:
            - ./mariadb.env

    nginx:
        hostname: nginx
        image: nginx
        container_name: nextcloud_nginx
        restart: unless-stopped
        volumes:
            - ./config/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
            - ./config/ssl/<example.domain>/fullchain.pem:/etc/ssl/nginx/fullchain.pem
            - ./config/ssl/<example.domain>/privkey.pem:/etc/ssl/nginx/privkey.pem
            - ./PersistentData/nextcloud:/usr/share/nginx/html/nextcloud:ro
        ports:
            - "80:80"
            - "443:443"
        depends_on:
            - nextcloud

    redis:
        image: redis
        container_name: nextcloud_redis
        restart: unless-stopped

    cron:
        image: nextcloud:fpm
        container_name: nextcloud_cron
        restart: unless-stopped
        volumes:
            - ./PersistentData/nextcloud:/var/www/html
        entrypoint: /cron.sh
        depends_on:
            - mariadb
            - redis

```

### nextcloud.env

```txt {hl_lines=["1-3", "6-8"]}
NEXTCLOUD_ADMIN_USER=<nextcloud_admin_user_to_be_created>
NEXTCLOUD_ADMIN_PASSWORD=<nextcloud_admin_user_password_to_be_created>
NEXTCLOUD_TRUSTED_DOMAINS=<example.domain>

MYSQL_HOST=mariadb
MYSQL_DATABASE=<nextcloud_database_to_be_used>
MYSQL_USER=<nextcloud_database_user_to_be_used>
MYSQL_PASSWORD=<nextcloud_database_user_password_to_be_used>

REDIS_HOST=redis
REDIS_HOST_PORT=6379
```

### mariadb.env

```txt {hl_lines=["1-4"]}
MARIADB_ROOT_PASSWORD=<root_password>
MARIADB_DATABASE=<nextcloud_database_to_be_created>
MARIADB_USER=<nextcloud_database_user_to_be_created>
MARIADB_PASSWORD=<nextcloud_database_user_password_to_be_created>
MARIADB_AUTO_UPGRADE=1
```

## docker-compose文件解析

&emsp;&emsp;在Nextcloud的官方docker页面<https://hub.docker.com/_/nextcloud> 有docker-compose的示例配置文件

### nextcloud-fpm

&emsp;&emsp;Nextcloud中所有的数据文件存放在容器内的/var/www/html中，可以将其映射到外部存储。data和config文件存储在其中单独的文件夹，apps文件夹是随着nextcloud一起提供的，无需在意。

&emsp;&emsp;使用环境变量可以第一次安装时自动配置相关设置，以使用MariaDB/MYSQL为例，配置数据库需要设置以下几个环境变量：

- MYSQL_HOST: Hostname of the database server using mysql / mariadb.
- MYSQL_DATABASE: Name of the database using mysql / mariadb.
- MYSQL_USER: Username for the database using mysql / mariadb.
- MYSQL_PASSWORD: Password for the database user using mysql / mariadb.

&emsp;&emsp;设置如下环境变量可以在安装nextcloud时自动配置管理员用户名和密码：

- NEXTCLOUD_ADMIN_USER: Name of the Nextcloud admin user.
- NEXTCLOUD_ADMIN_PASSWORD: Password for the Nextcloud admin user.

&emsp;&emsp;设置如下环境变量可以在安装时自动添加可信域名：

- NEXTCLOUD_TRUSTED_DOMAINS: (not set by default) Optional space-separated list of domains

&emsp;&emsp;如果使用了redis（推荐），还应添加下列变量指出redis服务器信息：

- REDIS_HOST (not set by default) Name of Redis container
- REDIS_HOST_PORT (default: 6379) Optional port for Redis, only use for external Redis servers that run on non-standard ports.
- REDIS_HOST_PASSWORD (not set by default) Redis password

&emsp;&emsp;PHP相关限制：（未测试）

- PHP_MEMORY_LIMIT: (default 512M) This sets the maximum amount of memory in bytes that a script is allowed to allocate. This is meant to help prevent poorly written scripts from eating up all available memory but it can prevent normal operation if set too tight.
- PHP_UPLOAD_LIMIT: (default 512M) This sets the upload limit (*post_max_size* and *upload_max_filesize*) for big files. Note that you may have to change other limits depending on your client, webserver or operating system. Check the [Nextcloud documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_files/big_file_upload_configuration.html) for more information.

&emsp;&emsp;当然，可以使用_FILE替代直接在docke-compose设置环境变量以传递敏感信息。我没仔细看<https://hub.docker.com/_/nextcloud>

### mariadb

&emsp;&emsp;mariadb的数据库文件位于容器内的/var/lib/mysql，可以将其映射到容器外部。

&emsp;&emsp;当使用docker创建新数据库时，可以使用以下环境变量来调整MariaDB的初始化，从10.5.10和10.6开始以后的所有版本中，MARIADB_\* 和 MYSQL_\* 具有等效的功能，且MARIADB_\*具有更高的优先级。

&emsp;&emsp;下列四个环境变量(or equivalents, including *_FILE)必须设定一个，以确定roog用户密码：

- MARIADB_ROOT_PASSWORD: This specifies the password that will be set for the MariaDB root superuser account.
- MARIADB_ROOT_PASSWORD_HASH: In order to have no plaintext secret in the deployment, MARIADB_ROOT_PASSWORD_HASH can be used as it is just the hash of the password. The hash can be generated with SELECT PASSWORD('thepassword') as a SQL query.
- MARIADB_RANDOM_ROOT_PASSWORD: Set to a non-empty value, like yes, to generate a random initial password for the root user. The generated root password will be printed to stdout (GENERATED ROOT PASSWORD: .....).
- MARIADB_ALLOW_EMPTY_ROOT_PASSWORD: Set to a non-empty value, like yes, to allow the container to be started with a blank password for the root user.

&emsp;&emsp;其他的环境变量（可选）：

- MARIADB_DATABASE: This variable allows you to specify the name of a database to be created on image startup.
- MARIADB_USER: These are used in conjunction to create a new user and to set that user's password. This user will be granted all access (corresponding to GRANT ALL) to the MARIADB_DATABASE database.
- MARIADB_PASSWORD: Both user and password variables are required for a user to be created.
- MARIADB_PASSWORD_HASH: See MARIADB_ROOT_PASSWORD_HASH above for how to get a password hash for MARIADB_PASSWORD_HASH.
- MARIADB_AUTO_UPGRADE: Set MARIADB_AUTO_UPGRADE to a non-empty value to have the entrypoint check whether mysql_upgrade/mariadb-upgrade needs to run, and if so, run the upgrade before starting the MariaDB server.
- MARIADB_DISABLE_UPGRADE_BACKUP: Before the upgrade, a backup of the system database is created in the top of the datadir with the name system_mysql_backup_*.sql.zst. This backup process can be disabled with by setting MARIADB_DISABLE_UPGRADE_BACKUP to a non-empty value.

### nginx

&emsp;&emsp;编写配置文件nginx.conf并映射到容器内的/etc/nginx/nginx.conf，作为配置文件；同时还需要把nextcloud中的/var/www/html映射到nginx容器内部，作为网站内容。

### redis

### cron

&emsp;&emsp;官方文档中关于background jons的介绍：<https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/background_jobs_configuration.html>

&emsp;&emsp;使用docker部署cron时，[cron的运行需要一个独立的容器并和nextcloud共享数据文件，并且据说所占用的内存很少，不会为主机带来性能问题](https://github.com/nextcloud/docker/issues/1695#issuecomment-1043169925)；可以参考官方示例中使用docker-compose运行cron的配置文件：<https://github.com/nextcloud/docker/blob/master/.examples/docker-compose/insecure/mariadb/apache/docker-compose.yml>。

&emsp;&emsp;此外，还可以使用主机的cron并搭配docker exec命令实现定时的后台任务。

## 更新nextcloud版本

&emsp;&emsp;当使用 docker-compose 时，更新容器的新版本只需要运行:

```bash
docker-compose pull
docker-compose up -d
```

&emsp;&emsp;但是不能跨越多个大版本升级，此时需要修改nextcloud:\<version\>-fpm指定大版本号进行以此升级。

## nextcloud 配置文件

&emsp;&emsp;nextcloud 支持使用多个config.php配置文件；可以在/var/www/html/config/路径下放置\*.config.php文件实现对nextcloud的配置，且这些文件中配置的参数优先级高于config.php中的设定。

```php
<?php
$CONFIG = array (
  'remember_login_cookie_lifetime' => 0,
  'default_phone_region' => 'CN',
  'defaultapp' => 'calendar,files,dashboard',
);

```

&emsp;&emsp;***remember_login_cookie_lifetime*** 记住登录 cookie 的生命周期，为0则关闭浏览器后cookie立即失效。  
&emsp;&emsp;***default_phone_region*** 使用 ISO 3166-1 国家代码为 Nextcloud 服务器上的电话号码设置默认区域。  
&emsp;&emsp;***defaultapp*** 设置用户登录后首先进入的应用；参数为在点击Nextcloud顶部的“应用程序”菜单后，对应的URL名称，如 documents，calendar，gallery等。可以使用","分隔应用列表。设置多个应用名称时，如果用户没有进入列表中的第一个应用程序（如列表中的第一个应用没启用），则 Nextcloud 将尝试使用户进入第二个应用程序，依此类推。如果 Nextcloud 没有找到已启用的应用程序，则默认为仪表板应用程序。

## nginx 配置文件

### nginx.conf

```conf {hl_lines=[35, 44, 96, "157-158"]}
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    upstream php-handler {
        server nextcloud:9000;
    }

    server {
        listen 80;
        listen [::]:80;
        #server_name nextcloud.example.com;
        server_name <example.domain>;

        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        #server_name nextcloud.example.com;
        server_name <example.domain>;

        error_page 497 =301 https://$http_host$request_uri;

        ssl_certificate     /etc/ssl/nginx/fullchain.pem;
        ssl_certificate_key /etc/ssl/nginx/privkey.pem;

        ssl_protocols             TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
        ssl_ciphers               ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers on;
        ssl_ecdh_curve            secp384r1;

        # HSTS settings
        # WARNING: Only add the preload option once you read about
        # the consequences in https://hstspreload.org/. This option
        # will add the domain to a hardcoded list that is shipped
        # in all major browsers and getting removed from this list
        # could take several months.
        add_header Strict-Transport-Security "max-age=15768000; includeSubDomains; preload;" always;

        # set max upload size
        client_max_body_size 100G;
        fastcgi_buffers 64 4K;
                
        fastcgi_send_timeout 300s;
        fastcgi_read_timeout 300s; 

        # Enable gzip but do not remove ETag headers
        gzip on;
        gzip_vary on;
        gzip_comp_level 4;
        gzip_min_length 256;
        gzip_proxied expired no-cache no-store private no_last_modified no_etag auth;
        gzip_types application/atom+xml application/javascript application/json application/ld+json application/manifest+json application/rss+xml application/vnd.geo+json application/vnd.ms-fontobject application/x-font-ttf application/x-web-app-manifest+json application/xhtml+xml application/xml font/opentype image/bmp image/svg+xml image/x-icon text/cache-manifest text/css text/plain text/vcard text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy;

        # Pagespeed is not supported by Nextcloud, so if your server is built
        # with the `ngx_pagespeed` module, uncomment this line to disable it.
        #pagespeed off;
        
        # HTTP response headers borrowed from Nextcloud `.htaccess`
        add_header Referrer-Policy                      "no-referrer"   always;
        add_header X-Content-Type-Options               "nosniff"       always;
        add_header X-Download-Options                   "noopen"        always;
        add_header X-Frame-Options                      "SAMEORIGIN"    always;
        add_header X-Permitted-Cross-Domain-Policies    "none"          always;
        add_header X-Robots-Tag                         "none"          always;
        add_header X-XSS-Protection                     "1; mode=block" always;

        # Remove X-Powered-By, which is an information leak
        fastcgi_hide_header X-Powered-By;

        # Path to the root of your installation
        root /usr/share/nginx/html/nextcloud;

        # Specify how to handle directories -- specifying `/index.php$request_uri`
        # here as the fallback means that Nginx always exhibits the desired behaviour
        # when a client requests a path that corresponds to a directory that exists
        # on the server. In particular, if that directory contains an index.php file,
        # that file is correctly served; if it doesn't, then the request is passed to
        # the front-end controller. This consistent behaviour means that we don't need
        # to specify custom rules for certain paths (e.g. images and other assets,
        # `/updater`, `/ocm-provider`, `/ocs-provider`), and thus
        # `try_files $uri $uri/ /index.php$request_uri`
        # always provides the desired behaviour.
        index index.php index.html /index.php$request_uri;

        # Rule borrowed from `.htaccess` to handle Microsoft DAV clients
        location = / {
            if ( $http_user_agent ~ ^DavClnt ) {
                return 302 /remote.php/webdav/$is_args$args;
            }
        }

        location = /robots.txt {
            allow all;
            log_not_found off;
            access_log off;
        }
        
        # Make a regex exception for `/.well-known` so that clients can still
        # access it despite the existence of the regex rule
        # `location ~ /(\.|autotest|...)` which would otherwise handle requests
        # for `/.well-known`.
        location ^~ /.well-known {
            # The rules in this block are an adaptation of the rules
            # in `.htaccess` that concern `/.well-known`.

            location = /.well-known/carddav { return 301 $scheme://$http_host/remote.php/dav/; }
            location = /.well-known/caldav  { return 301 $scheme://$http_host/remote.php/dav/; }

            location /.well-known/acme-challenge    { try_files $uri $uri/ =404; }
            location /.well-known/pki-validation    { try_files $uri $uri/ =404; }

            # Let Nextcloud's API for `/.well-known` URIs handle all other
            # requests by passing them to the front-end controller.
            return 301 $scheme://$http_host/index.php$request_uri;
        }

        # Rules borrowed from `.htaccess` to hide certain paths from clients
        location ~ ^/(?:build|tests|config|lib|3rdparty|templates|data)(?:$|/)  { return 404; }
        location ~ ^/(?:\.|autotest|occ|issue|indie|db_|console)                { return 404; }

        # Ensure this block, which passes PHP files to the PHP process, is above the blocks
        # which handle static assets (as seen below). If this block is not declared first,
        # then Nginx will encounter an infinite rewriting loop when it prepends `/index.php`
        # to the URI, resulting in a HTTP 500 error response.
        location ~ \.php(?:$|/) {
            fastcgi_split_path_info ^(.+?\.php)(/.*)$;
            set $path_info $fastcgi_path_info;

            try_files $fastcgi_script_name =404;

            include fastcgi_params;
            # fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;
            fastcgi_param PATH_INFO $path_info;
            #fastcgi_param HTTPS on;

            fastcgi_param modHeadersAvailable true;         # Avoid sending the security headers twice
            fastcgi_param front_controller_active true;     # Enable pretty urls
            fastcgi_pass php-handler;

            fastcgi_intercept_errors on;
            fastcgi_request_buffering off;
        }

        location ~ \.(?:css|js|svg|gif)$ {
            try_files $uri /index.php$request_uri;
            expires 6M;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        location ~ \.woff2?$ {
            try_files $uri /index.php$request_uri;
            expires 7d;         # Cache-Control policy borrowed from `.htaccess`
            access_log off;     # Optional: Don't log access to assets
        }

        # Rule borrowed from `.htaccess`
        location /remote {
            return 301 /remote.php$request_uri;
        }

        location / {
            try_files $uri $uri/ /index.php$request_uri;
        }
    }
}
```

&emsp;&emsp;***root /usr/share/nginx/html/nextcloud;*** 设置了网站的根目录位置；需要通过docker-compose把nextcloud中的/var/www/html映射到 *root ...* 设置的路径下。

&emsp;&emsp;***fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name;*** 用来配置fastcgi传递nginx到php-fpm的信息，*/var/www/html* 需要设定为nextcloud容器中网站的路径；若nginx中的网站路径使用 *root ...* 设定为与nextcloud中路径相同(*/var/www/html*)，则可以使用  *$document_root* 代替 */var/www/html*。

## ~~MariaDB 10.6兼容性问题~~

&emsp;&emsp;nextcloud25已无此问题。  
&emsp;&emsp;nextcloud22和mariadb10.6有兼容性问题，对docker-compose文件中mariadb的command添加--innodb_read_only_compressed=OFF可暂时解决。

<https://github.com/nextcloud/server/issues/25436>

## nextcloud 维护模式

```bash
cd /var/www/nextcloud
sudo -u www-data php occ maintenance:mode --on
sudo -u www-data php occ maintenance:mode --off
```
