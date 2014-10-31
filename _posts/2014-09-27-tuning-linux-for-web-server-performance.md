---
layout: post
title : Web服务器性能优化之Linux系统配置
category : 系统管理
tagline: ""
tags : [Linux, Web, 性能优化]
---
{% include JB/setup %}

Linux系统有些配置，比如TCP连接配置、进程允许打开的最大文件数、最大子进程数等，会影响Web服务器的性能。

## 修改网络配置

    # sysctl -a | grep tcp_tw
    net.ipv4.tcp_tw_recycle = 0
    net.ipv4.tcp_tw_reuse = 0
    # sysctl -a | grep tcp_syncookies
    net.ipv4.tcp_syncookies = 1
    # sysctl -a | grep tcp_fin_timeout
    net.ipv4.tcp_fin_timeout = 60
    # sysctl -a | grep somaxconn
    net.core.somaxconn = 128

相应的改进是修改/etc/sysctl.conf，添加以下几行：

    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_fin_timeout = 5
    net.core.somaxconn = 2048

执行以下命令使配置生效：

    # sysctl -p

## 修改系统限制

    # ulimit -a
    core file size          (blocks, -c) 0
    data seg size           (kbytes, -d) unlimited
    scheduling priority             (-e) 0
    file size               (blocks, -f) unlimited
    pending signals                 (-i) 111205
    max locked memory       (kbytes, -l) 64
    max memory size         (kbytes, -m) unlimited
    open files                      (-n) 1024
    pipe size            (512 bytes, -p) 8
    POSIX message queues     (bytes, -q) 819200
    real-time priority              (-r) 0
    stack size              (kbytes, -s) 10240
    cpu time               (seconds, -t) unlimited
    max user processes              (-u) 1024
    virtual memory          (kbytes, -v) unlimited
    file locks                      (-x) unlimited

相应的改进： 

1\.修改/etc/security/limits.conf，添加以下行：

    *                -       nofile          102400

2\.修改/etc/security/limits.d/90-nproc.conf，修改以下配置：

    *       soft    nproc     1024 # -> 10240

最后，重启设备使配置生效：

    # reboot
