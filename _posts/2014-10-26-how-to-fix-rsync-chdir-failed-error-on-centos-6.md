---
layout: post
title : CentOS 6上如何解决rsync @ERROR: chdir failed问题
category : 系统管理
tagline: ""
tags : [Linux, CentOS, rsync, SELinux]
---
{% include JB/setup %}

## 问题描述

在RHEL或CentOS 6.x上如果希望rsync以守护进程方式运行，并且能够开机启动的话，网上很多文章中介绍的办法是把"/usr/bin/rsync --daemon"添加到/etc/rc.local文件中：

    # echo "/usr/bin/rsync --daemon" >> /etc/rc.local

但是，这样做的话存在一个问题，即每次重启服务器后客户端进行同步时会提示以下错误：

    @ERROR: chdir failed 
    rsync error: error starting client-server protocol (code 5) at main.c(1296) [receiver=2.6.8]

StactExchange上有一个[类似的问题](http://unix.stackexchange.com/questions/120679/configuring-anonymous-rsync-daemon)。

## 问题原因

遇到这个问题，可能的原因是：

- /etc/rsyncd.conf里配置的模块目录不存在（这个很容易检查出来）
- 与SELinux有关

检查是否与SELinux有关的方法是先临时关掉SELinux：

    # getenforce
    Enforcing        # 说明SELinux运行在enforcing模式
    # setenforce 0   # 使SELinux运行在permissive模式

然后检查是否还存在以上问题，如果问题不存在了，就说明确实与SELinux有关。如果确定与SELinux有关，可以进一步检查SELinux中与Rsync相关的Boolean数值：

    # getsebool -a | grep rsync 
    allow_rsync_anon_write --> off
    postgresql_can_rsync --> off
    rsync_client --> off
    rsync_export_all_ro --> off
    rsync_use_cifs --> off
    rsync_use_nfs --> off

我们发现rsync\_client和rsync\_export\_all\_ro这两项的值都是off，这就是问题所在。

## 解决办法

问题的解决办法很简单，只需要把rsync\_client和rsync\_export\_all\_ro这两项的值改成on即可。

    # setsebool -P rsync_export_all_ro 1
    # setsebool -P rsync_client 1
    # getsebool -a | grep rsync
    allow_rsync_anon_write --> off
    postgresql_can_rsync --> off
    rsync_client --> on
    rsync_export_all_ro --> on
    rsync_use_cifs --> off
    rsync_use_nfs --> off

关于SELinux的简单介绍可以参考[维基百科](http://en.wikipedia.org/wiki/Security-Enhanced_Linux)。RedHat的[官方文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/)对SELinux作了更详尽的介绍。另外，RedHat的官方文档中也有与rsync相关的配置例子：[⁠Configuration Examples](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Managing_Confined_Services/sect-Managing_Confined_Services-rsync-Configuration_Examples.html)，有兴趣的童鞋可以研究学习。

以上内容仅代表个人观点，如有不对之处，欢迎批评指正。
