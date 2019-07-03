---
title: Linux 基本安全设置
date: 2018-06-22
tags: linux
categories: notes
---

记录一点基本的安全设置。

服务器环境 Debian 8 x64

## 添加用户
```shell
useradd username
mkdir /home/username
mkdir /home/username/.ssh
chmod 700 /home/username/.ssh

usermod -s /bin/bash username
```

## ssh key 验证
```shell
# 添加公钥
vim /home/username/.ssh/authorized_keys

chmod 400 /home/username/.ssh/authorized_keys
chown username:username /home/username -R

vim /etc/ssh/sshd_config
```

修改如下配置
```
PermitRootLogin no
PasswordAuthentication no
AllowUsers username@你的VPN或固定IP
AddressFamily inet
```

重启生效 `service ssh restart`

## 配置 sudo
```shell
# 配置 sudo 密码
passwd username

apt install sudo
visudo
```

添加 %sudo 组
```
root    ALL=(ALL) ALL
%sudo   ALL=(ALL:ALL) ALL
```

用户添加到 sudo 组
```shell
usermod -aG sudo username
exec su -l username
```
## 配置防火墙
[详细用法](http://wiki.ubuntu.org.cn/Ufw%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97)

安装ufw
`apt install ufw`

默认拒绝所有
`ufw default deny`

添加需要开放的端口
`ufw allow 22`

启动 ufw
`ufw enable`

查看 ufw 状态
`ufw status`

## 安装 Fail2ban
`apt install fial2ban`
Fail2ban 软件包主动拦阻可疑行为。
这里使用默认配置，不做修改。

