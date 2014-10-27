---
layout: post
title : CentOS 6上rsync与SELinux有关的问题
category : 系统维护
tagline: ""
tags : [Linux, CentOS, SELinux, rsync]
---
{% include JB/setup %}

如果希望rsync以守护进程方式运行并且能够开机自启动的话，可以修改/etc/rc.local文件：

    $ echo "/usr/bin/rsync --daemon" >> /etc/rc.local

如果使用的是带有SELinux的Linux发行版，比如RHEL或CentOS，客户端提示的错误信息是：

    @ERROR: chdir failed 
    rsync error: error starting client-server protocol (code 5) at main.c(1296) [receiver=2.6.8]

但是如果
