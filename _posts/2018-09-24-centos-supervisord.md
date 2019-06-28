---
layout: post
title:  centos supervisord
date:   2018-09-24 00:00:00 +0800
categories: document
tag: centos
---

* content
{:toc}


安装：			{#install}
===

`pip install supervisor `



生成配置文件：		{#my-shell}
===

`echo_supervisord_conf > /etc/supervisord.conf`


启动	    {#start}
===

`supervisord -c /etc/supervisord.conf`


查看是否运行 {#viewstatus}
===
`ps aux | grep supervisord`

配置 
===
`vim /etc/supervisord.conf` 

添加如下内容：

```
[include]
files=/etc/supervisor/*.conf #若你本地无/etc/supervisor目录，请自建
cd /etc/supervisor
```

以ss为例，写一个守护线程：
===

`vim /etc/supervisor/shadowsocks.conf`

```
[program:shadowsocks]
	command = ssserver -c /etc/shadowsocks.json
	user = root
	autostart = true
	autoresart = true
	stderr_logfile = /var/log/supervisor/ss.stderr.log
	stdout_logfile = /var/log/supervisor/ss.stdout.log
	
```
重新加载配置：
`supervisorctl reload`

命令 
===

supervisord : 启动
supervisorctl reload :修改完配置文件后重新启动
supervisorctl status :查看supervisor监管的进程状态 
supervisorctl start 进程名 ：启动XXX进程 
supervisorctl stop 进程名 ：停止XXX进程 
supervisorctl stop all：停止全部进程，注：start、restart、stop都不会载入最新的配置文件。



