# FGC查看脚本
   当发生FGC时，运行此脚本
  
#  脚本
  #! /bin/bash
# 监控fgc产生时发生的情况，用于判断错误
pid=`ps -ef | grep jt808 | grep -v grep| awk '{print $2}'`
echo "pid=$pid"
flag=1
fgc1=0
while((flag==1))
do 
  sleep 1
  fgc2=`jstat -gcutil $pid | sed -n '2p' | awk '{print $10}'`

  echo "---------------start----------------">>gc.log
  echo "fgc2=$fgc2, fgc1=$fgc1" >> gc.log
  echo "fgc2=$fgc2, fgc1=$fgc1" 
  echo "-------------top--------------">>gc.log
  top -b| head -12 >> gc.log 
  # div=`expr $fgc2 - $fgc1` 
  #div=$(($fgc2-$fgc1))
  div=$(echo "$fgc2-$fgc1"|bc)
  echo "div=$div"
  if [ $(echo "$div > 0" | bc ) = 1 ]
    then 
      fgc1=$fgc2
      echo "------------------------------------出现了fgc---------------------------------" >> gc.log
      echo "------GC 情况-----"
      echo `jstat -gcutil $pid` >> gc.log
      for i in {1..3}
         do
        #查找消耗最高线程id
        # 加上-b不会出现乱码
        tid=`top -H -b -p $pid | head -10 | sed -n '8p' | awk '{print($1)}'`

        echo "tid=$tid"
        tid16=`printf "%x" $tid`
        echo "cpu 消耗最高线程 id 是${tid16}啊" >>gc.log
        echo "cpu 消耗最高线程 id 是${tid16}"

        date=`date`


        echo "-------------堆栈信息--${tid}--${tid16}---${date}------------" >> gc.log


        jstack $pid | grep -A 20 $tid16 >> gc.log
        jstack $pid > $pid$tid.jstack

        flag=0
        done
  fi 
done 
