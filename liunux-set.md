---
title: liunux set 一般用法
categories: linux
tags: linux set
---

#### **看大神们写的shell脚本中开头一般会有set设置，今天查了下，总结如下四个用法。**
<!-- more -->
|语法|说明|
|----|----|
|set -e|当脚本中出现或警告会终止错误后面的脚本运行|
|set +o noblog|允许脚本中使用类似*通配符|
|set -o noblog|脚本中通配符不起作用|
|set -x|+号后先打印命令再执行|

## **一、脚本中不加set默认会识别*等通配符。**
```
[root@k8s1 zg]# cat new.sh
#!/bin/bash
ls ./*
echo ==================
ls ~/*

[root@k8s1 zg]# ./new.sh
./new.sh
==================
/root/anaconda-ks.cfg

/root/zg:
new.sh
```

## **二、set +o noblog允许脚本中使用类似*通配符**
```
[root@k8s1 zg]# cat new.sh
#!/bin/bash
set +o noglob
ls ./*
echo ==================
ls ~/*

[root@k8s1 zg]# ./new.sh
./new.sh
==================
/root/anaconda-ks.cfg

/root/zg:
new.sh
```

## **三、set -o noblog脚本中使用类似*通配符不生效**
```
[root@k8s1 zg]# cat new.sh
#!/bin/bash
set -o noglob
ls ./*
echo ==================
ls ~/*

[root@k8s1 zg]# ./new.sh
ls: 无法访问./*: 没有那个文件或目录
==================
ls: 无法访问/root/*: 没有那个文件或目录
```


## **四、set-e脚本中出现错误就终止后续命令运行**
```
[root@k8s1 zg]# cat new.sh
#!/bin/bash
set -e
ls ./*
echo ==================
aldjflajsdfl
ls ~/*

[root@k8s1 zg]# ./new.sh
./new.sh
==================
./new.sh:行5: aldjflajsdfl: 未找到命令
```

## **五、set -x脚本中命令先用+号打印，再执行。**
```
[root@k8s1 zg]# cat new.sh
#!/bin/bash
set -x
ls ./*
echo ==================
aldjflajsdfl
ls ~/*

[root@k8s1 zg]# ./new.sh
+ ls ./new.sh
./new.sh
+ echo ==================
==================
+ aldjflajsdfl
./new.sh:行5: aldjflajsdfl: 未找到命令
+ ls /root/anaconda-ks.cfg /root/zg
/root/anaconda-ks.cfg

/root/zg:
new.sh
```
