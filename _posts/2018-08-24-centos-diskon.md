---
layout: post
title:  centos 磁盘初始化挂载
date:   2018-08-24 00:00:00 +0800
categories: document
tag: centos 
---

* content
{:toc}


列出所有硬盘			{#list-disks}
===

`fdisk -l | grep -P "Disk /dev/[a-z]{0,3}:"`

显示如图，可以看到有 sda，sdb

![/styles/images/centos/diskon/finddisks.png](https://torgor.github.io/styles/images/centos/diskon/finddisks.png)

因为磁盘较大原因，使用 parted 命令分区：

`parted /dev/sda`

依次执行的命令：

+	parted /dev/sda
+	p
+	mklabel gpt
+	y
+	mkpart
+	sda1
+	ext4
+	0
+	1000GB
+	Ignore
+	P

![/styles/images/centos/diskon/parted-cmd.png](https://torgor.github.io/styles/images/centos/diskon/parted-cmd.png)

查看分区：

`fdisk -l `

![/styles/images/centos/diskon/fdisk-l.png](https://torgor.github.io/styles/images/centos/diskon/fdisk-l.png)



格式化		{#formart-disk}
===

` mkfs.ext4 /dev/sda1`



创建挂载目录  {#create-data}
===

创建挂载目录 /data 在系统根目录

`cd ../`

`mkdir data`

`ll`

挂载卷到 /data 目录：

`mount /dev/sdb1 /data`

`vi /etc/fstab`

添加一行 

`/dev/sda1               /data                   ext4    defaults        1 1`

![/styles/images/centos/diskon/vim-fstab.png](https://torgor.github.io/styles/images/centos/diskon/vim-fstab.png)


重启系统 {#create-data}
===

重启并查看是否成功

![/styles/images/centos/diskon/success-df-h.png](https://torgor.github.io/styles/images/centos/diskon/success-df-h.png)


# 求关注
> 程序领域

![公众号](https://torgor.github.io/styles/images/my-public-ma.png)
![赞赏码](https://torgor.github.io/styles/images/my-zanshang-ma.png)





