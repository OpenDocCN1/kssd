

# TextMagic V1.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue171.htm`](https://www.pediy.com/kssd/pediy04/kanxue171.htm)

Software:TextMagic  Version 1.0  (1.67MB)   
  
Tool:  trw2000,w32dasm89 & hiew32   
  
http://www.alwaysu.net/software/web/texiao/textmagic%20%20en.exe   
  
  
求人指点有感:   
          近日在本论坛发帖求人指点 TextMagic  Version 1.0 存盘 6 个文字限制,没有得到响应.   
          有个别朋友极不耐烦,令我感慨万千:世界上真没有救世主,一切要靠我们自己.   
  
  
procedures:   
  
1.首先运行 trw2000,在存盘前用 bpx hmemcpy 设断点,找到 TextMagic V1.0 存盘 6 个文字限制的提示框   
的位址:   
  
:0040AF26 CALL 0043C3D6------执行此 call 则弹出存盘 6 个文字限制的提示框.   
  
!!!!!注:这个软件在 trw2000 环境下和在 w32dasm89 环境下 RVA 相同.   
  
  
2.运行 W32dasm89 反汇编 TextMagic.exe  ,点击 find text,填入 "Warning: This is a trial version of Textmagic",   
来到这里.   
  
!!!!(为什么不直接在 W32dasm89 的字符串参考里寻找上述内容呢?----因为找不到.)   
  
  
  
  
\* Possible Reference to String Resource ID=61478: "Warning: This is a trial version of Textmagic. Trial version"   
                                  |   
:0040AF1D 6826F00000              push 0000F026   
:0040AF22 C645FC05                mov \[ebp\-04\], 05   
:0040AF26 E8AB140300              call 0043C3D6   
:0040AF2B 8D83BA020000            lea eax, dword ptr \[ebx+000002BA\]   
:0040AF31 50                      push eax   
:0040AF32 E879FD0000              call 0041ACB0   
:0040AF37 83F806                  cmp eax, 00000006   
                                  !!!!!!!!!!!!!!!!!  ----这儿的 00000006 是什么意思?   
                                                          难道说它就是 6 个文字限制的标志?        
:0040AF3A 59                      pop ecx   
:0040AF3B 7605                    jbe 0040AF42   
    a33bh(内存地址)              !!!!!!!!!!!!!!-----------试着把 jbe 改为 jmp   
  
:0040AF3D 6A06                    push 00000006   
:0040AF3F 58                      pop eax   
    a340h   
:0040AF40 EB0D                    jmp 0040AF4F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AF3B(C)   
|   
:0040AF42 8D83BA020000            lea eax, dword ptr \[ebx+000002BA\]   
:0040AF48 50                      push eax   
:0040AF49 E862FD0000              call 0041ACB0   
:0040AF4E 59                      pop ecx   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0040AF40(U)   
|   
:0040AF4F 50                      push eax   
:0040AF50 8B4DDC                  mov ecx, dword ptr \[ebp\-24\]   
:0040AF53 8D83BA020000            lea eax, dword ptr \[ebx+000002BA\]   
:0040AF59 50                      push eax   
:0040AF5A 8D45B0                  lea eax, dword ptr \[ebp\-50\]   
:0040AF5D FF75E8                  push \[ebp\-18\]   
:0040AF60 50                      push eax   
:0040AF61 E89898FFFF              call 004047FE   
:0040AF66 8BCF                    mov ecx, edi   
:0040AF68 E8AEA6FFFF              call 0040561B   
:0040AF6D 668B4704                mov ax, word ptr \[edi+04\]   
:0040AF71 668B7F06                mov di, word ptr \[edi+06\]   
:0040AF75 663BC6                  cmp ax, si   
:0040AF78 7E05                    jle 0040AF7F   
:0040AF7A 0FBFC0                  movsx eax, ax   
:0040AF7D EB02                    jmp 0040AF81   
  
  
  
3.用 hiew32 把   
:0040AF3B 7605                    jbe 0040AF42   
  
改为:   
  
:0040AF3B 7605                  jmp 0040AF42   
  
4.运行 TextMagic Trial  Version 1.0 ,制作 18 个文字,然后存盘,结果大功告成.   
  
困扰我多日的难题,终于告破.  4dd466d0e1e0cc123afbe504da2462ca