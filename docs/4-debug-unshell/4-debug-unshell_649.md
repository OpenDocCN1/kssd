

# EXECryptor2251 脫殼 + 跨平台 OK

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-649.htm`](https://www.pediy.com/kssd/pediy07/pediy7-649.htm)

【破解作者】 SYSCOM  
【使用工具】 DS 2.7 ,IceExt ,IceDump ,LordPE ,ImportREC1.6 ,WinHex  
【破解平台】 Windows XP SP1  
【软件名称】 EXECryptor 2,2,5,1  
【下载地址】 http://www.pediy.com/tools.htm (看雪工具區)  
【软件简介】 加密 .EXE-工具   
【加壳方式】  EXECryptor  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  
\----------------------------------------------------------------------  
  
(1) 找尋 OEP:  
  
     使用 ESP 定律,再配合 EXECryptor 2,2,5,1 ,加密入口特徵碼  
      可以找出 OEP=5A3875  
  
(2)Dump Memory  
  
     先設斷點在, BPM 6DEE76 X  (TLS 進入點),後離開程序  
     再執行後,中斷再 6DEE76 ,後設斷點 BPX GetProcAddress  
      中斷後,返回主程序,6DEC97,令 A, 6DEC97-JMP 6DEC97  
     清除所有斷點後,開始 Dump Memory ,  
     令  Pagein D 400000 2DF000 \\??\\C:\\DUMP.BIN  
     按 F5 後,修正 PE ,令 RA=VA,RS=VS,成為有效 PE 檔  
     變成 DUMP.EXE (小圖示會出現)   
  
(3) 重建 New IAT  
  
     寫一 Patch ,還原 Import Table,先令 588162-JMP Patch+Offset 0X29  
  
+++++++++++++++++++ ASSEMBLY CODE START ++++++++++++++++++  
  
:00000000     mov edx, 004ED168  
:00000005     cmp word ptr \[edx+02\], 0040  
:0000000A     jb 0000001C  
:0000000C     cmp word ptr \[edx+02\], 005F  
:00000011     ja 0000001C  
:00000013     pushad  
:00000014     mov dword ptr \[004ED104\], esp  
:0000001A     jmp dword ptr \[edx\]  
:0000001C     add edx, 00000004  
:0000001F     cmp edx, 004ED908  
:00000025     jbe 00000005  
:00000027     jmp 00000027  
:00000029     mov dword ptr \[ebp-0C\], eax  
:0000002C     cmp ebp, 00130000  
:00000032     jb 00000038  
:00000034      mov esp, ebp  
:00000036      pop ebp  
:00000037      ret  
:00000038      mov dword ptr \[004ED100\], eax  
:0000003D      mov esp, dword ptr \[004ED104\]  
:00000043      popad  
:00000044      mov ebx, dword ptr \[004ED100\]  
:0000004A      mov dword ptr \[edx\], ebx  
:0000004C      jmp 0000001C  
  
+++++++++++++++++++ ASSEMBLY CODE END ++++++++++++++++++  
  
Patch 完後,還要再增加,兩個函數 Kernel32.dll and User32.dll  
  
Kernel32.dll                   User32.dll  
\---------------------------------------------------------  
GetModuleHandleA                MessageBoxA  
LoadLibraryA  
GetProcAddress  
ExitProcess  
VirtualAlloc  
VirtualFree  
\---------------------------------------------------------  
增加完後,用 ImportREC1.6 ,令 RAV = ED144 , Size= 7C4  
  重建 New IAT,得到 DUMP\_.EXE  
  
(4)用 LordPE,修正 OEP=1A3875 ,TLS=F2000 及 重新定位  
    新增加的,兩個函數 Kernel32.dll and User32.dll  
  
DllName          OFT        TDS         FC       Name        FT  
\--------------------------------------------------------------------  
Kernel32.dll   000AB18B   00000000   00000000   000AB0FB   000AB173  
User32.dll     000AB1C7   00000000   00000000   000AB1A7   000AB1C3  
\--------------------------------------------------------------------  
  
(5)去除自我校驗  
  
58309F-改-> C3  
533D30-改-> C3  
  
(6)TEST RUN OK~!!!

> 引用:
> 
> * * *
> 
> *最初由 syscom 发布*  
> **  
>   
> 我適用 IceExt 0.67 ,這個沒有影響,只是用來避開 Debug 檢測而已....  
>   
> 這是不錯的 Dump 點,用 SP2 ,也同樣適用的,和 OS 無關.... **
> 
> * * *

  
  
  
早上试了一下果真如此, 这个点好找而且干净, 严重学习了  
只是 pagein 我没法试, 能一次性 dump 2df000 这么大? 

> 引用:
> 
> * * *
> 
> *最初由 shoooo 发布*  
> **  
>   
>   
> 早上试了一下果真如此, 这个点好找而且干净, 严重学习了  
> 只是 pagein 我没法试, 能一次性 dump 2df000 这么大? **
> 
> * * *

  
  
!Dump  命令,必須是連續區塊的  VM  
  
Pagein 命令,可以不是連續區塊 VM,未使用 VM ,自動補 00  
  
可以  SIZE 2DF000 ,還可以更大........... 

 4dd466d0e1e0cc123afbe504da2462ca