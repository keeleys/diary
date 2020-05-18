title: nginx学习
tags: nginx
categories: nginx
description: nginx常用方法和配置
date: 2015-06-30 17:13:39
---

## 官网资料
### 下载地址
[http://nginx.org/](http://nginx.org/)

### wiki
[https://www.nginx.com/resources/wiki/start/](https://www.nginx.com/resources/wiki/start/)

<!-- more -->

## 启动

直接运行 `nginx.exe` 或者cmd 执行 `start nginx`以守护进程方式启动

1. 有可能一闪而过 那么需要查看`logs/error.log`, 查看错误日志，一般为端口冲突
2. 打开`conf/nginx.conf` ,找到如下代码,修改端口为81或者其他,再次运行
```conf
server {
  #监听81端口 localhost:81
  listen       81;
  server_name  localhost;

  #charset koi8-r;

  #access_log  logs/host.access.log  main;

  location / {
      root   html;
      index  index.html index.htm;
  }
  ...
}
```

## 一些命令 在nginx根目录 cmd执行

1. 基本
```
nginx -V //查看NGINX版本和配置文件路径
nginx -s stop          // 停止nginx
nginx -s reload       // 重新加载配置文件
nginx -s quit
nginx -c ~/Desktop/nginx.conf //指定配置文件启动
nginx -t //测试配置文件的正确性
```

## 反向代理

### 基础模式
```conf
server {
    #监听80端口 localhost:80
    listen       80;
    #nginx会匹配请求头中的host和server_name
    server_name  localhost;
    #charset koi8-r;
    #access_log  logs/host.access.log  main;
    location / {
        proxy_pass http://localhost:9080;
    }
```

### 反向负载均衡
```conf
#真实转发的地址
upstream real_path{
    server localhost:9080 weight=1 max_fails=2 fail_timeout=30s;
    server localhost:7080 weight=1 max_fails=2 fail_timeout=30s;
}
server {
    #监听80端口 localhost:80
    listen       80;
    server_name  localhost;
    #charset koi8-r;
    #access_log  logs/host.access.log  main;
    location / {
        proxy_pass http://real_path;
    }
}   
```

## nginx站点

### 在一个server块中配置多个站点
```conf
server
{
  listen 80;
  server_name ~^(.+)?\.howtocn\.org$;
  set $www_root $1; //正则匹配到的站点前缀储存起来，避免多个正则冲突
  index index.html;
  if ($host = ttianjun.com){ //重定向
  rewrite ^ http://www.ttianjun.com permanent;
  }
  root /data/wwwsite/ttianjun.com/$www_root/;
}
```

站点的目录结构应该如下
```
/data/wwwsite/ttianjun.com/www/
/data/wwwsite/ttianjun.com/nginx/
```
这样访问`www.ttianjun.com`时root目录为/data/wwwsite/ttianjun.com/www/，
nginx.ttianjun.com时为/data/wwwsite/ttianjun.com/nginx/

## nginx完整配置
参考官网[https://www.nginx.com/resources/wiki/start/topics/examples/full/](https://www.nginx.com/resources/wiki/start/topics/examples/full/)
```conf
user       www www;  ## Default: nobody
worker_processes  5;  ## Default: 1
error_log  logs/error.log;
pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http {
  include    conf/mime.types;
  include    /etc/nginx/proxy.conf;
  include    /etc/nginx/fastcgi.conf;
  index    index.html index.htm index.php;

  default_type application/octet-stream;
  log_format   main '$remote_addr - $remote_user [$time_local]  $status '
    '"$request" $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log   logs/access.log  main;
  sendfile     on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  server { # php/fastcgi
    listen       80;
    server_name  domain1.com www.domain1.com;
    access_log   logs/domain1.access.log  main;
    root         html;

    location ~ \.php$ {
      fastcgi_pass   127.0.0.1:1025;
    }
  }

  server { # simple reverse-proxy
    listen       80;
    server_name  domain2.com www.domain2.com;
    access_log   logs/domain2.access.log  main;

    # serve static files
    location ~ ^/(images|javascript|js|css|flash|media|static)/  {
      root    /var/www/virtual/big.server.com/htdocs; # 直接从这个目录拿
      expires 30d;
    }

    # pass requests for dynamic content to rails/turbogears/zope, et al
    location / {
      proxy_pass      http://127.0.0.1:8080;
    }
  }

  upstream big_server_com {
    server 127.0.0.3:8000 weight=5;
    server 127.0.0.3:8001 weight=5;
    server 192.168.0.1:8000;
    server 192.168.0.1:8001;
  }

  server { # simple load balancing
    listen          80;
    server_name     big.server.com;
    access_log      logs/big.server.access.log main;

    location / {
      proxy_pass      http://big_server_com;
    }
  }
}

```