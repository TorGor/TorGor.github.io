---
layout: post
title:  centos supervisord
date:   2018-09-24 00:00:00 +0800
categories: document
tag: centos
---

* content
{:toc}


安装 crontabs 并自启动			{#install-crontabs}
===

`yum install crontabs `

`systemctl enable crond ` （设为开机启动）

`systemctl start crond`（启动crond服务）
 
`systemctl status crond `（查看状态） 


自定义shell		{#my-shell}
===

`touch certbot.sh`


    #! /bin/bash 
    certbot renew  
    echo "Certbot renew -- crontabs run"



用户自定义定时任务	{#my-shell}
===

`vi /etc/crontab `

可以看到： 

    Example of job definition: 
    .---------------- minute (0 - 59) 
    | .------------- hour (0 - 23) 
    | | .---------- day of month (1 - 31) 
    | | | .------- month (1 - 12) OR jan,feb,mar,apr ... 
    | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat 
    | | | | | 
    * * * * * user-name command to be executed 
    

解释： 

   分钟(0-59) 小时(0-23) 日(1-31) 月(11-12) 星期(0-6,0表示周日) 用户名 要执行的命令
+ `*/30 * * * root /usr/local/mycommand.sh `(每天，每30分钟执行一次 mycommand命令)
+ `* 3 * * * root /usr/local/mycommand.sh` (每天凌晨三点，执行命令脚本，PS:这里由于第一个的分钟没有设置，那么就会每天凌晨3点的每分钟都执行一次命令)
+ `0 3 * * * root /usr/local/mycommand.sh` (这样就是每天凌晨三点整执行一次命令脚本)
+ `*/10 11-13 * * * root /usr/local/mycommand.sh` (每天11点到13点之间，每10分钟执行一次命令脚本，这一种用法也很常用)
+ `10-30 * * * * root /usr/local/mycommand.sh` (每小时的10-30分钟，每分钟执行一次命令脚本，共执行20次)
+ `10,30 * * * * * root /usr/local/mycommand.sh` (每小时的10,30分钟，分别执行一次命令脚本，共执行2次）


保存生效	{#save}
===

加载任务,使之生效：

`crontab /etc/crontab`

查看任务：

`crontab -l `

`$ crontab -u 用户名 -l （列出用户的定时任务列表）`

查看执行日志：

`tail  /var/log/cron`

![/styles/images/centos/crontab/log-cron.png]({{ '/styles/images/centos/crontab/log-cron.png' | prepend: site.baseurl  }})
