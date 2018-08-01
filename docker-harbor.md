---
title: centos7使用vmware/harbor搭建带证书的私有dockerhub
categories: docker
tags: docker私有仓库
---

参考：https://github.com/vmware/harbor/blob/master/docs/installation_guide.md
<!-- more -->

|Resource|Capacity|Description|
|--------|--------|-----------|
|CPU |minimal 2 CPU |4 CPU is prefered| 
|Mem |minimal 4GB |8GB is prefered| 
|Disk|minimal 40GB| 160GB is prefered| 

<br>

|安装前置条件|
|----------|
|1.python2.7以上|
|2.docker 1.10以上|
|3.docker-compose 1.6以上|
|4.Openssl|

<br>

## **当前测试环境：**

> 使用的是vm的虚拟机创建的cenos7（ip：192.168.1.86）

`python安装参考：`https://www.jianshu.com/p/ada7ca038d7f<br>
`docker-ce安装：`
https://docs.docker.com/install/linux/docker-ce/centos/#os-requirements
```
卸载本机旧版本docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
yum安装docker-ce
 sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
$ sudo yum install docker-ce
$ sudo systemctl start docker
$ sudo docker -v
Docker version 1.12.6, build 3e8e77d/1.12.6
```
`docker-compose安装`
https://docs.docker.com/compose/install/#install-compose
```
curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x docker-compose
docker-compose -v
docker-compose version 1.21.2, build a133471
```

## **下载harbor离线安装包**
```
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.5.2.tgz
tar xvf harbor-offline-installer-v1.5.2.tgz
cd harbor
vim harbor.cfg（这是hostname=本地ip）
./install.sh
```
>使用虚拟机中centos7执行harbor安装脚本（install.sh）时遇到个坑，因为我使用的harbor的离线安装包，需要使用docker load harbor的docker 镜像文件（比较大），虚拟机中docker默认的Thin Pool有限制，倒入的时候会报如下错误
```
 devmapper: Thin Pool has 68 free data blocks which is less than minimum required 708 free data blocks. Create more free space in thin pool or use dm.min_free_space option to change behavior
```
`解决办法是：`
```
  [root@k8s1 harbor]#  yum remove docker -y        #先卸载了docker
  [root@k8s1 harbor]#  rm -rf /var/lib/docker      #删除了/var/lib/docker
  [root@k8s1 harbor]#  yum install docker -y       #yum安装docker
  [root@k8s1 harbor]#  systemctl enable docker     #设置docker服务开机启动
  [root@k8s1 harbor]# systemctl start docker       #启动docker
```
> 虚拟机中新增一块虚拟20G IDE虚拟盘，然后挂在到系统中,更改docker lib库路径，再导入docker镜像就不会报错了。
```
[root@k8s1 harbor]# mkdir /docker
[root@k8s1 harbor]# mkfs.ext4 /dev/sda             #fdisk -l可查看新盘符(此处用是sdb),格式化sda
[root@k8s1 harbor]# echo '/dev/sda  /docker ext4    defaults    0  0' >> /etc/fstab  #设置开机自动挂在
[root@k8s1 harbor]# mount -a
[root@k8s1 harbor]# df -hl
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   14G  3.2G  9.6G   25% /
devtmpfs                 478M     0  478M    0% /dev
tmpfs                    489M     0  489M    0% /dev/shm
tmpfs                    489M  6.8M  482M    2% /run
tmpfs                    489M     0  489M    0% /sys/fs/cgroup
/dev/sdb                  40G  3.2G   35G    9% /docker
/dev/sda1                190M  124M   53M   71% /boot
tmpfs                     98M     0   98M    0% /run/user/0
[root@k8s1 harbor]# systemctl stop docker           #停止docker服务器
[root@k8s1 harbor]# mv /var/lib/docker/* /docker/   #将docker库中所有文件移动至/docker中
[root@k8s1 harbor]# rm -rf /var/lib/docker          #删除原docker库
[root@k8s1 harbor]# ln -s  /docker /var/lib/docker  #将/docker目录中映射到/var/lib/docker中
[root@k8s1 harbor]# systemctl start docker          #启动docker
```
`公司内部使用docker私有hub是需要登录和加密的，证书加密参照`<br>
https://github.com/vmware/harbor/blob/master/docs/configure_https.md
## **创建证书**
```
[root@k8s1 cert]#mkdir -p /system/cert&&cd /system/cert
```
- 创建CA证书
```
[root@k8s1 cert]# openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout ca.key \
    -x509 -days 365 -out ca.crt                 #一路回车，注意出现CN时要输入本机ip或者域名
```
- 创建通讯crt与key
```
[root@k8s1 cert]# openssl req \
    -newkey rsa:4096 -nodes -sha256 -keyout 192.168.1.86.key \
    -out 192.168.1.86.csr                       #一路回车，注意出现CN时要输入本机ip或者域名
```
- 生成的证书注册主机
```
[root@k8s1 cert] #openssl x509 -req -days 365 -in 192.168.1.86.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out 192.168.1.86.crt
[root@k8s1 cert] echo subjectAltName = IP:192.168.1.86 > extfile.cnf
[root@k8s1 cert] openssl x509 -req -days 365 -in 192.168.1.86.csr -CA ca.crt -CAkey ca.key -CAcreateserial -extfile extfile.cnf -out 192.168.1.86.crt
```
> 证书生成完毕：我们所需要的证书主要有192.168.1.86.crt和192.168.1.86.key
## **harbor的配置与安装**

