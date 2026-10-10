

# UltraEdit 9.00 

> 原文：[`www.pediy.com/kssd/pediy04/kanxue368.htm`](https://www.pediy.com/kssd/pediy04/kanxue368.htm)

程序名称：UltraEdit 9.00 简繁双语第二版   
软件简介：不用说了吧   
下载地址：http://202.102.249.231/hanhua/DOWNLOAD/cuedit900\_2.exe   
破解目的：找出注册码。   
工具：    Trw2000 1.22 ,w32dsm89 黄金版   
说明：   
    Uedit32 9.00 和 Uedit32 8.0x 一样采用 key 文件保护方式，用的是文件 Uedit32.reg。   
先在 Help 菜单下找到 Register UltraEdit-32，点击会弹出一个对话框。填入你的大名，   
还有注册码。我用 000000，12034-12340-12034-10234 注册（注册码形式：XX0XX-XXXX0-   
XX0XX-X0XXX）下断点 Bpx readfile do "d esp\-100",按几次 F5 你会在 SoftICE 的数据窗口看到 C:\\Program Files\\UltraEdit\\UEDIT32.reg 字样，按 12 次 F12 后，按 f10 一步步地跟踪到下面：   
  
  
  
  
\* Reference To: USER32.CreateWindowExA, Ord:0059h   
                                  |   
:00451608 FF1598775200            Call dword ptr \[00527798\]   
:0045160E A3AC275700              mov dword ptr \[005727AC\], eax   
:00451613 8D8700050000            lea eax, dword ptr \[edi+00000500\]   
:00451619 885810                  mov byte ptr \[eax+10\], bl   
:0045161C FF352C8F5600            push dword ptr \[00568F2C\]   
:00451622 6A01                    push 00000001   
:00451624 50                      push eax   
:00451625 FF75F0                  push \[ebp\-10\]   
:00451628 E83A2BFCFF              call 00414167          <--关键，跟进   
:0045162D 83C410                  add esp, 00000010   
:00451630 85C0                    test eax, eax   
:00451632 0F8489010000            je 004517C1   
:00451638 391D482A5700            cmp dword ptr \[00572A48\], ebx   
:0045163E 0F857D010000            jne 004517C1   
  
跟进上面的 call 后，一步步地跟踪到下面：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00414181(C)   
|   
:0041418D 395D10                  cmp dword ptr \[ebp+10\], ebx   
:00414190 0F84DE020000            je 00414474   
:00414196 83F806                  cmp eax, 00000006  <--名字大于等于 6   
:00414199 0F8CD5020000            jl 00414474        <--名字小于 6 就跳   
:0041419F 8065EC00                and byte ptr \[ebp\-14\], 00   
:004141A3 8065AC00                and byte ptr \[ebp\-54\], 00   
:004141A7 80A56CFFFFFF00          and byte ptr \[ebp+FFFFFF6C\], 00   
:004141AE 395D14                  cmp dword ptr \[ebp+14\], ebx   
:004141B1 0F84BD020000            je 00414474   
:004141B7 57                      push edi   
:004141B8 56                      push esi   
:004141B9 E872620C00              call 004DA430   
:004141BE 59                      pop ecx   
  
然后一步步地跟踪到下面，就到目的地了：   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0041441C(C)   
|   
:0041443E 8D8570FFFFFF            lea eax, dword ptr \[ebp+FFFFFF70\]   
:00414444 50                      push eax   
:00414445 8D45B0                  lea eax, dword ptr \[ebp\-50\]   
:00414448 50                      push eax          <--输入的注册码   
:00414449 E822760C00              call 004DBA70   
:0041444E 59                      pop ecx   
:0041444F 85C0                    test eax, eax   
:00414451 59                      pop ecx   
:00414452 7426                    je 0041447A   
:00414454 8D8530FFFFFF            lea eax, dword ptr \[ebp+FFFFFF30\]   
:0041445A 50                      push eax          <--正确的注册码   
:0041445B 8D45B0                  lea eax, dword ptr \[ebp\-50\]   
:0041445E 50                      push eax   
:0041445F E80C760C00              call 004DBA70    <--比较   
:00414464 59                      pop ecx   
:00414465 85C0                    test eax, eax   
:00414467 59                      pop ecx   
:00414468 7410                    je 0041447A   
:0041446A C705648F560001000000    mov dword ptr \[00568F64\], 00000001   
  
  
总结：   
  
用户名：000000   
授权码：B4F0J-Q901S-08R6A-A301X 

程序：Uedit32.exe   
添加中断地址列表：   
  
中断地址 451628   
中断次数 1   
第一字节 E8   
指令长度 5   
  
再次添加：   
  
中断地址 41445F   
中断次数 1   
第一字节 E8   
指令长度 5   
  
选择内存方式：83F860  4dd466d0e1e0cc123afbe504da2462ca