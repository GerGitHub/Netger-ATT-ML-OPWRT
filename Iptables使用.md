###Iptables使用   


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
注意：iptables的规则匹配顺序上从上到下的，也就是说如果上下两条规则有冲突时，将会以上面的规则为准。
