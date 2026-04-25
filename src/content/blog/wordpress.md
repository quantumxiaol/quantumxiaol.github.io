---
title: "Wordpress Deployment"
description: "在Linux上部署Wordpress"
pubDate: "Jul 08 2025"
# heroImage: "../../assets/blog-placeholder-3.jpg"
---


# Wordpress

## 安装Nginx、MariaDB和PHP：

    sudo apt install nginx mariadb-server php-fpm php-mysql php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip wget -y

## 数据库准备
启动 MariaDB 并设置开机启动

    sudo systemctl start mariadb
    sudo systemctl enable mariadb

运行MariaDB的安全安装脚本并创建一个新的数据库和用户用于WordPress。

    sudo mysql_secure_installation

登录MariaDB并创建数据库和用户：

    mysql -u root -p

在 MySQL 中执行以下命令

    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'your_password';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

## 安装Wordpress

创建不能登录的系统用户 wordpress

    sudo adduser --system --group --shell /usr/sbin/nologin wordpress

下载最新版WordPress并解压到Web根目录

    cd /tmp
    wget https://wordpress.org/latest.tar.gz
    tar xzvf latest.tar.gz
    sudo cp -a /tmp/wordpress/. /var/www/html/

设置文件权限

    sudo chown -R wordpress:wordpress /var/www/html
    sudo find /var/www/html -type d -exec chmod 755 {} \;
    sudo find /var/www/html -type f -exec chmod 644 {} \;

配置 PHP-FPM 使用 wordpress 用户运行

    sudo nano /etc/php/8.1/fpm/pool.d/www.conf

修改以下

    user = wordpress
    group = wordpress
    listen.owner = www-data
    listen.group = www-data

重启，注意实际版本号

    sudo systemctl restart php8.1-fpm

## 配置Nginx
创建一个Nginx服务器块配置文件以服务您的WordPress站点。

    sudo nano /etc/nginx/sites-available/wordpress
    

添加基本配置，完成后保存退出。需要注意域名

    server {
        listen 80;
        server_name example.com www.example.com;
        root /var/www/html;

        index index.php index.html index.htm;
        client_max_body_size 20M;
        location / {
            try_files $uri $uri/ /index.php?$args;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.1-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

        location ~ /\.ht {
            deny all;
        }
    }

启用该配置并重启Nginx。

    sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
    sudo systemctl restart nginx

测试 Nginx 配置并重启服务

    sudo nginx -t
    sudo systemctl restart nginx

## 配置 WordPress

    sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
    sudo nano /var/www/html/wp-config.php

修改 wp-config.php 文件

    sudo nano /var/www/html/wp-config.php

修改

    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wordpress');
    define('DB_PASSWORD', 'your_password');
    define('DB_HOST', 'localhost');


访问网站：http://your_server_ip

WordPress 将引导进入安装向导。

在向导中填写数据库信息：

    数据库名：wordpress
    数据库用户名：wordpress
    数据库密码：your_password
    数据库主机：localhost
    表前缀：可保留默认 wp_

点击“提交”后继续安装，设置管理员账号和博客名称即可完成安装。

## error

检查是否有些文件没有对应的访问权限