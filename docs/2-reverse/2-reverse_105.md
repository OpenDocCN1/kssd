

# pecompact 最新版辅助脚本 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6689.htm`](https://www.pediy.com/kssd/pediy06/pediy6689.htm)

笔快生锈了，发一贴:pecompact 最新版辅助脚本.注意了，这个脚本不能帮你搞定它的主程序(当然它的主程序也不难，压缩率还不错)，不过可以参考的说.也算是我换工作后的第一贴吧.  
这个应该是算最新版的了，5 月 7 号了，可能是换 boss 的原故，升级特利害.  
这么快就升到 2.08 版了  
Enjoy!  
  
标志:  
pecompact v2.08's signature for peid v0.92  
  
copy signature text->open"userdb.txt"->paste  
  
signature text:  
  

> ```
> 代码:
> \[PeCompact v2.08->Bitsum Technologies(signature by loveboom)\]
> signature = B8 ?? ?? ?? ?? 50 64 FF 35 00 00 00 00 64 89 25 00 00 00 00 33 C0 89 08 50 45 43 6F 6D
> ep\_only=true
> 
> ```
> 
>   

  
脚本:  
  
/\*  
//////////////////////////////////////////////////  
  Pecompact v2.08 OEP Finder v0.1 beta  
  Author:  loveboom  
  Email : bmd2chen@tom.com  
  OS    : winxp,OllyDbg 1.1C,OllyScript v0.7  
  Date  : 2004-5-8  
  Config: Ignore all exceptions.  
  Note  : Donn't use this script unpack "pe2gui.exe"  
        If you have one or more question, email me please,thank you!  
//////////////////////////////////////////////////  
\*/  
var addr  
  
start:  
  gpa "VirtualFree","kernel32.dll"  
  cmp $RESULT,0  
  JE lblabort  
  mov addr,$RESULT  
  bp addr  
  run  
  
lbl1:  
  run  
  bc addr   //Clear break point  
  rtu  
  
lbl2:  
  findop eip,#FFE0#  
  cmp $RESULT,0  
  je lblabort  
  mov addr,$RESULT  
  bp addr  
  eob lbl3  
  run  
  
lbl3:  
  bc addr  
  sto  
  
lblend:  
  cmt eip,"OEP Found,Please dumped it!"  
  msg "Script by loveboom\[DFCG\]\[FCG\],Thank you for using my script!"  
  ret  
  
lblabort:  
  msg "Error! script abort, Maybe target is not packed by pecompact v2.08,more information,mailto bmd2chen@tom.com."  
  ret  4dd466d0e1e0cc123afbe504da2462ca