Date: 2017-12-5
Title: n个奇数之和mysql存储过程
Intro: 5之后的第n个奇数之和mysql存储过程
Tags: mysql
Status: public
                                                          
<pre>
mysql> drop procedure  if exists test;        如果test存储过程存在就删除
mysql> delimiter ;;         设置输入已;;结尾
mysql> create procedure test (IN n int,OUT sum int)    #新建存储过程test并声明n具有IN，sum为OUT
    -> begin                                           #开始，固定格式
    -> declare b int;                                  #声明b，c为整型
    -> declare c int;                                  
    -> set b=5;                                        #设置b=5，c=0,sum=0
    -> set c=0;
    -> set sum=0;
    -> while c <= n do                                  #从0开始一直到n循环
    ->     if b%2 !=0 then  set sum=sum+b;              #奇数则计算sum之和
    ->     end if;                                      
    ->     set b=b+1;                                   #b，c依次递增
    ->     set c=c+1;
    -> end while;                                       #结束循环
    -> end;;
Query OK, 0 rows affected (0.00 sec)

mysql> call test(1,@s);;                                #执行存储过程
Query OK, 0 rows affected (0.00 sec)

mysql> select @s;;                                       #显示sum值
+------+
| @s   |
+------+
|    5 |
+------+
1 row in set (0.00 sec)
</pre>