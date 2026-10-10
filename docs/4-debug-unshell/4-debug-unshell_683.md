

# eXPressor1.8 试用版加壳记事本脱壳

> 原文：[`www.pediy.com/kssd/pediy11/105463.html`](https://www.pediy.com/kssd/pediy11/105463.html)

【文章标题】: eXPressor1.8 试用版加壳记事本脱壳  
【文章作者】: lovebird/爱鸟  
【作者邮箱】: lovebird\_ustc#126.com  
【作者 QQ 号】: 1282260064  
【软件名称】: 记事本  
【软件大小】: 脱前 53.5KB 脱后 65KB  
【下载地址】: 附件有  
【加壳方式】: eXPressor1.8 试用版  
【保护方式】: eXPressor 试用版加壳  
【编写语言】: VC  
【使用工具】: OD+dump 插件  
【操作平台】: SP3+VM6  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
\--------------------------------------------------------------------------------  
【详细过程】  
  主程序不能在虚拟机里面运行，可以下载修改版的虚拟机安装。主程序貌似是 Asprotect 加的壳，因为不是这阶段要学习的重点，所以不管它，等脱完压缩壳再弄。  
  程序界面和以前的 eXPressor 一样，貌似那个 Protect 在试用版不能用。所以我这边就看不到那些加密文件的东西。  
  这样，这款加密壳就变成没有任何技术含量的压缩壳了。把其他勾全部勾选上，除了资源压缩。Mute 填入 lovebird。  
  \[eXPressor 1.8 (Trial)-> CGSoftLabs   \* Sign.By.++++++++ \* 20100119\]  
  signature = 55 8B EC 81 EC 84 02 00 00 53 56 57 83 A5 A8 FD FF FF 00 F3  
  ep\_only = false  
  这个试用版壳平淡无奇，直接 ESP 定律就行。OD 不躲不藏，忽略一切异常。  
  01017442 > $  55            PUSH EBP                                 ;  停在这儿  
  01017443   .  8BEC          MOV EBP,ESP  
  01017445   .  81EC 84020000 SUB ESP,284  
  0101744B   .  53            PUSH EBX  
  0101744C   .  56            PUSH ESI  
  0101744D   .  57            PUSH EDI  
  0101744E   .  83A5 A8FDFFFF>AND DWORD PTR SS:\[EBP\-258\],0             ;  此处 hr esp  
  01017455   .  F3:           PREFIX REP:                              ;  多余的前缀  
  01017456   .  EB 0C         JMP SHORT notepadP.01017464  
  01017458   .  65 58 50 72 2>ASCII "eXPr-v.1.6.",0  
  01017464   >  A1 00700101   MOV EAX,DWORD PTR DS:\[1017000\]  
  01017469   .  05 00700101   ADD EAX,notepadP.01017000  
  =====================================================  
  shift+f9，点掉那个试用的 NAG，到了这儿：  
  01017B94   .  5E            POP ESI                                  ;  停在这儿 搂一眼 eax 就是 OEP  
  01017B95   .  5B            POP EBX  
  01017B96   .  8BE5          MOV ESP,EBP  
  01017B98   .  5D            POP EBP  
  01017B99   .  50            PUSH EAX  
  01017B9A   .  A1 04700101   MOV EAX,DWORD PTR DS:\[1017004\]  
  01017B9F   .  8378 6C 00    CMP DWORD PTR DS:\[EAX+6C\],0  
  01017BA3   .  75 05         JNZ SHORT notepadP.01017BAA  
  01017BA5   .  58            POP EAX  
  01017BA6   .  33C0          XOR EAX,EAX  
  01017BA8   .  40            INC EAX  
  01017BA9   .  C3            RETN  
  01017BAA   >  58            POP EAX  
  01017BAB   .  FFE0          JMP EAX                                  ;  跳到 oep  
  此处 F4 到 01017BAB，然后 F8 一下，就到了 OEP 了。  
  要想去除加壳后的 NAG，和以前的版本一样：  
  01017442 > $  55            PUSH EBP                                 ;  停在这儿  
  01017443   .  8BEC          MOV EBP,ESP  
  01017445   .  81EC 84020000 SUB ESP,284  
  0101744B   .  53            PUSH EBX  
  0101744C   .  56            PUSH ESI  
  0101744D   .  57            PUSH EDI  
  0101744E   .  83A5 A8FDFFFF>AND DWORD PTR SS:\[EBP\-258\],0             ;  此处 hr esp  
  01017455   .  F3:           PREFIX REP:                              ;  多余的前缀  
  01017456   .  EB 0C         JMP SHORT notepadP.01017464  
  01017458   .  65 58 50 72 2>ASCII "eXPr-v.1.6.",0  
  01017464   >  A1 00700101   MOV EAX,DWORD PTR DS:\[1017000\]  
  01017469   .  05 00700101   ADD EAX,notepadP.01017000  
  0101746E   .  A3 04700101   MOV DWORD PTR DS:\[1017004\],EAX  
  01017473   .  A1 04700101   MOV EAX,DWORD PTR DS:\[1017004\]  
  01017478   .  8378 70 00    CMP DWORD PTR DS:\[EAX+70\],0  
  0101747C      75 14         JNZ SHORT notepadP.01017492              ; 改成 jmp 保存即可  
  0101747E   .  6A 10         PUSH 10                                  ; /Style = MB\_OK|MB\_ICONHAND|MB\_APPLMODAL  
  01017480   .  68 F0700101   PUSH notepadP.010170F0                   ; |Title = "Info"  
  01017485   .  68 78700101   PUSH notepadP.01017078                   ; |Text = "This application was packed with an Unregistered version of eXPressor.  
For registration, please visit www.cgsoftlabs.ro"  
  0101748A   .  6A 00         PUSH 0                                   ; |hOwner = NULL  
  0101748C   .  FF15 50700101 CALL DWORD PTR DS:\[<&USER32.MessageBoxA>>; \\MessageBoxA  
  01017492   >  E8 F0FEFFFF   CALL notepadP.01017387  
    
    
\--------------------------------------------------------------------------------  
【经验总结】  
  秒脱方法:  
     直接 Ctrl+F 搜索指令 Jmp eax，下断 shift+F9 运行 取消断点后 F8 就到了 OEP   
    
\--------------------------------------------------------------------------------  
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!  
  
                                                       2010 年 01 月 19 日 0:56:45

上传的附件
| 附件请到原帖下载：[`bbs.pediy.com/showthread.php?t=105463`](http://bbs.pediy.com/showthread.php?t=105463)  
 |
|  |