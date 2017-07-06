# TOMATO服务项与用法

### 使用语法
   
    service <service> <action>
    where 


###  <action> 动作   
  
    start, stop, restart

###  <service> 服务项目  
   
     admin - 本地和远程的所有管理服务，如SSH，Telnet，HTTP，HTTPS
     cifs - 您的CIFS挂载的CIFS服务
     crond
     ctnf - 高级Conntrack和Netfilter，高级> Conntrack / netfilter Web界面中的所有内容
     ddns - 动态DNS服务
     dhcpc
     dhcpd - DHCP
     dns - DNS服务
     dnsmasq - Dnsmasq服务（具有集成DHCP服务器的轻量级缓存DNS代理）
     firewall - 任何防火墙相关
     jffs2 - JFFS2服务
     logging - 日志服务
     net
     ntpc - NTP服务
     qos - 服务质量服务
     restrict - 访问限制服务
     routing - 路由服务
     rstats - 重新启动带宽监控设置和服务
     rstatsnew - 与rstats-restart相同，但重置并启动新文件
     sched - Scheduler Services，如定时重启，重新连接和自定义脚本
     sshd
     telnetd
     upgrade
     upnp - 端口转发UPNP服务
     wan
    
  
### 例如:     
    service wan restart  #重新拨号  
     
+ 注意:以下两条命令作用相同, 重新所有服务,相当于重启   
   
      * -- Everything
      kill -HUP 1
     
