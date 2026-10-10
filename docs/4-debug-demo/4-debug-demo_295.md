

# WinZip8.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50137.htm`](https://www.pediy.com/kssd/pediy05/pediy50137.htm)

消除 WinZip8.1 版本过期提示框   
  
    到看雪论坛,几个月了,没有发过一篇有关调试软件的帖子,毕竟本人太懒了,跟完程序,从不做笔记.   
今天跟了一个这个程序,发现非常非常简单,几分钟就搞定了,时间充裕,灌灌水吧.   
  
调试人  :zmworm   
调试对象:WinZip 8.1 BETA(4286)   
软件来源:电脑报配套光盘 2001   
调试工具:SoftICE 4.25(NuMega DriverStudio 2.5 集成)   
修改工具:U-Edit   
调试难度:极易   
适应读者:初学者   
  
  
    关于 WinZip 的注册机的介绍已经很多了,我的 WinZip 也已经是注册版本了.可是自打今年某月,   
每次打开 winzip 就告诉我版本太旧了,要到网站更新.我懒的上网更新,毕竟 WinZip 的个头不小,   
而我的猫还在 56Kbps 上奔跑.   
    我已厌倦了点那个确定键.于是对 WinZip 动手术了:   
  
1\. 用 SoftICE 加载 Win98.   
2\. Ctrl-D 开启 SoftICE 窗口   
3\. 设断点 bpx GetLocalTime.因为对话框与时间有关,所以拦这个 API.   
与时间有关的 API: GetSystemTime GetLocalTime GetFileTime 等等.   
4.运行 WinZip,SoftICE 拦到断点,按 F5,又被 SoftICE 拦到   
5.按两下 F12,到 00403F1E 处,GetLocalTime 就在这个 CALL 里   
  
下面是我对这段代码的注释:   
  
\* Referenced by a CALL at Address:   
|:0043D17B    
|   
:00403F09 56                      push esi   
:00403F0A 6A1E                    push 0000001E                        ;d=30 日   
:00403F0C 6A01                    push 00000001                        ;m=1 月   
:00403F0E 68D2070000              push 0000007D2                      ;y=2002 年   
:00403F13 E859960400              call 0044D571                        ;ax\=\[(y-1980)\*16+m\]\*32+d   
:00403F18 83C40C                  add esp, 0000000C   
:00403F1B 668BF0                  mov si, ax                          ;si 为规定日期   
\* bpx local 第二次拦到,再 F12 两次后 到这里;   
                                  |   
:00403F1E E8BDFFFFFF              call 00403EE0   
  
\* 比较 ax 与 si,就是比较当前日期是否到了规定日期 2002.1.30 日   
                                  |   
:00403F23 663BC6                  cmp ax, si                          ;ax 为当前日期   
:00403F26 7653                    jbe 00403F7B                        ;-)一定要跳走呀,否则你看看下面 00403F5B 处的几行字   
  
;--------------不要到这里来-------------   
:00403F28 FF35ACC64A00            push dword ptr \[004AC6AC\]   
  
\* Possible StringData Ref from Data Obj ->"70X\_BETA1"   
                                  |   
:00403F2E BE10E34B00              mov esi, 004BE310   
:00403F33 C60589BF4C0001          mov byte ptr \[004CBF89\], 01   
:00403F3A 6A01                    push 00000001   
:00403F3C 56                      push esi   
:00403F3D FF35B0C64A00            push dword ptr \[004AC6B0\]   
:00403F43 E8D16C0500              call 0045AC19   
:00403F48 83C410                  add esp, 00000010   
:00403F4B 85C0                    test eax, eax   
:00403F4D 742C                    je 00403F7B   
:00403F4F FF35ACC64A00            push dword ptr \[004AC6AC\]   
:00403F55 56                      push esi   
:00403F56 BE44C54C00              mov esi, 004CC544   
  
\* Possible StringData Ref from Data Obj ->"This PRE-RELEASE BETA version "   
                                        ->"of WinZip is quite old.   
  
This "   
                                        ->"version will continue to run, "   
                                        ->"but will display this message "   
                                        ->"when you start it.  You are strongly "   
                                        ->"encouraged to download the current "   
                                        ->"version from the WinZip home page "   
                                        ->"at http://www.winzip.com."   
                                  |   
:00403F5B 68E8DC4B00              push 004BDCE8   
:00403F60 56                      push esi   
  
\* Reference To: USER32.wsprintfA, Ord:02B3h   
                                  |   
:00403F61 FF158CC44A00            Call dword ptr \[004AC48C\]            ;-(弹出对话框   
:00403F67 56                      push esi   
:00403F68 FF3538C14C00            push dword ptr \[004CC138\]   
:00403F6E 68DD000000              push 000000DD   
:00403F73 E85F840400              call 0044C3D7   
:00403F78 83C41C                  add esp, 0000001C   
  
  
\*------------------到这里就不显对话框了----------------------   
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00403F26(C), :00403F4D(C)   
|   
:00403F7B 5E                      pop esi   
:00403F7C C3                      ret   
  
  
了解了这段代码的含义,我们就可以做手术了.   
手术方案 1:永远跳   
将   
:00403F26 7653                    jbe 00403F7B   
改为   
:00403F26 EB53                    jmp 00403F7B   
  
用 U-Edit 打开 WinZip32.exe.在 3F26 处   
将    76   
改为  EB   
  
  
手术方案 2:改变规定日期   
将   
:00403F0E 68D2070000              push 0000007D2                      ;y=2002 年   
改为   
:00403F0E 6834080000              push 000000834                      ;y=2100 年   
  
用 U-Edit 打开 WinZip32.exe.在 3F0E 处   
将    D2 07   
改为  34 08   
  
我的 WinZip 永远不会过期了!!  4dd466d0e1e0cc123afbe504da2462ca