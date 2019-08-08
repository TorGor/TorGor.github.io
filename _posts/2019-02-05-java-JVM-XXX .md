---
layout: post
title:  JVM 调优及工具使用
date:   2019-02-05 00:00:00 +0800
categories: document
tag: JVM
---

* content
{:toc}


### jstat （JVM statistics Monitoring）：虚拟机统计信息监视工具

#### option
| 选项 | 作用 |
| ---- | ---- |
| -class | 监视类装载、卸载数量、总空间以及类装载所耗费的时间  |
| -gc | 监视Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量、已用空间、GC时间合计等信息   |
| -gccapacity | 监视内容与-gc基本相同，但输出主要关注Java堆各个区域使用到的最大、最小空间   |
| -gcutil | 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比  |
| -gccause | 与 -gcutil 功能一样，但是会额外输出导致上一次GC产生的原因  |
| -gcnew | 监视新生代GC状况  |
| -gcnewcapacity | 监视内容与 -gcnew 基本相同，输出主要关注使用到的最大、最小空间  |
| -gcold  | 监视老年代GC状况  |
| -gcoldcapacity | 监视内容与 -gcold 基本相同，输出主要关注使用到的最大、最小空间  |
| -gcpermcapacity | 输出永久代使用到的最大、最小空间  |
| -compiler | 输出JIT编译器编译过的方法、耗时等信息   |
| -printcompilation | 输出已经被JIT编译的方法  |


|  选项   | 作用  |
|  ----  | ----  |
| -class  | 监视类装载、卸载数量、总空间以及类装载所耗费的时间 |
| 单元格  | 单元格 |

![jstat 命令]({{ '/styles/images/jmm/JVM-jstat-help.png' | prepend: site.baseurl  }})


**C即Capacity 总容量，U即Used 已使用的容量**
* S0C : survivor0区的总容量
* S1C : survivor1区的总容量
* S0U : survivor0区已使用的容量
* S1C : survivor1区已使用的容量
* EC : Eden区的总容量
* EU : Eden区已使用的容量
* OC : Old区的总容量
* OU : Old区已使用的容量
* PC : 当前perm的容量 (KB)
* PU : perm的使用 (KB)
* YGC : 新生代垃圾回收次数
* YGCT : 新生代垃圾回收时间
* FGC : 老年代垃圾回收次数
* FGCT : 老年代垃圾回收时间
* GCT : 垃圾回收总消耗时间


#### example:
```jstat -gc 21332 2000 20```

每隔2000ms 输出 pid=21332 的 gc 情况，一共输出 20 次


### centos yum 安装 jdk 没有 jstat ，jps 命令解决办法

yum install java-1.8.0-openjdk-devel -y





