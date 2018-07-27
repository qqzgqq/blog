---
title: mysql常用命令
categories: mysql
tags: mysql
---

#### **一、查询logs表中所有信息**
```
mysql> select * from logs;                        
+----------+---------+---------------------+
| name     | caozuo  | uptime              |
+----------+---------+---------------------+
| guang3   | xinzeng | 2017-11-24 17:11:53 |
| guang3   | xiugai  | 2017-11-24 17:15:21 |
| guang3   | xiugai  | 2017-11-24 17:15:59 |
| guanggai | xiugai  | 2017-11-24 17:16:41 |
| guang    | delete  | 2017-11-24 17:21:05 |
| guang2   | delete  | 2017-11-24 17:21:05 |
+----------+---------+---------------------+
6 rows in set (0.00 sec)
```
<!-- more -->
#### **二、查询logs表中显示表中数**
```
mysql> select count(*) from logs;                    
+----------+
| count(*) |
+----------+
|        6 |
+----------+
1 row in set (0.00 sec)
```
#### **三、去重查询logs表中列name信息**
```
mysql> select distinct name from logs ;               
+----------+
| name     |
+----------+
| guang3   |
| guanggai |
| guang    |
| guang2   |
+----------+
4 rows in set (0.01 sec)
```
#### **四、查询logs表中列uptime大于2017-11-24 17:16:41的信息**
```
mysql> select * from logs where uptime > '2017-11-24 17:16:41';
+--------+--------+---------------------+
| name   | caozuo | uptime              |
+--------+--------+---------------------+
| guang  | delete | 2017-11-24 17:21:05 |
| guang2 | delete | 2017-11-24 17:21:05 |
+--------+--------+---------------------+
2 rows in set (0.00 sec)
```
#### **五、查询logs表中列uptime大于2017-11-24 17:16:41且小于2017-11-24 17:21:02的信息**
```
mysql> select * from logs where uptime between '2017-11-24 17:11:41' and '2017-11-24 17:21:02';
+----------+---------+---------------------+
| name     | caozuo  | uptime              |
+----------+---------+---------------------+
| guang3   | xinzeng | 2017-11-24 17:11:53 |
| guang3   | xiugai  | 2017-11-24 17:15:21 |
| guang3   | xiugai  | 2017-11-24 17:15:59 |
| guanggai | xiugai  | 2017-11-24 17:16:41 |
+----------+---------+---------------------+
4 rows in set (0.00 sec)
```
#### **六、查询logs表中列name为guanggai并且caozuo为guang3或xiugai的信息**
```
mysql> select * from logs where (caozuo='guang3' or caozuo='xiugai') and name='guanggai';
+----------+--------+---------------------+
| name     | caozuo | uptime              |
+----------+--------+---------------------+
| guanggai | xiugai | 2017-11-24 17:16:41 |
+----------+--------+---------------------+
1 row in set (0.00 sec)
```
#### **七、logs表中以列uptime倒叙显示信息**
```
mysql> select * from logs order by uptime desc;
+----------+---------+---------------------+
| name     | caozuo  | uptime              |
+----------+---------+---------------------+
| guang    | delete  | 2017-11-24 17:21:05 |
| guang2   | delete  | 2017-11-24 17:21:05 |
| guanggai | xiugai  | 2017-11-24 17:16:41 |
| guang3   | xiugai  | 2017-11-24 17:15:59 |
| guang3   | xiugai  | 2017-11-24 17:15:21 |
| guang3   | xinzeng | 2017-11-24 17:11:53 |
+----------+---------+---------------------+
6 rows in set (0.00 sec)
```
#### **八、查询logs表中name与uptime并以uptime正序显示**
```
mysql> select name,uptime from logs order by uptime asc;            
+----------+---------------------+
| name     | uptime              |
+----------+---------------------+
| guang3   | 2017-11-24 17:11:53 |
| guang3   | 2017-11-24 17:15:21 |
| guang3   | 2017-11-24 17:15:59 |
| guang2   | 2017-11-24 17:21:05 |
| guang    | 2017-11-24 17:21:05 |
+----------+---------------------+
6 rows in set (0.00 sec)
```
#### **九、向logs表中插入name=haha，caozuo=xinzeng的数据**
```
mysql> insert into logs(name,caozuo) values('haha','xinzeng');
Query OK, 1 row affected (0.01 sec)
```
#### **十、将logs表中列name=guang3更改为name=guang1且caozuo=xiugaii2**
```
mysql> update logs set name='guang1',caozuo='xiugaii2' where name='guang3';
Query OK, 3 rows affected (0.01 sec)
Rows matched: 3  Changed: 3  Warnings: 0
```
#### **十一、删除logs表中列name=guang3的数据**
```
mysql> delete from logs where name='guang3';
Query OK, 3 rows affected (0.00 sec)
```
#### **十二、删除logs表**
```
mysql> delete from logs;
Query OK, 4 rows affected (0.00 sec)
```
#### **十三、查询test表前2行**
```
mysql> select * from test limit 2;
+-----------+------+---------------------+
| name      | sex  | time                |
+-----------+------+---------------------+
| guanggai2 | V    | 2017-11-24 17:11:53 |
| xoapmig   | m    | 2017-12-04 15:58:24 |
+-----------+------+---------------------+
2 rows in set (0.00 sec)
```
#### **十四、查询test
表第二条至第三条数据**
```
mysql> select * from test limit 1,2;                              
+---------+------+---------------------+
| name    | sex  | time                |
+---------+------+---------------------+
| xoapmig | m    | 2017-12-04 15:58:24 |
| xoapmig | m    | 2017-10-11          |
+---------+------+---------------------+
2 rows in set (0.01 sec)
```
#### **十五、查询test表name中第4个字母为n的数据**
```
mysql> select * from test where name like "%%%n%”;                   
+-----------+------+---------------------+
| name      | sex  | time                |
+-----------+------+---------------------+
| guanggai2 | V    | 2017-11-24 17:11:53 |
+-----------+------+---------------------+
1 row in set (0.00 sec)
```
#### **十六、查询test表name中类似x*ap的数据**
```
mysql> select * from test where name like 'x_ap';
+------+------+-----------+
| name | sex  | time      |
+------+------+-----------+
| xoap | m    | 2017-9-11 |
+------+------+-----------+
1 row in set (0.00 sec)
```
#### **十七、查询pet表包含正好5个字符的名字**
```
mysql> SELECT * FROM pet WHERE name LIKE "_____";            
+-------+--------+---------+-----+------------+------+
| name  | owner  | species | sex | birth      | death|
+-------+--------+---------+-----+------------+------+
| Claws | Gwen   | cat     | m   | 1994-03-17 | NULL |
+------+---------+---------+-----+------------+------+
| Buffy | Harold | dog     | f   | 1989-05-13 | NULL |
+-------+--------+---------+-----+------------+------+
```
#### **十八、查询test表name中以ig结尾的数据**
```
mysql> select * from test where name regexp "ig$”;                
+---------+------+---------------------+
| name    | sex  | time                |
+---------+------+---------------------+
| xoapmig | m    | 2017-12-04 15:58:24 |
| Xoapmig | m    | 2017-10-11          |
+---------+------+---------------------+
2 rows in set (0.00 sec)
```
比如 SELECT * FROM ［user］ WHERE u_name LIKE ‘%三%’
将会把u_name为“张三”，“张猫三”、“三脚猫”，“唐三藏”等等有“三”的记录全找出来。