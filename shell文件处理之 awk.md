# shell文件处理之 awk

    
+ awk 去重复
      
      sort -n | awk '{if($0!=line)print;line=$0}'
