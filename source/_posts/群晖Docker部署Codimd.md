---
title: 群晖 Docker 部署 Codimd
date: 2020-03-22 16:28:58
tags:
    - Docker
    - 群晖
categories:
---

## 前言

在线文档协同编辑有很多选择，但是大多都没有支持 Markdown。

发现一款 Markdown 在线协同编辑平台 [hackmd](https://hackmd.io)（貌似打不开），社区免费版叫做 [Codimd](https://github.com/hackmdio/codimd)。

在 [demo.codimd.org](https://demo.codimd.org/) 体验了几天非常不错。

支持多人实时在线编辑、简单的版本控制、样式简洁、公式、目录、代码高亮、各种图表支持的也非常到位。

但是看着域名中的 demo 害怕突然哪天跟 hackmd.io 一样打不开就完蛋了，家里星际蜗牛 24 小时开机，打算部署在黑群晖的 Docker 中。


## 部署

黑群晖没有装 `docker-compose`，不过群晖的 Docker 控制台对新手也非常友好。

根据 [https://github.com/codimd/container](https://github.com/codimd/container) 仓库中的 `docker-compose.yml` 来简单配置一下就可以了。

提前拉一下需要的 images：
1. MySQL （Docker 控制台 - 注册表 - 搜 MySQL - 下载 latest 就可以了）
2. `quay.io/codimd/server:1.6.0` （Docker 控制台 - 映像 - 新增 - 从 URL 添加 ）


### Docker 部署 MySQL

试了下 Mariadb，但是环境变量貌似无法生效，换成 MySQL 也一样。

1. 选中 MySQL 映像，启动。
2. 勾选高权限、设置优先级（可选）
3. 进入高级设置，自动重启勾上（可选）
4. 选择“卷”映射数据目录到 `/var/lib/mysql`，映射 [ uft8.cnf ](https://github.com/codimd/container/blob/master/resources/utf8.cnf) 到 `/etc/mysql/conf.d/utf8.cnf`
5. 配置环境变量
    - `MYSQL_USER = hackmd`
    - `MYSQL_PASSWORD = xxxxxx`
    - `MYSQL_DATABSE = hackmd`
    - `MYSQL_ROOT_PASSWORD = xxx`
7. 其余保持默认，应用。


### Docker 部署 codimd/server

1. 选中 MySQL 映像，启动。
2. 勾选高权限、设置优先级（可选）
3. 进入高级设置，自动重启勾上（可选）
4. 端口映射自行按需配置
5. 选择 “链接” 选择前面部署的 mysql 容器名称设置别名为 `database`
6. 配置环境变量
    - `CMD_DB_URL = mysql:hackmd:xxx@database/hackmd` （格式为：`<databasetype>://<username>:<password>@<hostname>/<database>` 自行修改）
7. 其余保持默认，应用。


### 验证

分别查看 MySQL 、codimd/server 两个容器的日志有无错误信息。

MySQL 容器日志有 `[Server] X Plugin ready for connections. Socket: '/var/run/mysqld/mysqlx.sock' bind-address: '::' port: 33060` 类似即可。

codimd/server 容器日志有 `HTTP Server listening at 0.0.0.0:3000` 类似即可。