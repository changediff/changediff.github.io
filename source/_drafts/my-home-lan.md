---
title: 家庭局域网折腾
tags: ["networt", "lan", "路由", "局域网"]
---

# Debian
```
apt install isc-dhcp-server
vim /etc/default/isc-dhcp-server

ip addr add 192.168.4.1/24 dev eth1
```