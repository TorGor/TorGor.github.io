---
layout: post
title:  centos shadowsock 科学上网搭建
date:   2019-04-24 00:00:00 +0800
categories: document
tag: centos
---

* content
{:toc}


安装ssserver			{#install-ssserver}
===
 
`yum install python-setuptools && easy_install pip`

`pip install shadowsocks`


启动方式		{#start-way}
===

## Ssserver 启动

`sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start`

`sudo ssserver -p 8389 -k 111111-m rc4-md5 --user username -d start`

`sudo ssserver -d stop`

`tail /var/log/shadowsocks.log`



## 自制service 启动
`vim /etc/shadowsocks.json`
```
   {
   	"server":"0.0.0.0",
   	"local_address":"127.0.0.1",
   	"local_port":1086,
   	"timeout":300,
   	"method":"rc4-md5",
   	"port_password":
   	{
   		"8388":"111111",
   		"8389":"111111",
   		"8390":"111111"
   	}
   }
```
### 制作service ：

`vim /etc/systemd/system/shadowsocks.service1`

```
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```


## 开机自启动：

`systemctl daemon-reload`

`systemctl enable shadowsocks`

`systemctl start shadowsocks`


