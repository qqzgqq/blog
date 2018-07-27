---
title: Nginx搭建简单的ftp服务
categories: nginx
tags: nginx
---
.
<!-- more -->
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    tcp_nopush    on; 
   keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
    root    /;
    autoindex    on;                                  #开启索引功能
    autoindex_exact_size    off;                # 关闭计算文件确切大小（单位bytes），只显示大概大小（单位kb、mb、gb）
    autoindex_localtime    on;                  # 显示本机时间而非 GMT 时间
    }
}
```