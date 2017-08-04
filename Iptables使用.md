# Iptables使用    

## 注意   
iptables规则允许规则需放在阻止规则上面，因为linux执行规则时是从上往下，如果让系统先执行阻止规则，再去执行允许规则，允许规则会失效。    
iptables的规则匹配顺序上从上到下的，也就是说如果上下两条规则有冲突时，将会以上面的规则为准。

### 防暴力破解的命令一    

    iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --set
    iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
     
第一句是说，对于外来数据，如果是 TCP 协议，目标端口号是 22，网络接口是 eth0，状态是新连接，那么把它加到最近列表中。        
第二句是说，对于这样的连接，如果在最近列表中，并且在 60 秒内达到或者超过四次，那么丢弃该数据。其中的-m是模块的意思。         
也就是说，如果有人从一个 IP 一分钟内连接尝试四次 ssh 登录的话，那么它就会被加入黑名单，后续连接将会被丢弃。不过不知道多久以后那个 IP 才能重新连接上。      


需求1：只允许IP（1.1.1.1）访问linux服务器，阻止其它IP访问    
iptables -A INPUT -s 1.1.1.1 -j ACCEPT        
#允许指定IP访问    
iptables -A INPUT -s 0.0.0.0/0 -j drop             
#阻止任何IP访问    
service iptables save                                          
#保存iptables规则     
 
需求2：只允许IP（1.1.1.1）访问服务器的22端口，阻止其它IP访问服务器的22端口    
iptables -I INPUT -s 1.1.1.1 -p tcp --dport 22 -j ACCEPT         
#允许指定IP访问本服务器的22端口     
iptables -I INPUT -p tcp --dport 22 -j drop                                  
#阻止任何IP访问本服务器的22端口    
service iptables save                                                                     
#保存iptables规则   
 
需求3：禁ping
iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j drop     
#允许本服务器ping通别人，阻止别人ping通本服务器    
service iptables save     
#保存iptables防火墙设置    

### Iptables使用    
封单个IP的命令：iptables -I INPUT -s 124.115.0.199 -j DROP   
封IP段的命令：iptables -I INPUT -s 124.115.0.0/16 -j DROP    
封整个段的命令：iptables -I INPUT -s 194.42.0.0/8 -j DROP    
封几个段的命令：iptables -I INPUT -s 61.37.80.0/24 -j DROP    
只封80端口：iptables -I INPUT -p tcp –dport 80 -s 124.115.0.0/24 -j DROP    
解封：iptables -F    
清空：iptables -D INPUT 数字    
    
列出 INPUT链 所有的规则：iptables -L INPUT --line-numbers    
删除某条规则，其中5代表序号（序号可用上面的命令查看）：iptables -D INPUT 5    
开放指定的端口：iptables -A INPUT -p tcp --dport 80 -j ACCEPT    
禁止指定的端口：iptables -A INPUT -p tcp --dport 80 -j DROP    
拒绝所有的端口：iptables -A INPUT -j DROP    
    
以上都是针对INPUT链的操作，即是外面来访问本机的方向，配置完之后 需要保存，否则iptables 重启之后以上设置就失效    
service iptables save    
iptables 对应的配置文件  /etc/sysconfig/iptables    
     
### iptables使用二          
封单个IP的命令是：      
    iptables -I INPUT -s 211.1.0.0 -j DROP   
封IP段的命令是：
    iptables -I INPUT -s 211.1.0.0/16 -j DROP      
    iptables -I INPUT -s 211.2.0.0/16 -j DROP      
    iptables -I INPUT -s 211.3.0.0/16 -j DROP      
封整个段的命令是：   
    iptables -I INPUT -s 211.0.0.0/8 -j DROP       
封几个段的命令是：  
    iptables -I INPUT -s 61.37.80.0/24 -j DROP      
    iptables -I INPUT -s 61.37.81.0/24 -j DROP 
     
【解封】  
    iptables -D INPUT -s IP地址 -j REJECT      
如果发现input连接 命令不起作用，则可以 路由连接参数 使用下面命令      
iptables -A FORWARD -s 1.202.0.0/16 -j DROP            
在unix中IP子网掩码可用16,24,32等数字来表示,意思是:16表示子网掩码的前16位是全1,24、32以此类推。      
iptables -A FORWARD -s 61.172.0.0/16 -i 网卡名称 -j DROP      
--------有人说这种方法比较好用，这句比你防火墙管用--------      
那句只能禁止一个ip路由  #route add 61.172.0.0/16 reject      
这句可以封整段       #route add -net  61.172.0.0 netmask 255.255.0.0 reject          
下面看些实际例子，设计封第几个IP段的问题：      
------如果要封的内容是 061.037.080.000->061.037.081.255 -----      
    iptables -I INPUT -s 61.37.80.0/24 -j DROP      
    iptables -I INPUT -s 61.37.81.0/24 -j DROP            
-----用什么命令可以让iptables 封了 211.1.0.0 到 211.10.0.0 IP段?-----------      
    iptables -I INPUT -s 211.1.0.0/16 -j DROP      
    iptables -I INPUT -s 211.2.0.0/16 -j DROP      
    iptables -I INPUT -s 211.3.0.0/16 -j DROP      
------如果要封的内容是整段的 比如 211.0.0.0 - 211.255.255.255 -------------      
    iptables -I INPUT -s 211.0.0.0/8 -j DROP      
  
    
#  参考文章   

[iptables禁IP与解封IP常用命令](https://yusi123.com/3092.html)
