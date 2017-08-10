# Iptables使用    

## 注意 
iptables允许规则需放在阻止规则上面，因为linux执行规则时是从上往下，如果让系统先执行阻止规则，再去执行允许规则，允许规则会失效。    
iptables的规则匹配顺序上从上到下的，也就是说如果上下两条规则有冲突时，将会以上面的规则为准。   
tomato WEB工具/系统命令 输入以下命令 重启防火墙 使新添加规则生效    

    service firewall restart

## 限制规定时间内连接请求次数  方法1 
tomato 系统  Administration/scripts/firewall 系统管理/脚本/防火墙 添加以下命令   

    iptables -I INPUT -i ppp0 -p tcp -m multiport --dport 20,21,22,25,135,137:139,161,443,445,1080,3389 -m state --state NEW -m recent --name hack --set -j ACCEPT    
    iptables -I INPUT -i ppp0 -p tcp -m multiport --dport 20,21,22,25,135,137:139,161,443,445,1080,3389 -m state --state NEW -m recent --name hack --hitcount 3 --seconds 1800 --update -j DROP    
    
规则添加顺序与执行规则排列顺序 ---相反 即A,B,C会按C,B,A依序执行   
第2条 更新查询名为hack的记录表, 对在30分钟内对规定，端口, 已发送过3次新建连接请求的IP执行丢弃, 第4次即开始丢弃    
第1条 对规定端口发送新建连接请求的IP set记录到名为hack的列表中, 然后通过本规则执行下一规则   
当30分钟内被丢弃ip没有新的信息记录到hack列表，再发送新建连接请求时会被充许, 即P被封30分钟
    
## 限制规定时间内连接请求次数  方法2  与方法1看似相同
tomato 系统  Administration/scripts/firewall 系统管理/脚本/防火墙 添加以下命令   

    iptables -I INPUT -i ppp0 -p tcp -m multiport --dport 20,21,22,25,135,137:139,161,443,445,1080,3389 -m state --state NEW -m recent --name hack --hitcount 3 --seconds 1800 --update -j DROP    
    iptables -I INPUT -i ppp0 -p tcp -m multiport --dport 20,21,22,25,135,137:139,161,443,445,1080,3389 -m state --state NEW -m recent --name hack --set
    
规则添加顺序与执行规则排列顺序 ---相反 即A,B,C会按C,B,A依序执行   
第2条 对规定端口发送新建连接请求的IP set记录到名为hack的列表中, 然后通过本规则执行下一规则（需要全局默认规则为ACCEPT），可添加-j ACCEPT   
第1条 更新查询名为hack的记录表, 对在30分钟内对规定端口, 已发送过3次新建连接请求的IP执行丢弃, 由于set在前已先记录，所以本条规则只执行了2次ACCEPT，第3次就开始DROP丢弃    

    
## 1次性封死暴力IP的方法
该方法只要IP尝试与连接规定端口就被封死
tomato 系统  Administration/scripts/firewall 系统管理/脚本/防火墙 添加以下命令 

    iptables -I INPUT -i ppp0 -m recent --name hack --update -j DROP
    iptables -I INPUT 1 -i ppp0 -p tcp -m multiport --dport 20,25,443,135,137:139,161,1080,3389 -m state --state NEW -m recent --name hack --set -j DROP
    
规则添加顺序与执行规则排列顺序 ---相反 即A,B,C会按C,B,A依序执行   
第2条 对规定端口已发送连接请求的IP执行丢弃, 并记录到名为hack的列表中    
第1条 对名为hack的列表中记录的ip所发送的新建连接请求执行丢弃   
    
只要有人扫描来自ppp0端口的tcp端口20,25,443,135,137:139,161,1080,3389。就用iptables recent模块加入自定义列表hack，并将这些ip丢弃。 
在tomato下面可能还要启用Administration/Admin Access /Admin Restrictions 启用SSH限制 ddwrt没这个问题         

运行一段时间后查询hack可看到被拦接IP

    root@Da:/tmp/home/root# cat /proc/net/xt_recent/hack    
src=42.192.10.42 ttl: 112 last_seen: 72206479 oldest_pkt: 1 72206479   
src=1.163.192.146 ttl: 115 last_seen: 74430748 oldest_pkt: 1 74430748    
src=61.231.2.88 ttl: 115 last_seen: 73777360 oldest_pkt: 1 73777360    
src=91.188.124.166 ttl: 19 last_seen: 72714995 oldest_pkt: 1 72714995    
src=221.192.199.57 ttl: 117 last_seen: 74526423 oldest_pkt: 15 72231070, 7252680    
      
## rcheck和update的区别：    
rcheck从第1个包开始计算时间，update是在rcheck的基础上增加了从最近的DROP包开始计算阻断时间，具有准许时间和阻断时间，帮助中说update会更新last-seen时间戳。    
放在案例中，rcheck是接收到第1个数据包时开始计时，一个小时内仅限5次连接，后续的包丢弃，直到一小时过后又可以继续连接。update则是接收到第1个数据包时计算准许时间，在一个小时的准许时间内仅限5次连接，当有包被丢弃时，从最近的丢弃包开始计算阻断时间，在一个小时的阻断时间内没有接收到包，才可以继续连接。所以rcheck类似令牌桶，一小时给你5次，用完了抱歉等下个小时吧。update类似网银，连续输错5次密码，停止一小时，只不过update更严格，阻断时间是从最近的一次输错时间开始算，比如输错了5次，过了半个小时又输错一次，这时阻断时间不是剩半小时，而是从第6次重新计算，剩一小时。这个区别我不太会表述，大家自己做做测试就明白了。    
     
### 防暴力破解的命令一    

    iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --set
    iptables -I INPUT -p tcp --dport 22 -i eth0 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP
     
第一句是说，对于外来数据，如果是 TCP 协议，目标端口号是 22，网络接口是 eth0，状态是新连接，那么把它加到最近列表中。注：iptables默认ACCEPT state NEW新连接请求，不然在些就会DROP请求    
第二句是说，对于这样的连接，如果在最近列表中，并且在 60 秒内达到或者超过四次，那么丢弃该数据。其中的-m是模块的意思。         
也就是说，如果有人从一个 IP 一分钟内连接尝试四次 ssh 登录的话，那么它就会被加入黑名单，后续连接将会被丢弃。不过不知道多久以后那个 IP 才能重新连接上。      


##  字典攻击

Internet上的计算机难免遭受各类攻击，比如SSH字典攻击。如果你自信自己的密码足够强大，那么你也必须忍受系统日志中大量的“SSH Failed”记录。如果你习惯经常查看日志，那么这些记录会干扰你作出正确判断。Lesca尝试过各类方法，终于找到一种极为简单奏效的方法——利用iptables的recent模块：     

    # Prevent SSH Attack
    iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW -m recent --set --name SSH
    iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 3 --name SSH -j DROP
    # Enable Normal SSH Connection
    iptables -A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT
这条规则已经在服务器上跑了一个月，阻止了大部分攻击。它会统计一分钟内与本机22端口（SSH端口）的新建连接数，如果某IP一分钟内达到3次，就忽略之后的连接。    
另外如果你的默认INPUT策略是DROP，那么我们还需要明确告诉iptables“如果一分钟内没有连续连接超过3次，那么就允许连接”。    
最后提醒大家一点，“允许连接”和“阻止攻击”的顺序不能颠倒，否则等于没有过滤。iptables非常“忠诚”，它严格按照规则办事，所以你不应该假设它理解你的意图。    

## 
    
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
