Date: 2017-10-18
Title: 计算当天时间差小脚本
Intro: 计算当天时间差
Tags: linux
Status: public


---
<pre>
#!/bin/bash
#author zengguang
#date 2017-10-13 15:59:32
#describe time

read -p "请输入开始时间：" sxh sfz
read -p "请输入结束时间：" exh efz
psxh=`echo $sxh |sed 's/[0-9]//g'`
psfz=`echo $sfz |sed 's/[0-9]//g'`
pexh=`echo $exh |sed 's/[0-9]//g'`
pefz=`echo $efz |sed 's/[0-9]//g'`
if [ -z $sxh ]
then
echo "您输入的开始时间为空"
exit
elif [ -z $sfz ]
then
echo "您输入的开始时间为空"
exit
elif [ -z $exh ]
then
echo "您输入的结束时间为空"
exit
elif [ -z $efz ]
then
echo "您输入的结束时间为空"
exit
elif [ -n "$psxh" ]
then
echo "请正确输入开始时间"
exit
elif [ -n "$psfz" ]
then
echo "请正确输入开始时间"
exit
elif [ -n "$pexh" ]
then
echo "请正确输入结束时间"
exit
elif [ -n "$pefz" ]
then
echo "请正确输入结束时间"
exit
else
start1=`date +%s -d "$sxh:$sfz"`
        echo "start1 is:"$start1
end=`date +%s -d "$exh:$efz"`
        echo "end is" $end
      result=$(($end-$start1))
        echo "result is :"$result
       result2=$(($result/60))
echo "时间间隔了$result2分钟"
xiaoshi=`awk -v c=$result2 BEGIN'{ printf "%.2f",c/60 }'`
echo "时间间隔了$xiaoshi小时"
fi
</pre>

---


<pre>
#[🔸🔸🔸 root@f4f325c92850 ~]# ./jisuan.sh
#请输入开始时间：1 23
#请输入结束时间：12 43
#start1 is:1508088180
#end is 1508128980
#result is :40800
#时间间隔了680分钟
#时间间隔了11.33小时
</pre>
