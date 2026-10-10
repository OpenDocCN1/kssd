

# 几种少见的 PE 感染方式.

> 原文：[`www.pediy.com/kssd/pediy11/104328.html`](https://www.pediy.com/kssd/pediy11/104328.html)

祝看雪的各位新年快乐!  
  
第一种  
将代码(Shellcode)分散到多个文件.  
比如感染 QQ, 可以选择 QQ.exe 和 N 个 QQ.exe 自带的 Dll 文件.  
假设这里选择 QQ.exe IM.dll QQZip.dll  
主要代码插入 IM.dll 和 QQZip.dll, Load 代码插入 QQ.exe  
Load 代码只需要找到代码所在的两个 DLL 的位置 然后跳转过去就 OK 了.  
  
优点 比感染单个文件更难查杀(对于启发式来说),难清除(?),不用像单个文件一样太注意 Shellcode 的大小(在不增加文件大小的情况下).  
缺点 相对麻烦,稳定性没单个文件好.  
  
第二种  
属于 EAT HOOK. 但是比传统的 EAT HOOK 难发现.   
只需要一句 你们就能明白.  

引用: #pragma comment( linker, "/EXPORT:ftsWordBreak=lpk.ftsWordBreak") 

优点 非常隐蔽, 至少到现在为止, 所有的安全软件都不查杀这种方式.  
缺点 需要多存在一个 DLL 文件.  
  
P.S.  
上面两种感染方式我都没有见过具体样本或者代码. 但杀毒软件对于以上两种感染方式的查杀确实...  
现在只有木马, 没有病毒. 真龌龊.