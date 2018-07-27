---
title: docker部署etcd安全集群（带证书验证）
categories: docker
tags: etcd
---

## 一、环境介绍
|软件|版本|
|----|---|
|etcd|3.11|
|docker|1.12|    
|centos|7.4|

使用了三台虚拟机来实现docker部署etcd证书安全集群，虚拟机详情如下：

|计算机名|ip地址|
|-------|-----|
|GO_A|10.20.14.227|
|GO_B|10.20.14.228|
|GO_C|10.20.14.229|
<!-- more -->
需要使用etcd API3版本，便build了一个Dockerfile

## 二、准备工作
三台虚拟中操作安装docker,下载最新版etcd，将etcd 与etcdctl放在~/etcd文件夹中

```bash
[usr@GO_A]$ yum update
[usr@GO_A]$ yum install docker -y
[usr@GO_A]$ mkdir ~/etcd
[usr@GO_A]$ cd ~/etcd
[usr@GO_A]$ cat ~/etcd/Dockerfile
FROM        microbox/scratch
ADD         etcd        /bin/etcd
ADD         etcdctl        /bin/etcdctl
EXPOSE 2379 2380
ENTRYPOINT ["/bin/etcd"]
[usr@GO_A]$ sudo docker build -t docker_etcd .
```

## 三、证书生成
证书生成工具使用cfssl，下载好cfssl后放在/usr/bin/目录中，只在一台虚拟机中操作方法如下：

```bash
[usr@GO_A]$ mkdir ~/etcd_ssl
[usr@GO_A]$ cd ~/etcd_ssl
[usr@GO_A]$ cat etcd-root-ca-csr.json
{
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "O": "etcd",
      "OU": "etcd Security",
      "L": "Beijing",
      "ST": "Beijing",
      "C": "CN"
    }
  ],
  "CN": "etcd-root-ca"
}
[usr@GO_A]$ cat etcd-gencert.json
{
  "signing": {
    "default": {
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ],
        "expiry": "87600h"
    }
  }
}
[usr@GO_A]$ cat etcd-csr.json
{
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "O": "etcd",
      "OU": "etcd Security",
      "L": "Beijing",
      "ST": "Beijing",
      "C": "CN"
    }
  ],
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "localhost",
    "10.20.14.227",
    "10.20.14.228",
    "10.20.14.229"
  ]
}
[usr@GO_A]$ cat genCerts.sh
#!/bin/bash

cfssl gencert --initca=true etcd-root-ca-csr.json | cfssljson --bare etcd-root-ca
cfssl gencert --ca etcd-root-ca.pem --ca-key etcd-root-ca-key.pem --config etcd-gencert.json etcd-csr.json | cfssljson --bare etcd

[usr@GO_A]$ sudo ./genCerts.sh
2018/01/04 12:00:34 [INFO] generating a new CA key and certificate from CSR
2018/01/04 12:00:34 [INFO] generate received request
2018/01/04 12:00:34 [INFO] received CSR
2018/01/04 12:00:34 [INFO] generating key: rsa-4096
2018/01/04 12:00:37 [INFO] encoded CSR
2018/01/04 12:00:37 [INFO] signed certificate with serial number 531309111663139014479223287410519418718966943378
2018/01/04 12:00:37 [INFO] generate received request
2018/01/04 12:00:37 [INFO] received CSR
2018/01/04 12:00:37 [INFO] generating key: rsa-2048
2018/01/04 12:00:37 [INFO] encoded CSR
2018/01/04 12:00:37 [INFO] signed certificate with serial number 211638760810262365547576398622715505439166004704
2018/01/04 12:00:37 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").

[usr@GO_A]$ mkdir -p /system/{etcd_ssl/{etcd,key},etcd_backup}
[usr@GO_A]$ cp etcd.pem etcd-root-ca.pem etcd-key.pem /system/etcd_ssl
[usr@GO_A]$ cp etcd.pem /system/etcd_ssl/etcd/
[usr@GO_A]$ cp etcd-key.pem /system/etcd_ssl/key/
```
etcd-root-ca.pem 即CA证书
etcd.pem etcd-key.pem是以CA证书为基准生成的证书，用于client与server端，集群之间通讯验证
将上面三个证书放在在三台虚拟机的/system/etcd_ssl中

## 四、集群搭建
GO_A创建docker_etcd运行脚本
```bash
[usr@GO_A]$ cat docker_etcd_start.sh
#!/bin/bash
docker run -d -p 2380:2380 -p 2379:2379 \
 -v /system/etcd_ssl:/etcd_ssl \
 -v /system/etcd_backup:/etcd_date \
 --name etcd docker_etcd \
 -name etcd0 \
 -data-dir=/etcd_date/etcd0.etcd \
 -advertise-client-urls https://10.20.14.227:2379 \
 -listen-client-urls https://0.0.0.0:2379 \
 -initial-advertise-peer-urls https://10.20.14.227:2380 \
 -listen-peer-urls https://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd0=https://10.20.14.227:2380 \
 -initial-cluster-state new \
 -cert-file=/etcd_ssl/etcd/etcd.pem \
 -key-file=/etcd_ssl/key/etcd-key.pem \
 -trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -client-cert-auth=true \
 -auto-tls=true \
 -peer-cert-file=/etcd_ssl/etcd/etcd.pem \
 -peer-key-file=/etcd_ssl/key/etcd-key.pem \
 -peer-trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -peer-client-cert-auth=true \
 -auto-tls=true
```
GO_B创建docker_etcd运行脚本
```bash
[usr@GO_B]$ cat docker_etcd_start.sh
docker run -d -p 2380:2380 -p 2379:2379 \
 -v /system/etcd_ssl:/etcd_ssl \
 -v /system/etcd_backup:/etcd_date \
 --name etcd docker_etcd \
 -name etcd1 \
 -data-dir=/etcd_date/etcd1.etcd \
 -advertise-client-urls https://10.20.14.228:2379 \
 -listen-client-urls https://0.0.0.0:2379 \
 -initial-advertise-peer-urls https://10.20.14.228:2380 \
 -listen-peer-urls https://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd0=https://10.20.14.227:2380,etcd1=https://10.20.14.228:2380 \
 -initial-cluster-state existing \
 -cert-file=/etcd_ssl/etcd/etcd.pem \
 -key-file=/etcd_ssl/key/etcd-key.pem \
 -trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -client-cert-auth=true \
 -auto-tls=true \
 -peer-cert-file=/etcd_ssl/etcd/etcd.pem \
 -peer-key-file=/etcd_ssl/key/etcd-key.pem \
 -peer-trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -peer-client-cert-auth=true \
 -auto-tls=true
```

