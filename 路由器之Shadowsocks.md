## 路由器之Shadowsocks

### Advanced Tomato 安装 Shadowsocks-libev

+ 挂载USB, 安装 entware-ng    

+ 安装shadowsocks-libev          
      
    opkg update
    opkg install shadowsocks
     
+ winscp登录 或者 vi 命令修改 /opt/etc/init.d/S22shadowsocks              
    
      #!/bin/sh
    
      ENABLED=yes
      PROCS=ss-local
      ARGS="-c /opt/etc/shadowsocks.json"
      PREARGS=""
      DESC=$PROCS
      PATH=/opt/sbin:/opt/bin:/opt/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    
      . /opt/etc/init.d/rc.func     
    
      // 本行及以下内容为注释
      ROCS=ss-local 可改为 ROCS=ss-redir 
      ARGS="-c /opt/etc/shadowsocks.json" 必须改为 ARGS="-b 0.0.0.0 -c /opt/etc/shadowsocks.json" ，本人就掉在这坑里；
    
    
+ 修改 /opt/etc/shadowsocks.json     
   
      {
       "server":"your server ip",
       "server_port":your server prot,
       "local_address":"0.0.0.0",
       "local_port":1080,
       "password":"your server pwd",
       "timeout":60,
       "method":"aes-256-cfb"
      }
   
+ 关于掉坑的问题           
  [Asuswrt-merlin固件entware-arm环境SS番茄教程（基本完工，持续...](http://www.52asus.com/thread-1009-1-1.html)
