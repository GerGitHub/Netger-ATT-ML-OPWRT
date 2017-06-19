# Wan获取公网IP命令脚本  

### 本命令主要针对部分大局域网用户, WAN口IP有时是公网有时是以10开头的大局域网IP,以下有2种方法    
方案1可以定时或长时段检查,可以减少路由器运行负荷,但50次不确定获取得公网IP, 还有运营商的不定时IP更换可能打乱计划   
方案2无论是否获取公网IP每分钟都要检查,以保证获取公网IP,会增加路由器运行负荷,但也不大   


### 1.每天定时执行1次脚本,会连续检测50次
   
    #!/bin/sh
    killall pppd
    sleep 1
    pppd file /tmp/ppp/options.wan0 >/dev/null 2>&1 &
    for i in `seq 50`
    do
      IPF=`ifconfig |grep -A1 "ppp0" |grep "inet" |awk -F . '{print $1}'|awk -F \: '{print $2}'`
      if [ "$IPF" = "10" ]
      then
        echo "Bad IP Go Redial"
        killall pppd
        sleep 1
        pppd file /tmp/ppp/options.wan0 >/dev/null 2>&1 &
        sleep 30
      else
        echo "Good IP"
        break
      fi
    done
   
语法解释:  
先断网,重拨1次
for循环do与done之间的命令执行50次, 50次之后不论是否获得公网IP脚本都将中断;   
cru a GDDIP "* 5 * * * /jffs/scripts/GDDIP.sh"  , 每天5点钟执行该命令



### 2.每分钟去执行检查1次WAN口IP地址
         
命令如下:     

    #!/bin/sh
    IPF=`ifconfig |grep -A1 "ppp0" |grep "inet" |awk -F . '{print $1}'|awk -F \: '{print $2}'`
    if [ "$IPF" = "10" ]
      then
        echo "Bad IP Go Redial"
        killall pppd
        sleep 1
        pppd file /tmp/ppp/options.wan0 >/dev/null 2>&1 &
      else
        echo "Good IP"
        break
     fi
   
语法解释:   
第2行获取WAN口IP的第1段数字,  
后段验WAN口IP头是否是10,如果是10就中断拨号进程, (延迟1秒), 再执行拨号; 否则IP头不是10则当前WAN口是公网IP,并中断此循环;

+ 新建命令文件 GDDIP.sh 保存至 /jffs/scripts/ 目录
+ 在 /jffs/scripts/init-start 文件内写入计划命令   
  cru a GDDIP "*/1 * * * * /jffs/scripts/GDDIP.sh"  , 每1分钟执行1次
  
  
  
###  其它几种获取WAN口IP命令方法   
