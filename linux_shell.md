---
title: linux命令积累
categories: linux
tags: shell
---




## 一、uniq     linux中去重命令
<!-- more -->
```shell
[#45#root@6b1092cdb680 nginx]# cat /proc/cpuinfo | grep "cpu cores"
cpu cores    : 4
cpu cores    : 4
cpu cores    : 4
cpu cores    : 4
[#46#root@6b1092cdb680 nginx]# cat /proc/cpuinfo | grep "cpu cores" | uniq
cpu cores    : 4
查看cpu型号
[#1#root@6b1092cdb680 ~]# cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
 4  Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
# 总核数 =物理CPU个数 X 每颗物理CPU的核数
# 总逻辑CPU数=物理CPU个数 X 每颗物理CPU的核数 X 超线程数

# 查看物理CPU个数
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)
cat /proc/cpuinfo| grep "cpu cores"| uniq


# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l
```
## 二、数组求和
```shell
localhost:aa zengguang$ cat 11.test | awk '{ aa +=$1 } END { print aa }'
55
localhost:aa zengguang$ cat 11.test
1
2
3
4
5
6
7
8
9
10
```
## 三、创建多个文件夹
```shell
mkdir -m 775 dir1

mkdir  -p /home/dir1/dir2 当dir1不存在是，不加-p参数会创建失败，加上后创建成功

mkdir -p /home/{test1/test2,test3/test4}  会同时创建/home/test1/test2、/home/test3/test4 目录
```
## 四、ifconfig commond not find
```shell
yum install net-tools
```
## 五、查看文件前几行
```shell
sed -n ‘1,5p’ 文件
```
## 六、curl查看网站http_code值，判断网站是否运行正常，该值如果等于000或者大于500则运行异常
```shell
curl -m 5 -s -w %{http_code} www.baidu.com -o /dev/null
```
## 七、红色高亮显示
```shell
echo -e '\E[31m' "hello"
echo -e '\E[32m' "hello”  绿色
```
## 八、shell脚本中增加set -e后脚本运行出错后就不会继续向下执行了。
## 九、在文件中某位置插入数据
```shell
[#46#tongxin@dtest ~]$ cat test
2222$
3333
[#49#tongxin@dtest ~]$ (echo '0a';echo 'start';echo '.';echo 'wq')|ed -s test
[#50#tongxin@dtest ~]$ cat test
start
2222$
3333
[#51#tongxin@dtest ~]$ (echo '1a';echo 'start1';echo '.';echo 'wq')|ed -s test
[#52#tongxin@dtest ~]$ cat test
start
start1
2222$
3333
```
## 十、删除文本中某行
```shell
sed -i 1,10d file
```
## 十一、端口抓包命令
```shell
sudo tcpdump -i any -n tcp port 10050
```
## 十二、定时删除7天前文件
```shell
find ./ -maxdepth 1  -ctime +7 -type d -exec rm -rf {} \;

crontab -e
* 0 * * * /home/dicom/zhouwei/rm.sh >> /dev/null&
```
## 十三、linux let字符转数字运算
```shell
a=10
b=5
let c=$a-$b
echo $c
5
```
## 十四、linux重定向方式调用
```shell
root@e190832af5c8:~# cat << EOF
> nihao
> 1111
> 2222
> 3333
> EOF
nihao
1111
2222
3333
root@e190832af5c8:~# cat << EOF > ceshi.sh
> echo "hello"
> echo "world"
> EOF
root@e190832af5c8:~# chmod 755 ceshi.sh
root@e190832af5c8:~# ./ceshi.sh
hello
world
root@e190832af5c8:~# cat << test > 1.sh
cp ./ceshi.sh ./ceshi2.sh
test
root@e190832af5c8:~# chmod 755 1.sh
root@e190832af5c8:~# ./1.sh
root@e190832af5c8:~# ls
1.sh  ceshi.sh    ceshi2.sh  sql.sh
root@e190832af5c8:~# cat sql.sh
#!/bin/bash
mysql -uroot -p1q2w3e <<EOF 2> /dev/null
create database ceshi2;
show databases;
EOF
```
## 十五、linux随机32位密码生成器
```shell
 < /dev/urandom tr -dc '12345!@#$%qwertQWERTasdfgASDFGzxcvbZXCVB'| head -c 32 ;echo
```
## 十六、linux grep 去注释和空行
 ```shell
 cat /etc/openvpn/server.conf | grep -v "#"|grep -v ^$
 ```
## 十七、linux 命令结尾初加2>* 即把错误吸入到*
```shell
[#68#zengguang@localhost ~]$ ls asdf 2>cuowu.log
[#69#zengguang@localhost ~]$ ls
Applications    Desktop         Documents       Downloads       Library         Movies          Music           Pictures        Public          PycharmProjects aa              cuowu.log       pi.ovpn         server.conf     test
[#70#zengguang@localhost ~]$ cat cuowu.log
ls: asdf: No such file or directory
[#79#zengguang@localhost ~]$ ls test1 >123 2>&1
[#80#zengguang@localhost ~]$ ls
123             Applications    Desktop         Documents       Downloads       Library         Movies          Music           Pictures        Public          PycharmProjects aa              pi.ovpn         test
[#81#zengguang@localhost ~]$ cat 123
ls: test1: No such file or directory
```
## 十八、shell 将字符串加密
```shell
[#35#root@master html]# echo "1q2w3e"|base64 -i
MXEydzNlCg==
```
## 十九、网页显示字体颜色
```html
<span style="color: gray">just do it</span></br>
<!DOCTYPE html>
<html>
<head>
<title>TongXin Docker Hub</title>
<style>
    body {
        width: 40em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>TongXin Docker Hub List</h1>
```
## 二十、linux中判断输入的是否为数字
```shell
echo "111asd" | sed ’s/[0-9]//g’    为空则是数字，不为空则不是数字
```
## 二十一、linux中计算两数字相除保留两个小数
```shell
awk  BEGIN’{ printf  "%.2f", 9/2 }’
 xiaoshi=`awk -v c=$result2 BEGIN'{ printf "%.2f",c/60 }'`
 ```
## 二十二、更改linux用户密码
```shell
echo ”guang：123”|chpasswd
```
## 二十三、linux中shell逐行读取
```shell
[🔸🔸🔸 root@f4f325c92850 ~]# cat 2.sh
#!/bin/bash
cat ./1.sh|while read line
do
    echo $line
    echo "----------"
done
[🔸🔸🔸 root@f4f325c92850 ~]# ./2.sh
#!/bin/bash
----------
./2.sh
----------
echo "1 diao 2 "
```
## 二十四、linux应用程序后台运行加错误写入文本
```shell
nohup /bin/dcmp > out.file 2>&1 &
```