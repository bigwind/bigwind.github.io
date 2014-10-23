---
layout: post
title : 博客服务器迁移到亚马逊AWS
category : 系统维护
tagline: ""
tags : [AWS, Nginx, PHP]
---
{% include JB/setup %}

之前博客放在了[HawkHost][1]上，用了将近三年时间，到今年11月份就要到期了。因为在HawkHost上买的是共享主机，自己没权限随便折腾，而且网站打开速度也比较慢，所以打算不再续费。目前比较火的是VPS，但是国外VPS，像[linode][2]和[DigitalOcean][3]，价格都偏贵，国内阿里云最低配的[ECS][4]每个月也得五六十元，其他公司的云服务又不靠谱。在浏览亚马逊的AWS时，看到它现在推出了一个[免费体验套餐][5]，体验时间是一年，配置跟阿里云最低配的ECS差不多，所以果断注册试用。迁移步骤如下：

- 在亚马逊AWS的管理后台创建一个EC2实例。
- 在创建好的EC2实例上搭建好LAMP环境，参见[如何搭建LAMP环境][6]。
- 把HawkHost上的WordPress程序和数据库迁移到EC2实例上。
- 修改域名的DNS记录到EC2实例的IP地址。

迁移完后，网站可以访问，但是遇到一个问题，文章页面打不开(404)。这个应该是因为Apache httpd的配置问题导致的。考虑到自己对Apache httpd的配置不熟，比较熟悉的是Nginx的配置，所以我决定把Web服务器换成Nginx，使用Nginx+PHP-FPM的方式。配置步骤如下：

- 停掉Apache httpd：

<pre lang="sh">
$ sudo service httpd stop
$ sudo chkconfig httpd off
</pre>

- 安装nginx和php-fpm：

<pre lang="sh">
$ sudo yum -y install nginx php-fpm
$ sudo service nginx start
$ sudo service php-fpm start
$ sudo chkconfig nginx on
$ sudo chkconfig php-fpm on
</pre>

- 在浏览器中访问http://blog.sousth.com/，出现nginx的默认页面。
- 注释掉/etc/nginx/nginx.conf中的server配置。
- 在/etc/nginx/conf.d中添加blog.conf，内容如下：

<pre lang="text">
server {
    listen 80;
    server_name blog.sousth.com;
    root /var/www/blog;

    location / {
        index index.php index.html index.htm;

        if (-f $request_filename) {
            expires 30d;
            break;
        }

        if (!-e $request_filename) {
            rewrite ^(.+)$ /index.php?q=$1 last;
        }
    }

    location ~ .php$ {
        fastcgi_pass   localhost:9000;  # port where FastCGI processes were spawned
        fastcgi_index  index.php;
        include /etc/nginx/fastcgi_params;
    }
}
</pre>

- 修改/etc/nginx/fastcgi_params，添加以下配置：

<pre lang="text">
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  PATH_INFO          $fastcgi_path_info;
fastcgi_param  PATH_TRANSLATED    $document_root$fastcgi_path_info;
</pre>

- 测试配置并重启Nginx：

<pre lang="sh">
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo service nginx restart
Stopping nginx:                                            [  OK  ]
Starting nginx:                                            [  OK  ]
</pre>

配置完成后，在浏览器中打开博客主页、文章页面和管理页面，都可以正常访问，大功告成。后续再将DNS服务由HawkHost迁移回GoDaddy。

参考资料：

- <http://www.sitepoint.com/lightning-fast-wordpress-with-php-fpm-and-nginx/>
- <http://wiki.nginx.org/PHPFcgiExample>

 [1]: https://www.hawkhost.com/
 [2]: https://www.linode.com/
 [3]: https://www.digitalocean.com/
 [4]: http://help.aliyun.com/view/11108189_13433720.html?spm=5176.383518.4.2.I3F1Dd
 [5]: https://aws.amazon.com/cn/free/
 [6]: http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/install-LAMP.html
