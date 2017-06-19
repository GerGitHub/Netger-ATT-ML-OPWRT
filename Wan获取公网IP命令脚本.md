# Wan获取公网IP命令脚本  

### 本命令主要针对部分大局域网用户, WAN口IP有时是公网有时是以10开头的大局域网IP     
          

命令如下:     

    #!/bin/sh
    IPF=`ifconfig |grep -A1 "ppp0" |grep "inet" |awk -F . '{print $1}'|awk -F \: '{print $2}'`
    if [ "$IPF" = "10" ]
      then
        echo "Bad IP Go Redial"
        killall pppd
        sleep 3
        pppd file /tmp/ppp/options.wan0 >/dev/null 2>&1 &
      else
        echo "Good IP"
        break
     fi
   
语法解释:   
第2行获取WAN口IP的第1段数字, 还有其它几种获取WAN口IP    

后段验WAN口IP头是否是10,如果是10就中断拨号进程, 等待3秒, 再执行拨号; 否则IP头不是10则当前WAN口是公网IP,并中断此循环;
