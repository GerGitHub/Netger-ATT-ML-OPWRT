# shell文件处理之 sed

# sed命令 Option
     
    -n：静默模式 
    -i:直接修改原文件
    -e scripts -e script:可以同时执行多个脚本。
    -f /path/to/sed_scripts  命令和脚本保存在文件里调用。
       sed -f /path/to/scripts  file 
    -r：表示使用扩展的正则表达式。
        只是进行操作，不显示默认模式空间的数据。
    '/AA/d' 删除符合条件行 
    '/AA/p' 显示符合条件行
    's/AA/BB'替换内容  
    '/^AA/'每行开头
    '/AA\|BB' 多条件或者匹配或
    '/AA/{/BB}'多条件和并且匹配与
  
  
# sed command

    address:指定处理的行范围
    
    sed 'addressCommand' file ... 
    对符合地址范围进行操作。
    Address: 
    1.startline,endline 
     比如1,100
       $:最后一行
    2./RegExp/ 
        /^root/
    3./pattern1/,/pattern2/ 
        第一次被pattern匹配到的行开始，至第一次pattern2匹配到的行结束，这中间的所有行。
    4.LineNumber 
       指定行 
    5.startline，+N 
       从startline开始，向后的N行。
 
    Command：
     d:删除符合条件的行。
        sed '3,$d' /etc/fstab
        sed '/oot/d' /etc/fstab 
        注意：模式匹配，要使用 // 
        sed '1d' file 
    p:显示符合条件的行 
        sed '/^\//d' /etc/fstab 
        sed '/^\//p' /etc/fstab 
          会显示两次
          先显示P匹配，再显示所有模式空间的数据。
           a \string:在指定的行后面追加新行，内容为"string"
          sed '/^\//a \# hello world' /etc/fstab 
          添加两行：
          sed '/^\//a \#hello world \n #hi' /etc/fstab 
    
    i \sting:在指定行的前面添加新行，内容为string。
    
    r file:将指定的文件的内容添加在指定行后面。
      sed '2r /etc/issue'   /etc/fstab 
      sed '$r /etc/issue' /etc/fstab 
    
    w file:将地址指定的范围的内容另存至另一文件中。
      sed '/oot/w /tmp/oot.txt' /etc/fstab 
     
    s/pattern/string/：查找并替换 
         sed  's/oot/OOT/'  /etc/fstab 
         sed 's/^\//#/' /etc/fstab 
         sed 's/\//#/'/etc/fstab 仅替换每一行第一次被模式匹配的串。
      加修饰符 
       g：全局替换 
       i：忽略大小写 
       sed 's/\//#/g'/etc/fstab
       s///:s###
       s@@@
       sed 's#+##' 
       
    后向引用
    
    l..e:like----->liker 
        love----->lover 
    	 
    sed 's#l..e#&r#' file
    &:表示模式匹配的引用 
    
    sed 's#l..e#\1r#' file 
    
    like---->Like
    love---->Love 
    sed 's#l\(..e\)#L\1#g' file 
    
    
    history |sed 's#[[:space:]]##g'
    history | sed 's#^[[:space:]]##g'
    
    sed ''dirname


   
###  删除文件中含特定字符串的行[bash]:
      
     sed -e '/abc/d'  a.txt   // 删除a.txt中含"abc"的行，但不改变a.txt文件本身，操作之后的结果在终端显示
    
     sed -e '/abc/d'  a.txt  > a.log   // 删除a.txt中含"abc"的行，将操作之后的结果保存到a.log
     
     sed '/abc/d;/efg/d' a.txt > a.log    // 删除含字符串"abc"或“efg"的行，将结果保存到a.log

      
其中，"abc"也可以用正则表达式来代替。
            
     curl -k https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts | sed '/github./d;baidu./d;/qq./d;tencent./d' >/opt/Shells/GoNoAD/StevenBlack      
     sed '/abc/d;/xyz/d' 可删除指定字符abc和xyz行
     
     
### 删除文件 多个 或者 关键字符方式 匹配或
       
sed 匹配100_1000或bigger_1000    

    sed -n '/100_1000\|bigger_1000/p' 20160220
    sed -n '/1001000∥bigger1000/p' 20160220
        
注： 用  \| 或 // 表示匹配或者 前后条件之一   /p 显示符合条件行 


### 删除文件 多个 和 并且 关键字符方式 匹配与
     
      sed -i '/^0\|^:/{/github/d}' aa.txt  删除aa.txt中以 0 或者 ：开头，并且含有github的行
      
      
### 例子🌰
     1.删除/etc/grub.conf文件中行首的空白符；
        sed  's/^[[:space:]]+//g' /etc/grub.conf 
     2.替换/etc/inittab文件中"id:3:initdefault:"一行中的3
       sed 's#id:3:init#id:5:initd#'
       sed 's@\(id:\)[0-9]\(:initdefault:\)@\15\2@g' /etc/inittab 
     3.删除/etc/inittab文件中的空白行。
       sed '/^$/d' /etc/inittab
     4.删除/etc/inittab文件中开头的#号
       sed 's/^#//'  
     5.删除莫文件中开头的#号以及空白行。
       sed 's/^[[:space:]]+//g' 
     6.删除某文件中以空白字符后面跟#类的行中开头的空白字符以及#
       sed -r 's/^[[:space:]]+#//g' 
     7.取出一个文件路径的目录名称
       echo '/etc/rc.d'|sed -r 's@^(/.*/)[^/]+/?@\1@g'

      
      