> 解压完harbor离线安装包后编辑harbor.cfg文件，需要修改的参数如下：
```
hostname=192.168.1.86                #改成本机ip或者域名
ui_url_protocol = https          #协议改为https
ssl_cert = /system/cert/192.168.1.86.crt
ssl_cert_key = /system/cert/192.168.1.86.key
```
> 保存退出后然后运行安装脚本
```
[root@k8s1 harbor]# ./install.sh                #等待安装完成。
[root@k8s1 harbor]# docker ps -a
CONTAINER ID        IMAGE                                  COMMAND                  CREATED             STATUS                       PORTS                                                              NAMES
e284c6fe107d        vmware/harbor-jobservice:v1.5.2        "/harbor/start.sh"       2 hours ago         Up About an hour                                                                                harbor-jobservice
901396649ea3        vmware/nginx-photon:v1.5.2             "nginx -g 'daemon ..."   2 hours ago         Up About an hour (healthy)   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp   nginx
b1fdd865e3a6        vmware/harbor-ui:v1.5.2                "/harbor/start.sh"       2 hours ago         Up About an hour (healthy)                                                                      harbor-ui
3582ebd4cc54        vmware/harbor-adminserver:v1.5.2       "/harbor/start.sh"       2 hours ago         Up About an hour (healthy)                                                                      harbor-adminserver
ce0684f4e056        vmware/registry-photon:v2.6.2-v1.5.2   "/entrypoint.sh se..."   2 hours ago         Up About an hour (healthy)   5000/tcp                                                           registry
7bc01fd5cd51        vmware/harbor-db:v1.5.2                "/usr/local/bin/do..."   2 hours ago         Up About an hour (healthy)   3306/tcp                                                           harbor-db
6bd7f638c608        vmware/redis-photon:v1.5.2             "docker-entrypoint..."   2 hours ago         Up About an hour             6379/tcp                                                           redis
1e7310db3ac2        vmware/harbor-log:v1.5.2               "/bin/sh -c /usr/l..."   2 hours ago         Up About an hour (healthy)   127.0.0.1:1514->10514/tcp                                          harbor-log
```
`如果registry启动失败，则查看/var/log/harbor/registry.log日志`<br>
> 一般删除harbor/common/config/registry/config.yml中swiff相关信息即可。然后再重启registry容器。

> 当容器都运行正常，使用ie访问192.168.1.86登录admin密码Harbor12345此用户是超级管理员，可添加用户，权限给予管理员权限。

`其他设备如要上传或者下载此私有docker hub镜像，需要向管理员索取192.168.1.86.crt证书并docker login 192.168.1.86方可上传下载。`

## **客户端安装证书**
- linux系统导入证书：

    - 1.mkdir /etc/docker/cert.d/192.168.1.86                                    #创建docker证书目录
    - 2.cp 192.168.1.86.crt /etc/docker/cert.d/192.168.1.86/ca.crt               #将证书改名ca.crt并放置在此路径下
    - 3.systemctl restart docker                                                 #加入ca证书后重启docker生效
    - 4.docker login 192.168.1.86                                                #输入用户名和密码后登录成功

- Mac系统导入证书：
    - 1.双机192.168.1.86.crt证书，系统默认钥匙串访问。
    - 2.进入钥匙串访问，找到该证书，选择使用信任。
    - 3.重启docker让证书生效
    - 4.docker login 192.168.1.86                                                #输入用户名和密码后登录成功


`如果登录时出现此类问题`<br>
```
Error response from daemon: Get https://192.168.1.86/v2/: x509: certificate signed by unknown authority
```
>那是因为证书问题，主要查看服务端registry容器是否报错，或者客户端docker添加完证书后未重启导致。

## **Docker 登录成功后验证上传镜像：**
- 1.将镜像改名192.168.1.86/library/*    （library）
```
docker tag quay.io/coreos/etcd:v2.3.8 192.168.1.86/library/etcd:V1
```
- 2.docker push 192.168.1.86/library/etcd:V1 