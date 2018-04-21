---
layout: post
tagline: "项目无Bug之日即是下线之时"
title: Nginx+Lua+Redis架构尝试
category : 系统架构
tags : [web, nginx, lua, redis]
---
## 简介
利用nginx的高并发,redis的高读性能搭建一个高可用的WEB服务

### 适用场景
多读少写的web服务

{{ site.excerpt_separator }}

## 系统搭建
### 下载nginx模块
```sass
 git clone https://github.com/simpl/ngx_devel_kit.git
 git clone https://github.com/chaoslawful/lua-nginx-module.git
 git clone https://github.com/agentzh/redis2-nginx-module.git
 git clone https://github.com/agentzh/set-misc-nginx-module.git
 git clone https://github.com/agentzh/echo-nginx-module.git
 # nginx 1.2 之后就内置此模块
 git clone https://github.com/catap/ngx_http_upstream_keepalive.git
```
### 安装nginx
```sass
  apt-get install libpcre3 libpcre3-dev libltdl-dev libssl-dev libjpeg62 libjpeg62-dev libpng12-0 libpng12-dev libxml2-dev libcurl4-openssl-dev libmcrypt-dev autoconf libxslt1-dev libgd2-noxpm-dev libgeoip-dev libperl-dev -y
  wget http://nginx.org/download/nginx-1.5.12.tar.gz
  tar zxvf nginx-1.5.12.tar.gz
  cd nginx-1.5.12
  ./configure --prefix=/usr/local/nginx --with-debug --with-http_addition_module \
--with-http_dav_module --with-http_flv_module --with-http_geoip_module \
--with-http_gzip_static_module --with-http_image_filter_module --with-http_perl_module \
--with-http_random_index_module --with-http_realip_module --with-http_secure_link_module \
--with-http_stub_status_module --with-http_ssl_module --with-http_sub_module \
--with-http_xslt_module --with-ipv6 --with-sha1=/usr/include/openssl \
--with-md5=/usr/include/openssl --with-mail --with-mail_ssl_module \
--add-module=../ngx_devel_kit \
--add-module=../echo-nginx-module \
--add-module=../lua-nginx-module \
--add-module=../redis2-nginx-module \
--add-module=../ngx_http_upstream_keepalive \
--add-module=../set-misc-nginx-module

  make
  make install
```
 * 可以在浏览器中输入 http://localhost, 如果显示Welcome to nginx!则表示nginx正常工作
 * nginx配置文件在 /usr/local/nginx/conf/nginx.conf
 
### 安装lua模块
```sass
 sudo apt-get install lua5.1
```
安装完可通过lua命令进入命令行界面
{% gist 9809438 %}

### 安装redis
```sass
 wget https://redis.googlecode.com/files/redis-2.6.14.tar.gz
 tar zxvf redis-2.6.14.tar.gz
 cd redis-2.6.14
 make 

 /etc/init.d/redis-server start
```

## 测试
 * ab -n 50000 -c 1024 "http://localhost/json?key=ttlsa"
 * ![ab测试结果]({{ site.img_root }}/ab-test1.png)
 * 可以看出大概有8000的TPS

### 功能测试

### 性能测试

## 参考
 * [nginx+lua+redis构建高并发应用](http://www.ttlsa.com/nginx/nginx-lua-redis/)
