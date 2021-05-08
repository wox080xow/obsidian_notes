# 糟糕的printf
### 緣起
本來想要script做一個簡單查看MacBook Pro硬體的功能，本來是用echo輸出文字，但是排版有點辛苦，想說看看有沒有其他指令可以用，於是發現了printf。不過真正使用後才發現是個災難，printf在macOS上無法正確處理中文！我弄了一個示範給大家看看...

### 示範
編輯文件`printftest.sh`，其實隨便叫什麼名字都行啦！內容如下：
```
echo 'printf "%10s\\n" <string>'
printf "%10s\n" 0123456789
printf "%10s\n" abcdefghij
printf "%10s\n" ajkfj
printf "%10s\n" 一二三四五
printf "%10s\n" 一二三四
printf "%10s\n" 一二12
echo
echo 'printf "%-10s\\n" <string>'
printf "%-10s\n" 0123456789
printf "%-10s\n" abcdefghij
printf "%-10s\n" ajkfj
printf "%-10s\n" 一二三四五
printf "%-10s\n" 一二三四
printf "%-10s\n" 一二12
echo
echo 'printf "%10s %-10s\\n" <string> <string>'
printf "%10s %-10s\n" 0123456789 0123456789
printf "%10s %-10s\n" abcdefghij abcdefghij
printf "%10s %-10s\n" ajkfj ajkfj
printf "%10s %-10s\n" 一二三四五 一二三四五
printf "%10s %-10s\n" 一二三四 一二三四
printf "%10s %-10s\n" 一二12 一二12
```
執行`printftest.sh`
```
sh printftest.sh
```
大家可以看到如下慘狀
- 完全不處理純中文
- 中英數混和的字串只處理到一半，傻眼...

![](https://i.imgur.com/QENCd33.png)

### 碎念
這才想到某個DBA同事正在處理中文寬度的問題，由於前端網頁的程式只抓字元數，字元數相同但是中文字比較多時，字串會太寬導致部分資訊被切掉，我猜應該就是類似printf功能的function出的包...
