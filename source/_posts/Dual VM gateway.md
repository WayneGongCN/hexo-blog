---
title: 双虚拟机隔离中网关机的配置
date:  2018-06-11
tags: privacy
categories: notes
---


## 2018-04-03 更新

推荐使用 [whonix](https://www.whonix.org/),安全性更高，基本免配置开箱即用。

---


## 使用虚拟机保护隐私

更多内容，见
[如何搭配“多重代理”和“多虚拟机”](https://program-think.blogspot.com/2015/04/howto-cover-your-tracks-8.html)


## 双虚拟机方案

网关机器使用两张网卡， `网络地址转换(NAT)`网卡与宿主机连接访问不安全的网络， `Host-Only` 与需要上网的机器连接通过网关机内的代理端口上网。


## 网关机的配置

- 系统:
    - [Ubuntu 18 server](https://www.ubuntu.com/download/server)

- VirtualBox: 
    - 硬盘、CPU、内存按需分配，
    - 网卡1 `Host-Only` , 网卡2 `网络地址转换(NAT)`。

- 更新
    - `sudo apt update`
    - `sudo apt upgrade`


#### 安装 shadowsocks

```
sudo apt-get install python-pip
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```


#### 配置 sslocal

配置文件：`/etc/shadowsocks/sslocal.json`
```json
{
    "server": "my_server_ip",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "mypassword",
    "timeout": 300,
    "method": "aes-256-cfb",
    "fast_open": false
}
```


#### sslocal 开机启动

配置文件：`/etc/systemd/system/sslocal.service`
```
[Unit]
Description=Daemon to start Shadowsocks Client
Wants=network-online.target
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/sslocal -c /etc/shadowsocks/sslocal.json --pid-file /var/run/sslocal.pid --log-file /var/log/sslocal.log   

[Install]
WantedBy=multi-user.target
```

执行 `systemctl enable /etc/systemd/system/shadowsocks.service`


#### 安装 tor

`sudo apt install tor`


#### 配置 tor

配置文件：`/etc/tor/torrc`
```
SOCKSPort 0.0.0.0:9050
Socks5Proxy 0.0.0.0:1080
```
tor 开机自启无需配置
