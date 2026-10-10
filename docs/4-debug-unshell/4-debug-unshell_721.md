

# 铁甲 1.01

> 原文：[`www.pediy.com/kssd/pediy05/pediy50177.htm`](https://www.pediy.com/kssd/pediy05/pediy50177.htm)

jdpack(铁甲 1.01)的脱壳及破解   
  
这是一个加壳软件,软件加壳后可以检测 trw 及 sice,它的 1.00 版检测到 trw 或 sice 时只是提示,到了 1.01 版,检测到就会出现非法操作.   
未注册版好像没有时间限制,但是给软件加壳后每次运行就会提示"Unregistered JDPack.This file PACKED by Unregistered JDPack1.0\* from http://www.tlzj18.com"   
使用工具:TRW kWDSM   
下载地址(1.00 and 1.01): http://person.longcity.net/home0/flyfancy/jd.rar   
  
  
  
脱壳:   
我最初追踪的是 1.00 版,当时软件还只是提示,我就很轻易的找到了关键   
bpx getversionexa   
g   
断下后   
先 bc(因为以后有几个 getversionexa 的调用,但是与脱壳无关)   
下面会有 2 个 Jz,但是不能让它跳,否则就提示检测到调试器(因为我用的是 TRW)   
输入 r fl z   
之后就是一个 jmp   
F8 继续,后面就很简单了,我只记得快到的时候会有   
popa   
jmp eax   
然后 makepe 即可   
  
  
  
1.01 的脱壳方法也是   
bpx getversionexa   
g   
就到了这里   
0187:0040E250  CALL    NEAR \[EBP+004031B1\]    //停在这里,先 bc   
0187:0040E256  LEA      EBX,\[EBP+00403449\]   
0187:0040E25C  CMP      DWORD \[EBX+10\],BYTE +01   
0187:0040E260  JZ      0040E276            (JUMP)//下 r fl z,跳的话就完了.   
0187:0040E262  CMP      DWORD \[EBX+10\],BYTE +02   
0187:0040E266  JZ      0040E26A   
0187:0040E268  JMP      SHORT 0040E284            //F8 进去   
  
  
0187:0040E284  MOV      EDX,\[EBP+00403441\]      //到了这里   
0187:0040E28A  MOV      ESI,\[EBP+004031D9\]   
0187:0040E290  ADD      ESI,EDX   
0187:0040E292  MOV      EAX,\[ESI+0C\]   
0187:0040E295  OR      EAX,EAX   
0187:0040E297  JZ      NEAR 0040E3EA            (NO JUMP)    //看到这里吗,就 g 40E3EA 吧   
0187:0040E29D  ADD      EAX,EDX   
0187:0040E29F  MOV      \[EBP+004031A5\],EAX   
0187:0040E2A5  MOV      EBX,EAX   
  
  
0187:0040E3EA  MOV      EDX,\[EBP+00403441\]   
0187:0040E3F0  MOV      EAX,\[EBP+004031D5\]   
0187:0040E3F6  ADD      EAX,EDX   
0187:0040E3F8  MOV      \[ESP+1C\],EAX   
0187:0040E3FC  POPA      
0187:0040E3FD  PUSH    EAX   
0187:0040E3FE  RET                                //  到了这里,按 F8 就返回程序的 OEP 处了,直接 makepe 即可   
  
  
  
  
  
破解:   
破解它可难倒了我,我用 kwdsm 反汇编后,半天都摸不着头脑   
  
没有办法只有载入已加壳的 Notepad.exe 跟踪,发现多了一段代码   
0187:0040D3EA 6A30            PUSH    BYTE +30        //就是多了这么一段   
0187:0040D3EC 8D9D4D324000    LEA      EBX,\[EBP+0040324D\]    //   
0187:0040D3F2 53              PUSH    EBX            //   
0187:0040D3F3 8D9D61324000    LEA      EBX,\[EBP+00403261\]    //   
0187:0040D3F9 53              PUSH    EBX            //   
0187:0040D3FA 6A00            PUSH    BYTE +00        //   
0187:0040D3FC FF95D5314000    CALL    NEAR \[EBP+004031D5\]    //这里弹出提示窗口   
0187:0040D402 8B9565344000    MOV      EDX,\[EBP+00403465\]   
0187:0040D408 8B85ED314000    MOV      EAX,\[EBP+004031ED\]   
0187:0040D40E 03C2            ADD      EAX,EDX   
0187:0040D410 8944241C        MOV      \[ESP+1C\],EAX   
0187:0040D414 61              POPA      
0187:0040D415 50              PUSH    EAX   
0187:0040D416 C3              RET      
  
  
再次使用 KWDSM 反汇编,找到   
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00402BE8(C), :00402E57(C)   
|   
:00402FAA 6A30                    push 00000030                //    
:00402FAC 8D9D4D324000            lea ebx, dword ptr \[ebp+0040324D\]   
:00402FB2 53                      push ebx   
:00402FB3 8D9D61324000            lea ebx, dword ptr \[ebp+00403261\]   
:00402FB9 53                      push ebx   
:00402FBA 6A00                    push 00000000   
:00402FBC FF95D5314000            call dword ptr \[ebp+004031D5\]        //这断和那断多的代码不是一样吗?    
:00402FC2 8B9565344000            mov edx, dword ptr \[ebp+00403465\]   
:00402FC8 8B85ED314000            mov eax, dword ptr \[ebp+004031ED\]   
:00402FCE 03C2                    add eax, edx   
:00402FD0 8944241C                mov dword ptr \[esp+1C\], eax   
:00402FD4 61                      popad   
:00402FD5 50                      push eax   
:00402FD6 C3                      ret   
  
  
于是跳转到 402e57   
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00402FA5(U)   
|   
:00402E52 8B460C                  mov eax, dword ptr \[esi+0C\]   
:00402E55 0BC0                    or eax, eax   
:00402E57 0F844D010000            je 00402FAA                //这里改为 0F8465010000 即可    
:00402E5D 03C2                    add eax, edx   
:00402E5F 8985BD314000            mov dword ptr \[ebp+004031BD\], eax   
:00402E65 8BD8                    mov ebx, eax   
:00402E67 50                      push eax   
:00402E68 FF95E4334000            call dword ptr \[ebp+004033E4\]   
:00402E6E 0BC0                    or eax, eax   
:00402E70 7555                    jne 00402EC7   
:00402E72 53                      push ebx   
:00402E73 FF95E8334000            call dword ptr \[ebp+004033E8\]   
:00402E79 0BC0                    or eax, eax   
:00402E7B 754A                    jne 00402EC7   
:00402E7D FFB5BD314000            push dword ptr \[ebp+004031BD\]   
:00402E83 8D9D06334000            lea ebx, dword ptr \[ebp+00403306\]   
:00402E89 53                      push ebx   
:00402E8A 8D9D01354000            lea ebx, dword ptr \[ebp+00403501\]   
:00402E90 53                      push ebx   
:00402E91 FF95D9314000            call dword ptr \[ebp+004031D9\]   
:00402E97 83C40C                  add esp, 0000000C   
:00402E9A 6A30                    push 00000030   
:00402E9C 8D9DA8324000            lea ebx, dword ptr \[ebp+004032A8\]   
:00402EA2 53                      push ebx   
:00402EA3 8D9D01354000            lea ebx, dword ptr \[ebp+00403501\]   
:00402EA9 53                      push ebx   
:00402EAA 6A00                    push 00000000   
:00402EAC FF95D5314000            call dword ptr \[ebp+004031D5\]   
:00402EB2 83BDDD31400001          cmp dword ptr \[ebp+004031DD\], 00000001   
:00402EB9 7408                    je 00402EC3   
:00402EBB 6A00                    push 00000000   
:00402EBD FF95C5314000            call dword ptr \[ebp+004031C5\]   
  
  
我最终都没有发现可以让它变为注册版的办法,以我肤浅的看法就是作者根本没有提供完整版本,因为我用 ResScope 打开时就发现连 Unregistered 都是直接打上去的,哈哈   
  
自己作了个汉化破解版   
http://person.longcity.net/home0/flyfancy/cjd.rar  4dd466d0e1e0cc123afbe504da2462ca