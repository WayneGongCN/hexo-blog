---
title: Docker 安装 ownCloud
date: 2017-12-17
tags:
  - docker
  - javascript
categories: notes
---


## 安装内核模块包
```shell
$ sudo apt-get update

$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
```

## 使用 APT 镜像源 安装

### 添加使用 HTTPS 传输的软件包以及 CA 证书
```shell
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

### 添加软件源的 GPG 密钥
```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

### 向 `source.list` 中添加 Docker 软件源
```shell
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```

## 安装 Docker CE
更新 apt 软件包缓存，并安装 `docker-ce`：
```shell
$ sudo apt-get update

$ sudo apt-get install docker-ce
```

## 启动 Docker CE
```shell
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

## 测试 Docker 是否安装正确
```shell
$ docker run hello-world

Unable to find image 'hello-world:latest' locally
......
Hello from Docker! ......

```
有如上输入则代表安装成功


## 配置 docker-compose 

### 下载最新版本的Docker Compose
```shell
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```
[检查最新版本](https://github.com/docker/compose/releases)，版本号自行更改

### 对二进制文件应用可执行权限
```shell
$ sudo chmod +x /usr/local/bin/docker-compose
```

### 测试
```shell
$ docker-compose --version
docker-compose version 1.17.0, build 1719ceb
```

### 编写配置 docker-compose.yml
```yml
version: '2'
services:
  owncloud:
    image: owncloud
    links:
      - mysql:mysql
    volumes:
      - "/data/db/owncloud/data:/var/www/html/data"
      - "/data/db/owncloud/config:/var/www/html/config"
      - "/data/db/owncloud/apps:/var/www/html/apps"
    ports:
      - 8000:80
  mysql:
    image: mysql
    volumes:
      - "/data/db/mysql:/var/lib/mysql"
    ports:
      - 3306:3306
    environment:
      MYSQL_ROOT_PASSWORD: "password"
      MYSQL_DATABASE: ownCloud
```

### docker-compose 运行和停止
`docker-compose` 必须在 `docker-compose.yml` 文件所在目录中执行

```shell
docker-compose 后台启动
$ docker-compose up -d

docker-compose 查看状态
$ docker-compose ps

docker-compose 停止和删除
$ docker-compose stop
$ dcoker-compose rm

相当上面两条命令
$ dcoker-compose down
```

## 启动并配置 owncloud
```shell
$ docker-compose up -d
```
![enter image description here](https://i.imgur.com/UtGufzX.jpg)

> 数据库用户：root
> 数据库密码：password
> 数据库名：owncloud
> 数据库主机：mysql