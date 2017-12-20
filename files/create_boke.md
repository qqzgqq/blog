Date: 2017-10-11 
Title: justwriting +dropbox+visual studio code搭建个人博客
Intro: 搭建个人博客
Tags: justwriting
Status: public

---
# **目录：**
## **1.各软件简介**

JustWriting是支持Markdown的极简的博客系统,JustWriting不须要数据库支持，仅仅须要把代码上传到web空间就可以好处不用多说了，简洁，方便，直观，Justwriting的创意来自Farbox,在这里向FarBox及其团队致敬。

Web代理使用的是nginx

visual studio code是一款非常好用多功能编辑器，在该编辑器中安装了Markdown+Math，用于编写*.md文档

dropbox主要用于上传*.md文档到vps服务器中。

## **2.准备工作**

Vps服务器（有独立ip，我用的搬瓦工的，操作系统用的是centos7）
域名（解析vps的ip地址）
参考[https://github.com/hjue/JustWriting/blob/master/README.zh.md](https://github.com/hjue/JustWriting/blob/master/README.zh.md)下载justwriting软件包

## **3.各软件安装**

### **安装nginx**
<pre>
mkdir -p /system/nginx
cd /system
wget http://tengine.taobao.org/download/tengine-2.0.0.tar.gz
tar zxvf tengine-2.0.0.tar.gz
cd tengine-2.0.0
./configure —prefix=/system/nginx        //如果报错，就yum install -y pcre pcre-devel openssl openssl-devel 
make
make install
</pre>

### **下载安装justwriting**
<pre>
cd ~
wget https://github.com/hjue/JustWriting/archive/master.zip
unzip master.zip
mv ~/JustWriting-master /system/nginx/html/
</pre>

### **配置nginx**

`cat /system/nginx/conf/nginx.conf`
<pre>
worker_processes 1;

events {
  worker_connections  1024;
}

http {
  include       mime.types;
  default_type  application/octet-stream;

  gzip on;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_proxied any;
  gzip_disable "msie6";
  gzip_types text/plain text/css application/x-javascript text/xml application/xml application/xml+rss text/javascript application/json;

  keepalive_timeout  65;

  include /system/nginx/conf/docker/*.conf;  //更改nginx配置文件目录
}
</pre>

`cat /system/nginx/conf/docker/just.conf`

<pre>
server {
        listen       80;
        server_name www.zengxiaoran.com;                //域名

        root /system/nginx/html/JustWriting-master/;    //根目录
        index index.html index.php;

        # set expiration of assets to MAX for caching
        location ~* \.(ico|css|js|gif|jpe?g|png)(\?[0-9]+)?$ {
                expires max;
                log_not_found off;
        }

       location ~* \.(md)$  {
          deny all;
        }

        location / {
                # Check if a file exists, or route it to index.php.
                try_files $uri $uri/ /index.php$uri?$args;
        }

        location ~* \.php {

                fastcgi_pass 127.0.0.1:9000;
                fastcgi_index index.php;
                fastcgi_split_path_info ^(.+\.php)(/?.*)$;
                fastcgi_param PATH_INFO $fastcgi_path_info;
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }


}
</pre>

`/system/nginx/sbin/nginx`     //启动nginx

### **安装php运行环境**

<pre>
yum -y groupinstall "Development Tools"    安装开发软件包

yum -y install httpd mysql mysql-server php php-mysql php-common php-mbstring php-gd php-odbc php-pear curl curl-devel net-snmp net-snmp-devel perl-DBI php-xml ntpdate php-bcmath        安装所需的依赖包

yum install -y php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-bcmath php-mhash php-fpm                   安装PHP组件，使PHP支持 MariaDB

</pre>

### **配置PHP**

`vim /etc/php.ini`
<pre>
date.timezone = Asia/Shanghai

max_execution_time = 300

max_input_time = 300

post_max_size = 32M

memory_limit = 128M
disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd,posix_getegid,posix_geteuid,posix_getgid,posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_isatty,posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname

#列出PHP可以禁用的函数，如果某些程序需要用到这个函数，可以删除，取消禁用。

expose_php = Off #禁止显示php版本的信息

short_open_tag = ON #支持php短标签

open_basedir = /system/nginx/html/JustWriting-master/
</pre>

`vim /etc/php-fpm.d/www.conf`
<pre>
listen = 127.0.0.1:9000
user = apache
group = apache
chdir = /system/nginx/html/JustWriting-master/
</pre>

`systemctl restart php-fpm.service`

`/system/nginx/sbin/nginx -s reload`        重新启动nginx，输入域名后即可看到博客主页

### **在Linuxserver上安装Dropbox**
32位系统

`cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86" | tar xzf -`

64位系统

`cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -`

启动dropbox

`~/.dropbox-dist/dropboxd  `

假设你是第一次在你的server上执行Dropbox，复制提示的链接在浏览器中打开，用你的Dropbx用户登录就可以完毕对server的授权。完毕授权后。Ctrol+C中断这个脚本。
建议为server专门(这里称作副帐号)创建一个Dropbox账号，把主账号的博客文章文件夹Share给这个用户。上述登录也使用副帐号登录。这样做尽管麻烦。但比較安全，而且不会把你主账号全部的Dropbox文件都同步到server上。
启动dropbox
<pre>
$ wget https://www.dropbox.com/download?dl=packages/dropbox.py
$ python dropbox.py start
</pre>
在你的用户根文件夹中会出现个Dropbox文件夹，用ln命令将你Dropbox的文章文件夹链接到Justwriting的posts文件夹。我的操作例如以下。供參考
<pre>
cd ~/Dropbox
ln -s /system/nginx/html/JustWriting-master/posts/ justwriting
</pre>

## *还有一点要注意上传到博客的md文档开头必须是如下格式才行*

<pre>
Date: 2017-10-11
Title: nginx反向代理配置
Intro: guang
Tags: nginx
Status: public
</pre>
正文

终于弄完了，有问题可以发送邮件qqzgqq@126.com
---
参考 [http://www.cnblogs.com/mengfanrong/p/4708783.html](http://www.cnblogs.com/mengfanrong/p/4708783.html)

[https://github.com/hjue/JustWriting](https://github.com/hjue/JustWriting)