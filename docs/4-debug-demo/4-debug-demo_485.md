

# 大富翁五

> 原文：[`www.pediy.com/kssd/pediy03/forum1260.htm`](https://www.pediy.com/kssd/pediy03/forum1260.htm)

哈！哈！哈！我等了两年的大富翁五终于出来了，可是又是要用光盘才能玩，没办法啦！只好把它彻底破解，以免每次玩的时候都要放光盘。  
       (让小弟来发挥一下自己的文采吧！）  
  
  首先运行 TRW2000 <-----唔好话我知你无啊！  
  按  CTRL+N  
  
  在对话框中输入：bpx getdrivetypea  
  
 按 F5 回到程序，按确定，这时会被 Trw2000 拦截到。   
  
再按 pmodule  <-------直到程序入口处麻！  
你看到以下代码：  
:00411C00 83C40C                  add esp, 0000000C  
:00411C03 8D4C2418                lea ecx, dword ptr \[esp+18\]  
:00411C07 51                      push ecx  
:00411C08 FFD3                    call ebx  
:00411C0A 83F805                  cmp eax, 00000005 <-----比较 EAX 是否等于 5 如果不相等就跳。  
:00411C0D 7534                    jne 00411C43      <----这个很重要的，万不能让它跳啊！否刚要放光盘的了。(记住与其将这里改为 jz 411c43  倒不如将其改为：jnz 411c56 那么下面的代码就不用改了)  
:00411C0F 6A00                    push 00000000  
:00411C11 6A00                    push 00000000  
:00411C13 6A00                    push 00000000  
:00411C15 6A00                    push 00000000  
:00411C17 6A00                    push 00000000  
:00411C19 8D54247C                lea edx, dword ptr \[esp+7C\]  
:00411C1D 6880000000              push 00000080  
:00411C22 8D442430                lea eax, dword ptr \[esp+30\]  
:00411C26 52                      push edx  
:00411C27 50                      push eax  
:00411C28 FFD5                    call ebp  
:00411C2A 83F801                  cmp eax, 00000001  
:00411C2D 7514                    jne 00411C43    <------不要让它跳到。否则 GMAE OVER！  
:00411C2F 8B4C2410                mov ecx, dword ptr \[esp+10\]  
:00411C33 8D542468                lea edx, dword ptr \[esp+68\]  
:00411C37 51                      push ecx  
:00411C38 52                      push edx  
  
\* Reference To: KERNEL32.lstrcmpA, Ord:02FCh  
                                  |  
:00411C39 FF15FC104F00            Call dword ptr \[004F10FC\]  
:00411C3F 85C0                    test eax, eax  
:00411C41 7413                    je 00411C56    <----这个一定要使它跳。  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00411BEE(C), :00411C0D(C), :00411C2D(C)  
|  
:00411C43 46                      inc esi      <----如果跳到这里再改就难了。  
:00411C44 83FE1A                  cmp esi, 0000001A  
:00411C47 7296                    jb 00411BDF  
:00411C49 5F                      pop edi  
:00411C4A 5E                      pop esi  
:00411C4B 5D                      pop ebp  
:00411C4C 33C0                    xor eax, eax  
:00411C4E 5B                      pop ebx  
:00411C4F 81C4D8000000            add esp, 000000D8  
:00411C55 C3                      ret  
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00411C41(C)  
|  
:00411C56 8D4641                  lea eax, dword ptr \[esi+41\]   《---如果程式来到这里就放心了一半。但还有下文哦！^\_^  
:00411C59 5F                      pop edi  
:00411C5A 5E                      pop esi  
:00411C5B 5D                      pop ebp  
:00411C5C 5B                      pop ebx  
:00411C5D 81C4D8000000            add esp, 000000D8  
:00411C63 C3                      ret   《-----回到主程式到按 F10 一直到,以下代码：  
  
:00411E0F 0F84F7000000            je 00411F0C  <---这个嘛!只要将其改为 je 411e79 就能成功进入游戏了。  
  
  
最后总结一下啦！只要将 411c0d  jnz 411c43  改为：jnz 411c56  
                      411e0f jz 411f0c    改为: jz 411e79  
  
       或者  
   用 HIEW6.40 将大富翁五的执行文件打开找:  
      f8057346a00        改为：f80575476a00  
      85c00f84f7000000     改为：85c00f8464000000  
                                                                      2001 年 7 月 20 写  
                                                                       富强工作室  
                                                                E-MAIL:lfq168@163.com  
                                                                主页:lfq668.yeah.net