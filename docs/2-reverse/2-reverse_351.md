

# 让 WinDbg、IDA、VC 自动下载符号表

> 原文：[`www.pediy.com/kssd/pediy11/107893.html`](https://www.pediy.com/kssd/pediy11/107893.html)

很多人跟踪到 Windows 自带的 DLL 里的时候有时会很茫然……  
突然找到一种方法能让它们都有符号表……  
然后里面的各种变量名、函数名等全部显现出来……  
  
设置方法：  
“我的电脑”属性中，高级设置里，  
有个环境变量，  
变量名：\_NT\_SYMBOL\_PATH  
变量值：SRV\*{$Path}\*http://msdl.microsoft.com/download/symbols/  
将“{$Path}”替换为要存储 pdb 符号表文件的路径，比如：C:\\PDB  
  
于是在 VC 里写程序调试时，或者用 IDA 时，都会从网上自动下载符号表……  
msvcrt.pdb，ole32.pdb，System.Data.pdb，ws2\_32.pdb，WebDev.WebServer.pdb……  
  
但是 VC 运行程序时会自动下载完所有要用到的 pdb，所以第一次启动会很慢，若想立即看结果，只要把网络断开就行了……  
仅此分享……![](img/cda69129882b40377a87144c782680e1.jpg "微笑")