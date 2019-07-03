---
title: IPv6 DDNS
date: 2019-06-11
tags: network
categories: notes
---


## Cloudflare IPv6 DDNS


### 配置光猫

超级管理员密码 `nE7jA%5m` 登陆光猫，`网络` - `网络设置` - `IP模式` 选择 `IPv4%IPv6`。


### 路由器配置

启用 IPv6，
连接类型 `Native DHCPv6`、`Stateless:RA`，DNS 自动获取，内网 Ipv6 地址通过 DHCPv6 获取。


### 重新拨号

重新拨号，此时路由器与内网设备应该能正常获取到 IPv6 地址。

[测试地址](https://ipv6-test.com/)


### DDNS

由于 IPv6 地址动态获取且每次都不一样，所以需要做动态域名解析，确保外网能通过域名连接家里都设备。

通过 Cloudflare 提供的 [API](https://api.cloudflare.com/#dns-records-for-a-zone-update-dns-record) 进行 DDNS。

对于 `Padavan`、`OpenWrt` 等比较开放都路由器固件，可以使用下面的 [Shell 脚本](https://gist.github.com/zowiegong/6349d420789bb70aaebc7ce7eb1daccf)进行 DDNS:

```shell
#!/bin/sh

EMAIL=
CF_API_KEY=
CF_ZONE_ID=
CF_DNS_ID=

DNS_RECORD=
RECORD_TYPE=AAAA

ROUTER_NETWORK_DEVICE=

# 获取当前 DNS 记录
resolving_ip=$(curl -k -X GET "https://api.cloudflare.com/client/v4/zones/${CF_ZONE_ID}/dns_records/${CF_DNS_ID}" -H "X-Auth-Email:${EMAIL}" -H "X-Auth-Key:${CF_API_KEY}" -H "Content-Type: application/json" | awk -F '"' '{print $18}')

# 获取本机 ipv6 地址
current_ip=$(ip -o -6 addr list $ROUTER_NETWORK_DEVICE | awk '{print $4}' | cut -d/ -f1 | head -n 1)

echo "当前解析 IP: $resolving_ip"
echo "当前本机 IP: $current_ip"

# 没有变化
if [ $resolving_ip = $current_ip ];
    then
    echo "IP 未变化"

# 修改 DNS 记录
else
    echo "更新解析记录"
    curl -k -X PUT "https://api.cloudflare.com/client/v4/zones/${CF_ZONE_ID}/dns_records/${CF_DNS_ID}" -H "X-Auth-Email:${EMAIL}" -H "X-Auth-Key:${CF_API_KEY}" -H "Content-Type: application/json" --data '{"type":"'$RECORD_TYPE'","name":"'$DNS_RECORD'","content":"'$current_ip'"}'
fi

echo " "
```


### 定时更新

使用 crontab 定时更新解析记录。

`0 * * * * /path/cloudflare_ddns.sh >> /path/ddns.log`

每小时检查 IPv6 地址是否变化，然后更新解析记录，并输出日志到文件。