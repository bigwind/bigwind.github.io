---
layout: post
title : CentOS 6上如何解决rsync @ERROR: chdir failed问题
category : 系统管理
tagline: ""
tags : [Linux, CentOS, SELinux, rsync]
---
{% include JB/setup %}

如果希望rsync以守护进程方式运行，并且能够开机启动的话，网上文章中介绍的一般办法是把/usr/bin/rsync --daemon添加到/etc/rc.local文件中：

    $ echo "/usr/bin/rsync --daemon" >> /etc/rc.local

但是，这样做的话存在一个问题：如果使用的是默认开启SELinux的Linux发行版（比如RHEL、Fedora或CentOS等），每次重启服务器后客户端进行同步时会提示以下错误：

    @ERROR: chdir failed 
    rsync error: error starting client-server protocol (code 5) at main.c(1296) [receiver=2.6.8]


