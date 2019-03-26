---
title: nginx实现前后端分离
date: 2019-03-22 15:33:24
categories:
 - nginx
tags:
 - nginx
 - 前后端分离
---





## nginx实现前后端分离





### 1、安装nginx

[参考 https://blog.csdn.net/mybook201314/article/details/73604640](https://blog.csdn.net/mybook201314/article/details/73604640)



### 2、配置conf



~~~
路径：nginx/conf/nginx.conf
~~~





#### 2.1、设置前端代码位置

~~~shell
location / {
	# root写前端代码位置 Linux 如：/home/frontend/xxx
	root   E:\\desktop\\HTML;
	index  index.html index.htm;
}
~~~



#### 2.2、配置后台请求拦截

~~~shell
#后台跳转
#拦截所有前端 以 /api 开始的请求
#转发到 后端服务 http://localhost:8080/backend_project_name/api/......
location ~ ^/api {

	proxy_set_header Host $host;
	proxy_set_header X-Real-Ip $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forworded-For $http_x_forwarded_for;
	
	# 存 cookie
	proxy_cookie_path //backend_project_name/ / ;
	
	# 重写 后台请求项目名称
	# 若后台项目名称为/，或者没有，则此不用配置
	rewrite /(.+)$ /backend_project_name/$1 break;
	
	# 后台服务路径
	proxy_pass   http://localhost:8080;
}
~~~



#### 2.3、完整配置



~~~shell

#定义Nginx运行的用户和用户组
#user  nobody;

#nginx进程数，建议设置为等于CPU总核心数
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#进程文件
#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


#http服务器
http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

	## 自定义服务 可以定义多个，监听不同端口
    server {
	
		// nginx服务监听端口
        listen       80;
		
		// nginx服务名称
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

		// 前端位置配置
        location / {
			# root写前端代码位置 Linux 如：/home/frontend/xxx
            root   E:\\desktop\\HTML;
            index  index.html index.htm;
        }

        
        #配置500 
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

		#后台跳转
		#拦截所有前端 以 /api 开始的请求
		#转发到 后端服务 http://localhost:8080/backend_project_name/api/......
        location ~ ^/api {
		
			proxy_set_header Host $host;
			proxy_set_header X-Real-Ip $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forworded-For $http_x_forwarded_for;
			
			# 设置 cookie
			proxy_cookie_path //backend_project_name/ / ;
			
			# 重写 后台请求项目名称
			# 若后台项目名称为/，或者没有，则此不用配置
			rewrite /(.+)$ /backend_project_name/$1 break;

			# 后台服务路径
            proxy_pass   http://localhost:8080;
        }
    }
}

~~~



















