

# 快速判断 Armadillo 壳版本

> 原文：[`www.pediy.com/kssd/pediy06/pediy6691.htm`](https://www.pediy.com/kssd/pediy06/pediy6691.htm)

偶尔看到的，试了一下，感觉比较好用，原文在[`66.98.132.48/forum/showthread.php?t=6085`](http://66.98.132.48/forum/showthread.php?t=6085)  
  
大概是这样  
设断点 CreateThread 或者 SetProcessWorkingSetSize  
停下来后，Ctrl+F9 返回，  
然后右键 Search for -- All Referenced text strings  
到 text reference 窗口，右键 Search for text  
选中 Entire scope 搜索字符串: armVersion  
来到这里  
Text strings referenced in 01030000..01077FFF, item 57  
 Address=0103481C  
 Disassembly=PUSH 106300C  
 Text string=ASCII "   <armVersion xsi:type="xsd:string">%s</armVersion>  
"  
貌似 Armadillo 的 xml 格式配置文件?  
双击该行来到 disassemble 窗口，向上拖动一行  
看到  
01034817    68 44300601     PUSH 1063044                             ; ASCII "3.76"  
0103481C    68 0C300601     PUSH 106300C                             ; ASCII "   <armVersion xsi:type="xsd:string">%s</armVersion>  
"  
啊哈，这正是 Armadillo 的版本号，据说此法从 Armadillo3.20c 到最新版的 Armadillo 都支持。

 4dd466d0e1e0cc123afbe504da2462ca