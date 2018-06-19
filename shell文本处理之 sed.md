# shell文本处理之 sed
   
###  删除文件中含特定字符串的行[bash]:
      
     sed -e '/abc/d'  a.txt   // 删除a.txt中含"abc"的行，但不改变a.txt文件本身，操作之后的结果在终端显示
    
     sed -e '/abc/d'  a.txt  > a.log   // 删除a.txt中含"abc"的行，将操作之后的结果保存到a.log
     
     sed '/abc/d;/efg/d' a.txt > a.log    // 删除含字符串"abc"或“efg"的行，将结果保存到a.log

      
其中，"abc"也可以用正则表达式来代替。
            
     curl -k https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts | sed '/github./d;baidu./d;/qq./d;tencent./d' >/opt/Shells/GoNoAD/StevenBlack      
     sed '/abc/d;/xyz/d' 可删除指定字符abc和xyz行