GO_C创建docker_etcd运行脚本
```bash
[usr@GO_C]$ cat docker_etcd_start.sh
#!/bin/bash
docker run -d -p 2380:2380 -p 2379:2379 \
 -v /system/etcd_ssl:/etcd_ssl \
 -v /system/etcd_backup:/etcd_date \
 --name etcd docker_etcd \
 -name etcd2 \
 -data-dir=/etcd_date/etcd2.etcd \
 -advertise-client-urls https://10.20.14.229:2379 \
 -listen-client-urls https://0.0.0.0:2379 \
 -initial-advertise-peer-urls https://10.20.14.229:2380 \
 -listen-peer-urls https://0.0.0.0:2380 \
 -initial-cluster-token etcd-cluster-1 \
 -initial-cluster etcd0=https://10.20.14.227:2380,etcd1=https://10.20.14.228:2380,etcd2=https://10.20.14.229:2380 \
 -initial-cluster-state existing \
 -cert-file=/etcd_ssl/etcd/etcd.pem \
 -key-file=/etcd_ssl/key/etcd-key.pem \
 -trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -client-cert-auth=true \
 -auto-tls=true \
 -peer-cert-file=/etcd_ssl/etcd/etcd.pem \
 -peer-key-file=/etcd_ssl/key/etcd-key.pem \
 -peer-trusted-ca-file=/etcd_ssl/etcd-root-ca.pem \
 -peer-client-cert-auth=true \
 -auto-tls=true
```
```
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
```
<font color=#adf1234d>先启动GO_A中docker_etcd_start.sh脚本后查看docker是否正常启动,然后注册GO_B,GO_C为其集群成员后再启动后面两台虚拟机启动脚本。</font>

### 先启动GO_A中docker_etcd_start.sh脚本后查看docker是否正常启动
```bash
[usr@GO_A]$ sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                                        NAMES
7f63819903d2        etcd_tongxin        "/bin/etcd -name e..."   2 days ago          Up 2 days                0.0.0.0:2379-2380->2379-2380/tcp, 4001/tcp   etcd
```
### 注册GO_B,GO_C为其集群成员
```bash
[usr@GO_A]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.227:2379/v2/members -XPOST -H "Content-Type: application/json" -d '{"peerURLs":["https://10.20.14.228:2380"]}'

[usr@GO_A]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.227:2379/v2/members -XPOST -H "Content-Type: application/json" -d '{"peerURLs":["https://10.20.14.229:2380"]}'

```
### 启动GO_B,GO_C的etcd启动脚本
```bash
[usr@GO_B]$ sudo ./docker_etcd_start.sh

[usr@GO_C]$ sudo ./docker_etcd_start.sh
```
## 五、测试验证
### 在GO_B和GO_C中查看是否集群成员是否存在
```bash
[usr@GO_B]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.227:2379/v2/members
{"members":[{"id":"3355db0d79cca526","name":"etcd1","peerURLs":["https://10.20.14.228:2380"],"clientURLs":["https://10.20.14.228:2379"]},{"id":"3b59591f890d9d8b","name":"etcd0","peerURLs":["https://10.20.14.227:2380"],"clientURLs":["https://10.20.14.227:2379"]},{"id":"62b8dae900b4cbb1","name":"etcd2","peerURLs":["https://10.20.14.229:2380"],"clientURLs":["https://10.20.14.229:2379"]}]}
```
```bash
[usr@GO_C]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.227:2379/v2/members
{"members":[{"id":"3355db0d79cca526","name":"etcd1","peerURLs":["https://10.20.14.228:2380"],"clientURLs":["https://10.20.14.228:2379"]},{"id":"3b59591f890d9d8b","name":"etcd0","peerURLs":["https://10.20.14.227:2380"],"clientURLs":["https://10.20.14.227:2379"]},{"id":"62b8dae900b4cbb1","name":"etcd2","peerURLs":["https://10.20.14.229:2380"],"clientURLs":["https://10.20.14.229:2379"]}]}
```
### 在GO_A中创建一个key，GO_B查看是否存在
```bash
[usr@GO_B]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.227:2379/v2/message -XPUT -d value="Hello world"
{
    "action": "set",
    "node": {
        "createdIndex": 2,
        "key": "/message",
        "modifiedIndex": 2,
        "value": "Hello world"
    }
}
usr@GO_B]$ curl --cacert /system/etcd_ssl/etcd-root-ca.pem --cert /system/etcd_ssl/etcd/etcd.pem --key /system/etcd_ssl/key/etcd-key.pem https://10.20.14.228:2379/v2/keys/message
{"action":"get","node":{"key":"/message","value":"Hello world","modifiedIndex":182,"createdIndex":182}}
```
### 停止或删除一个节点后再运行启动脚本会自动加入集群中，前提是不要删掉/system/etcd_backup文件夹