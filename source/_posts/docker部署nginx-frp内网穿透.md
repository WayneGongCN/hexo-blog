---
title: Docker部署 Nginx + frp 内网穿透
date: 2020-03-23 14:17:20
tags:
  - Docker
  - frp
  - nginx
categories:
---


## 前言

由于没有公网 IP，加上黑群晖没有洗白，无法进行外网的远程连接。

通过 frp 内网穿透后可以通过 `sub.domain.com:xxx` 的形式访问到内网的 web 服务，但是带上端口号十分不优雅也难以记忆。

在公网服务器上部署 Nginx 反向代理到 frp 的特定端口，即可在使用内网服务时不需要端口号了。


## 服务部署

通过 docker-compose 启动 Dcoker 容器，然后通过 links 将 nginx 与 frp 容器连接，即可在 nginx 容器中将请求转发到 frp 容器。


前置条件：
- 服务端需要 docker 环境及 docker-compose
- nginx 镜像
- snowdreamtech/frps 镜像

frp docker 部署有个坑，一开始 `fatedier/frp` 镜像一直不能成功访问，报 `read error` 的错误。

后来一看镜像更新时间已经是 4 年前了，换了 Pulls 量最多的 `snowdreamtech/frps` 解决。


### 目录结构

```
.
├── docker-compose.yml
├── frp
│   └── config
│       └── frps.ini
└── nginx
    ├── config
    │   ├── conf.d
    │   │   └── default.conf
    │   └── nginx.conf
    └── logs
        ├── access.log
        └── error.log
```

`docker-compose.yml` 内容如下：
```yml
version: "3"
services:
  nginx:
    image: nginx
    container_name: nginx
    restart: always
    volumes:
      - ./nginx/config/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/config/conf.d:/etc/nginx/conf.d
      - ./nginx/logs:/var/log/nginx
    ports:
      - "80:80"
    environment:
      - NGINX_PORT=80
    links:
      - frp
  
  frp:
    image: snowdreamtech/frps
    container_name: frp
    restart: always
    ports:
      - "7000:7000" # 里的端口与 frps.ini 中 bind_port 保持一致
    volumes:
      - ./frp/config/frps.ini:/etc/frp/frps.ini
```

nginx 的 `default.conf` 配置文件（可以先 run 一个 nginx 容器然后 通过 `docker cp` 得到默认配置）:
```conf
server {
    listen       80;
    server_name  *.domain.com;

    location / {
        proxy_pass http://frp:8888; # 这里的端口与 frps.ini 中 vhost_http_port 保持一致
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-Proto https;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

`frps.ini` 配置文件：
```ini
[common]
bind_port = 7000
vhost_http_port = 8888
```

`frpc.ini` 配置文件（这里是 frp 客户端配置，仅做参考）:
```ini
[common]
server_addr = x.x.x.x
server_port = 7000

[xx sercer]
type = http
local_port = xxx
custom_domains = sub.domain.com
```

完成所有配置之后在 `docker-compose.yml` 所在的目录执行 `docker-compose up` 完成验证即可。


## 验证

按照上述配置之后，原来通过 `sub.domain.com:8888` 才能访问的内网 web 服务现在只需要通过 `sub.domain.com` 访问即可。