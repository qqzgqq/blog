Date: 2017-10-11 
Title: nginx反向代理配置
Intro: nginx反向代理
Tags: nginx
Status: public


<pre>
upstream txecs{
                server 127.0.0.1:8080;
              }
server{
        listen 80;
        server_name zabbix.txzs.org;
        access_log  /var/log/nginx/web1_access.log;

        location / {
        proxy_pass http://txecs;
        proxy_read_timeout 300;
        proxy_connect_timeout 300;
        proxy_redirect     off;

        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   Host              $http_host;
        proxy_set_header   X-Real-IP         $remote_addr;
                }
        }
</pre>