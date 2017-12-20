Date: 2017-11-15
Title: centos7下etcd静态配置集群搭建
Intro: etcd静态配置集群搭建
Tags: etcd
Status: public


## **一、环境介绍**

etcd主要功能是分布式的存储键值，优点不多说了，分布是集群，自动选举等等，自行百度，主要说下配置方法，折腾了几天，终于优点眉目了，记录下操作方法，本文参考了如下链接<br>
[https://www.cnblogs.com/breg/p/5728237.html](https://www.cnblogs.com/breg/p/5728237.html)<br>
[http://www.361way.com/etcd-cluster/5468.html](http://www.361way.com/etcd-cluster/5468.html)<br>
[https://n40lab.wordpress.com/2016/08/01/installing-coreos-etcd-server-on-centos-7/](https://n40lab.wordpress.com/2016/08/01/installing-coreos-etcd-server-on-centos-7/)<br>

创建了4个虚拟机，虚拟机中安装系统为centos7（虚拟机使用的docker）
<pre>
for((i=1,i<5,i++))
do
    docker run -d --name etcd$i --hostname=etcd$i --privileged=true centos7 /usr/sbin/init
    docker exec -it etcd$i bash -c “yum update -y&& yum install net-tools -y”
done
</pre>
```
hostname    ip
etcd1       192.168.0.5
etcd2       192.168.0.6
etcd3       192.168.0.7
etcd4       192.168.0.8
```
将上述ip表添加到/etc/hosts中

## **二、etcd安装与配置**
### **yum安装etcd**

`yum install etcd -y`

#### *验证etcd是否可用*
<pre>
[root@etcd1 /]# systemctl enable etcd&&systemctl start etcd
[root@etcd1 /]# etcdctl set hello world
world
[root@etcd1 /]# etcdctl get hello
world
</pre>
>这里的集群模式是指完全集群模式，当然也可以在单机上通过不同的端口，部署伪集群模式，只是那样做只适合测试环境，生产环境考虑到可用性的话需要将etcd实例分布到不同的主机上，这里集群搭建有三种方式，分布是静态配置，etcd发现，dns发现。默认配置运行etcd，监听本地的2379端口，用于与client端交互，监听2380用于etcd内部交互。etcd启动时，集群模式下会用到的参数如下：
>1. --name<br>
>2. etcd集群中的节点名，这里可以随意，可区分且不重复就行<br>
>3. --listen-peer-urls<br>
>4. 监听的用于节点之间通信的url，可监听多个，集群内部将通过这些url进行数据交互(如选举，数据同步等)<br>
>5. --initial-advertise-peer-urls<br>
>6. 建议用于节点之间通信的url，节点间将以该值进行通信。<br>
>7. --listen-client-urls<br>
>8. 监听的用于客户端通信的url,同样可以监听多个。<br>
>9. --advertise-client-urls<br>
>10. 建议使用的客户端通信url,该值用于etcd代理或etcd成员与etcd节点通信。<br>
>11. --initial-cluster-token etcd-cluster-1<br>
>12. 节点的token值，设置该值后集群将生成唯一id,并为每个节点也生成唯一id,当使用相同配置文件再启动一个集群时，只要该token值不一样，etcd集群就不会相互影响。<br>
>13. --initial-cluster<br>
>14. 也就是集群中所有的initial-advertise-peer-urls 的合集<br>
>15. --initial-cluster-state new<br>
>16. 新建集群的标志，初始化状态使用 new，建立之后改此值为 existing<br>

#### *yum安装etcd需要修改两个配置文件*
<pre>
<font color=#00FF00 size=4>[root@etcd1 etcd]# vim /lib/systemd/system/etcd.service</font>
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
# set GOMAXPROCS to number of processors
<font color=#DF0101 size=3>ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/bin/etcd \
--name=\"${ETCD_NAME}\" \
--data-dir=\"${ETCD_DATA_DIR}\" \
--listen-peer-urls=\"${ETCD_LISTEN_PEER_URLS}\" \
--advertise-client-urls=\"${ETCD_ADVERTISE_CLIENT_URLS}\" \
--initial-cluster-token=\"${ETCD_INITIAL_CLUSTER_TOKEN}\" \
--initial-cluster=\"${ETCD_INITIAL_CLUSTER}\"  \
--initial-cluster-state=\"${ETCD_INITIAL_CLUSTER_STATE}\" \
--listen-client-urls=\"${ETCD_LISTEN_CLIENT_URLS}\""</font>
Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
</pre>

<pre>
<font color=#00FF00 size=4>[root@etcd1 /]# vim /etc/etcd/etcd.conf</font>
ETCD_NAME=etcd01
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd1:2380"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd02=http://etcd2:2380,etcd03=http://etcd3:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd1:2379"
</pre>

#### *推荐二进制包安装etcd*
yum虽然也能用，还是二进制包的方法操作简单些。

下载稳定版本的etcd二进制包<br>
<pre>
mkdir /var/lib/etcd;mkdir /etc/etcd; groupadd -r etcd; useradd -r -g etcd -d /var/lib/etcd -s /sbin/nologin -c "etcd user" etcd;chown -R etcd:etcd /var/lib/etcd
</pre>
<pre>
ETCD_VERSION=`curl -s -L https://github.com/coreos/etcd/releases/latest | grep linux-amd64\.tar\.gz | grep href | cut -f 6 -d '/' | sort -u`; ETCD_DIR=/opt/etcd-$ETCD_VERSION; mkdir $ETCD_DIR;curl -L https://github.com/coreos/etcd/releases/download/$ETCD_VERSION/etcd-$ETCD_VERSION-linux-amd64.tar.gz | tar xz --strip-components=1 -C $ETCD_DIR; ln -sf $ETCD_DIR/etcd /usr/bin/etcd && ln -sf $ETCD_DIR/etcdctl /usr/bin/etcdctl; etcd --version
</pre>

四台虚拟机新建etcd服务启动脚本
<pre>
<font color=#00FF00 size=4 >[root@etcd1 /]# vim /lib/systemd/system/etcd.service</font>
[Unit]
Description=etcd service
After=network.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
User=etcd
ExecStart=/usr/bin/etcd
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
</pre>
`etcd静态模式集群配置（即知道需要哪几台虚拟机需要建立集群）etcd1，etcd2与etcd3建立集群后etcd4加入集群。`<br>
修改etcd1配置文件
<pre>
[root@etcd1 /]# cat /etc/etcd/etcd.conf
ETCD_NAME=etcd01
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd1:2380"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd02=http://etcd2:2380,etcd03=http://etcd3:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd1:2379"
</pre>
修改etcd2配置文件
<pre>
[root@etcd2 /]# cat /etc/etcd/etcd.conf
ETCD_NAME=etcd02
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd2:2380"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd02=http://etcd2:2380,etcd03=http://etcd3:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd2:2379"
</pre>
修改etcd3配置文件
<pre>
[root@etcd3 /]# cat /etc/etcd/etcd.conf
ETCD_NAME=etcd03
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd3:2380"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd02=http://etcd2:2380,etcd03=http://etcd3:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd3:2379"
</pre>

配置好3台虚拟机之后同时启动etcd服务<br>
`systemctl start etcd`<br>
正常情况可以查看到成员列表`etcdctl member list`
<pre>
[root@etcd1 /]# etcdctl member list
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
</pre>
## <font color=#DF0101 size=5>etcd配置过程中遇到的问题，添加节点，删除节点，备份数据</font><br>
### 本地连接报错
<pre>
[root@etcd2 /]# etcdctl member list
Error:  client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:2379: getsockopt: connection refused
; error #1: dial tcp 127.0.0.1:4001: getsockopt: connection refused

error #0: dial tcp 127.0.0.1:2379: getsockopt: connection refused
error #1: dial tcp 127.0.0.1:4001: getsockopt: connection refused
</pre>
>如果出现如上的错误，是因为ETCD_LISTEN_CLIENT_URLS参数没有配置http://127.0.0.1:2379而导致的，所以这里我使用了0.0.0.0代表了监控所有地址。
除此之外仍上述报错，当配置的很混乱的时候可以删掉/var/lib/etcd/中的default.etcd文件,前提是数据不重要哈。还要注意default.etcd文件夹权限改为etcd：etcd<br>
`chown -R etcd:etcd default.etcd `此步很重要，另外还需要细心检查/etc/etcd/etcd.conf和/lib/systemd/system/etcd.service是否有错误。改完之后再重启etcd服务（`systemctl start etcd`）就ok了。

### etcd集群中添加一节点etcd04<br>
在现有集群etcd三台虚拟机中任何一台操作添加新节点etcd04
<pre>
[root@etcd1 /]# etcdctl member list
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
[root@etcd1 /]# etcdctl member add <font color=#00FF00>etcd04 http://etcd4:2380</font>
Added member named etcd04 with ID ec8c708bb5b2fdc6 to cluster

<font color=#00FF00>ETCD_NAME="etcd04"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd03=http://etcd3:2380,etcd02=http://etcd2:2380,etcd04=http://etcd4:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"</font>   
<font color=#DF0101>注意此处三条必须要与etcd04配置文件（/etc/etcd/etcd.conf）相符合</font>
[root@etcd1 /]# etcdctl member list
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
<font color=#00FF00>ec8c708bb5b2fdc6[unstarted]: peerURLs=http://etcd4:2380</font>
</pre>
启动etcd服务之前修改etcd04配置文件
<pre>
[root@etcd4 /]# vim /etc/etcd/etcd.conf
ETCD_NAME=etcd04
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://etcd4:2380"
ETCD_INITIAL_CLUSTER="etcd01=http://etcd1:2380,etcd02=http://etcd2:2380,etcd03=http://etcd3:2380,etcd04=http://etcd4:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="http://etcd4:2379"
</pre>
`如果/var/lib/etcd/中存在default.etcd一定要删除，否侧etcd服务启动失败（因为集群中给etcd04分配的物理id与现有库中的id不相同）`<br>
<pre>
[root@etcd4 /]# rm -rf /var/lib/etcd/default.etcd/
[root@etcd4 /]# systemctl start etcd        
[root@etcd4 /]# etcdctl member list         #启动服务后再查询etcd04已加入集群中
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
<font color=#00FF00>ec8c708bb5b2fdc6: name=etcd04 peerURLs=http://etcd4:2380 clientURLs=http://etcd4:2379 isLeader=false</font>
</pre>




### etcd集群中删除etcd04节点<br>
现有集群节点是4个
<pre>
[root@etcd1 /]# etcdctl member list
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
<font color=#DF0101>b28ea7a9615a2c72</font>: name=etcd04 peerURLs=http://etcd4:2380 clientURLs=http://etcd4:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
</pre>
在除etcd04节点外的任意一台中操作删除etcd04节点
<pre>
[root@etcd1 /]# etcdctl member remove <font color=#DF0101>b28ea7a9615a2c72</font>
Removed member b28ea7a9615a2c72 from cluster
[root@etcd1 /]# etcdctl member list
3b05c2e4e4c104b4: name=etcd01 peerURLs=http://etcd1:2380 clientURLs=http://etcd1:2379 isLeader=false
bd388e7810915853: name=etcd03 peerURLs=http://etcd3:2380 clientURLs=http://etcd3:2379 isLeader=true
d282ac2ce600c1ce: name=etcd02 peerURLs=http://etcd2:2380 clientURLs=http://etcd2:2379 isLeader=false
</pre>

### etcd集群中备份数据<br>
etcd的数据应该算是很安全的，即便出现单点故障，数据也不会丢失，前提是节点不在同一台服务器中
etcd的数据备份比较。。。网上的方法<br>`etcdctl backup --data-dir /var/lib/etcd/default.etcd/ --backup-dir /home/etcd_backup`
备份完之后，发现没有db文件，好尴尬啊，所有我直接cp了，<br>`cp -r /var/lib/etcd/default.etcd/  /home/etcd_backup`<br>
`chown -R etcd:etcd default.etcd`嗯，就可以了。
