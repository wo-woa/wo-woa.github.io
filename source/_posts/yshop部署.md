---
title: yshop部署
date: 2021-2-3 15:22:11
tags: [Java,Linux]
categories: java
description: yshop部署
---

## Mysql部署

使用docker拉去mysql5.7的镜像

由于为了兼容之前的版本，解决sql_mode=only_full_group_by的问题 [链接](https://blog.csdn.net/qq_42175986/article/details/82384160)，需要修改配置文件（该作者的方法在Windows上没有问题，但是linux上需要判断版本来修改，可以参考下面的评论）。应该是可以通过挂载将配置文件放置服务器上修改就可以，但是不知道为什么就是挂载失败，故采用直接修改的方式参考 [链接](https://blog.csdn.net/laizidiyu151/article/details/108770487)，注意该作者install拼写错误。另外在我的环境中，my.cnf文件基本上就是空白的都是注释，所以不能直接添加配置，需要先添加mysqld如下。

```ini
[mysqld]
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```



挂载的方式

```
docker run -d -p 3307:3306 -v /home/mysql/conf:/etc/mysql -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name yshop_mysql mysql:5.7
```

另一种启动方式

```
docker run -di --name=yshop_mysql -p 3307:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=r	 centos/mysql-57-centos7
```

这种配置有一点区别，还有就是这种内置了一些软件如vim



## nginx部署

在nginx.conf的http内部添加include  vhost/*.conf;

之后需要添加的端口就直接添加文件到vhost就可以了，管理起来比较方便



后台的前端配置文件  ssl版本

```json
server
{
        listen 443 ssl;
        #listen [::]:81 default_server ipv6only=on;
	server_name www.yixiang.co;
	#ssl on;
	ssl_certificate httpssl/3414321_www.yixiang.co.pem;
	ssl_certificate_key httpssl/3414321_www.yixiang.co.key;
	ssl_session_timeout 5m;
	ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
   	ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    	ssl_prefer_server_ciphers on;
        index index.html;
        root /home/wwwroot/system/yshop;

	
        location / {
		try_files $uri $uri/ @router;
		index index.html;
	}
	location @router {
		rewrite ^.*$ /index.html last;
	}	


	location ~* \.(eot|ttf|woff)$ {
              #  add_header Access-Control-Allow-Origin *;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }
	
      
	access_log  /home/wwwlogs/yshop.log;
	
}
```

修改为本地端口http版

```json
server
{
	listen 8013;
	server_name localhost;
	#ssl on;
	index index.html;
	root /opt/yshop/yshop-qd;
    location / {
		try_files $uri $uri/ @router;
		index index.html;
	}
	location @router {
		rewrite ^.*$ /index.html last;
	}	
	location ~* \.(eot|ttf|woff)$ {
		#  add_header Access-Control-Allow-Origin *;
	}
	location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
	{
		expires      30d;
	}
	location ~ .*\.(js|css)?$
	{
		expires      12h;
	}

	access_log  /opt/yshop/yshop-qd.log;
	
}
```

解决了图片和css加载失败的问题



uniapp

```json
server
{
        listen 8080;
        server_name localhost;
        #ssl on;
        index index.html index.htm;
        root /opt/yshop/yshop-uniapp;
	
	location / {
                try_files $uri $uri/ @router;
                index index.html;
        }

        location @router {
                rewrite ^.*$ /index.html last;
        }

	location ~* \.(eot|ttf|woff)$ {
              #  add_header Access-Control-Allow-Origin *;
        }

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }
	
	access_log   /opt/yshop/yshop-uniapp.log;
}
```

