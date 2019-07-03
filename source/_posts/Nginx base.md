---
title: Nginx 基本配置
date: 2017-12-18
tags:
  - linux
  - nginx
categories: notes
---


## 安装
ubuntu 系统安装 nginx
```shell
sudo apt-get install nginx
```

编译安装
```shell
./configure
make
sudo make install
```

编译参数
```shell
./configure \
--user=nginx                          \
--group=nginx                         \
--prefix=/etc/nginx                   \
--sbin-path=/usr/sbin/nginx           \
--conf-path=/etc/nginx/nginx.conf     \
--pid-path=/var/run/nginx.pid         \
--lock-path=/var/run/nginx.lock       \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module        \
--with-http_stub_status_module        \
--with-http_ssl_module                \
--with-pcre                           \
--with-file-aio                       \
--with-http_realip_module             \
--without-http_scgi_module            \
--without-http_uwsgi_module           \
--without-http_fastcgi_module
```
[more](http://nginx.org/en/docs/configure.html)

## 基本操作
启动(start)、重启(restart)，停止(stop) nginx
```shell
sudo /etc/init.d/nginx restart

# or
sudo service nginx restart

# reload
sudo nginx -s reload
```

## 配置文件语法
nginx 是模块化的系统

通过`configure`的参数编译可以配置模块

`http_gzip_static_module`负责压缩，`http_ssl_module`负责加密 ......

整个配置文件都是由指令来控制, events, http, server, 和 location等

配置文件 `/etc/nginx/nginx.conf` ， `nginx -t` 显示

### 主配置文件:
```
user www-data;
worker_processes 1;
pid /run/nginx.pid;

events {
  worker_connections 768;
}

http {

  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;

  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  gzip on;
  gzip_disable "msie6";

  include /etc/nginx/conf.d/*.conf;
  include /etc/nginx/sites-enabled/*;
}
```

指令和指令之间是有层级和继承关系的, 如 http 内的指令会影响到 server 的

假如要部署一个web服务, 在/etc/nginx/conf.d/目录下新增一个文件即可

http 和 events 还有 mail 是同级

### 一个简单的配置：
```
server {
  listen 80;
  root /home/yinsigan/foo;
  server_name foo.bar.com;
  location / {

  }
}
```
listen 监听的端口
root 网站根目录
server_name 指定域名

### 稍微详细的配置：
```
server {
    listen      80;
    server_name example.org www.example.org;
    root        /data/www;

    location / {
        index   index.html index.php;
    }

    location ~* \.(gif|jpg|png)$ {
        expires 30d;
    }

    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME
                      $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }
}
```
访问 `http://example.org` , 读取`/data/www/index.html` , 找不到就会读取`index.php` , 转发到 `fastcgi_pass` 里面的逻辑

访问 `http://example.org/about.gif` , 读取`/data/www/about.gif` ，缓存30天。


## 反向代理

### 最简单的反向代理
反向代理依赖于`ngx_http_proxy_module`这个module

反向代理服务器能代理的请求的协议包括http(s)，FastCGI，SCGI，uwsgi，memcached等

把 https 协议的图片请求反向代理到 http 协议的真实图片上
https协议的图片是不存在，而它有一个地址实际指向的内容是 http 协议中的图片。

```
# https
server {
  server_name www.example.com;
  listen       443;
  location /newchart/hollow/small/nsh000001.gif {
    proxy_pass http://image.sinajs.cn/newchart/hollow/small/nsh000001.gif;
  }
}
```

## gzip压缩 
需要nginx已经有编译过`ngx_http_gzip_module`

压缩是需要消耗CPU，但能提高传缩的速度

### 使用 

是否编译了`ngx_http_gzip_module`

    sudo nginx -V


输出 `--with-ngx_http_gzip_module` ok

配置nginx的gzip:

```
http {
        gzip on;
        gzip_disable "msie6";

        gzip_vary on;
        gzip_proxied any;
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        gzip_http_version 1.1;
        gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;
        server {
                location ~ ^/assets/ {
                   gzip_static on;
                   expires max;
                   add_header Cache-Control public;
                }        
        }
}
```

最重要的是 http 中 `gzip on` 、`gzip_types`

`gzip_vary` 等都是一些配置

在需要压缩的静态资源那里加上 

    gzip_static on;
    expires max;
    add_header Cache-Control public;


`sudo nginx -s reload` 重新加载生效

`Content-Encoding: gzip` 成功
