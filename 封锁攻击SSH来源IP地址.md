#封锁攻击SSH来源IP地址

### 命令方式
       
    cat /mnt/sda1/Shells/black2.list | awk '/Failed/{print $(NF-3)}' |sort |uniq -c|awk  '{print $2"="$1}'  


  #由于tomato没有uniq命令 使用awk  '{a[$1]++}END{for (j in a) print a[j],j | "sort -r -k 1"}' 来替代 uniq -c 功能， 去重复并统计重复数

    cat /var/log/messages | awk '/Bad password | Login attempt/{print $(NF-0)}'|awk -F : '{print $1}'|sort |awk  '{a[$1]++}END{for (j in a) print a[j],j | "sort -r -k 1"}'|awk  '{print $2"="$1}'  
  
  
## 参考资料 1
[ssh访问控制，多次失败登录即封掉IP，防止暴力破解](http://www.cnblogs.com/panblack/p/secure_ssh_auto_block.html)          
一、系统：Centos6.3 64位   
二、方法：读取/var/log/secure，查找关键字 Failed，例如（注：文中的IP地址特意做了删减）：    
      
Sep 17 09:08:09 localhost sshd[29087]: Failed password for root from 13.7.3.6 port 44367 ssh2             
Sep 17 09:08:20 localhost sshd[29087]: Failed password for root from 13.7.3.6 port 44367 ssh2            
Sep 17 09:10:02 localhost sshd[29223]: Failed password for root from 13.7.3.6 port 56482 ssh2             
Sep 17 09:10:14 localhost sshd[29223]: Failed password for root from 13.7.3.6 port 56482 ssh2            
         
从这些行中提取IP地址，如果次数达到10次(脚本中判断次数字符长度是否大于1)则将该IP写到 /etc/hosts.deny中。          
          
三、步骤：         
1、先把始终允许的IP填入 /etc/hosts.allow ，这很重要！比如：    
   
      sshd:19.16.18.1:allow            
      sshd:19.16.18.2:allow                
2、脚本 /usr/local/bin/secure_ssh.sh             
                    
        #! /bin/bash
        cat /var/log/secure|awk '/Failed/{print $(NF-3)}'|sort|uniq -c|awk '{print $2"="$1;}' > /usr/local/bin/black.list
        for i in `cat  /usr/local/bin/black.list`
        do
          IP=`echo $i |awk -F= '{print $1}'`
          NUM=`echo $i|awk -F= '{print $2}'`
          if [ ${#NUM} -gt 1 ]; then
            grep $IP /etc/hosts.deny > /dev/null
            if [ $? -gt 0 ];then
              echo "sshd:$IP:deny" >> /etc/hosts.deny
            fi
          fi
        done
        
 3、将secure_ssh.sh脚本放入cron计划任务，每1分钟执行一次。           
       
      # crontab -e
      */1 * * * *  sh /usr/local/bin/secure_ssh.sh
      
## 参考资料 2   
[shell awk 统计重复个数](http://leichenlei.iteye.com/blog/1676649)                
有文件file.log内容如下：         

http://www.sohu.com/aaa     
http://www.sina.com/111           
http://www.sohu.com/bbb                
http://www.sina.com/222             
http://www.sohu.com/ccc           
http://www.163.com/zzz         
http://www.sohu.com/ddd           
要统每个域名出现次数：               
http://www.sohu.com 4               
http://www.sina.com 2                 
http://www.163.com 1              
              
答案是：  awk -F / '{a[$3]++} END{for(i in a){print i,a[i] | "sort -r -k 2"}}' file.log;             
            
解释一下，awk语法就不说了：            
-F参数是制定awk分隔符，这里制定的是 /,所以每行被分成4个部分。             
sort 的-r是降序，-k是按照第几组字符排序，从1开始。              
a可以理解成key-value形式的对象，域名做key 个数做value。            
在end动作里完成对结果a的打印，         
注意： 这个方法还可以用来统计日志中响应时间等等。    
    
## 参考资料 3
[awk的sort命令学习一例](http://liran728729.blog.51cto.com/2505117/1152213)    
采用awk内建的排序函数asort，asorti进行。             
#cat 1 | awk '{a[$NF]=$0}END{l=asorti(a,b);for(i=1;i<=l;i++)print a[b[i]]}            
思路，用每一行的第三列作为数组下标，整行作为值，然后对下标进行排序，后输出数组的值。                   
伪代码如下：             
a[56]="12 34 56"            
a[12]="78 90 12"              
a[89]="23 45 89"    
然后对下标排序。          
a[12]="78 90 12"       
a[56]="12 34 56"                   
a[89]="23 45 89"                   
然后输出就OK了            
            
1，NF为域的个数       
#cat 1 | awk '{print NF,$NF}'           
3 56       
3 12                     
3 89             
从上面可以看出上面默认分为了3个域，所以NF=3，$NF=$3,awk是行读入，所以$3即为输出每一行的第三个域的值。         
2，asort，asorti的用法           
参考：http://www.gnu.org/software/gawk/manual/gawk.html#Array-Sorting           



