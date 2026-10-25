
# 彩尊圣手黄金版-全国彩票缩水王 V1.50 的脱壳

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-615.htm`](https://www.pediy.com/kssd/pediy07/pediy7-615.htm)

【破解作者】 Anwil  

【作者邮箱】 略  

【作者主页】 略  

【使用工具】 `Ollydbg`、`PEiD`、`LordPE`、`ImportREC 1.6`  

【破解平台】 `WinXP`  

【软件名称】 彩尊圣手黄金版-全国彩票缩水王 `V1.50`  

【下载地址】 `http://www3.skycn.com/soft/21727.html`  

【加壳方式】 `Armadillo 1.xx - 2.xx` -> `Silicon Realms Toolworks`  

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）  

--------------------------------------------------------------------------------

## 【脱壳过程】

虽说是 `Armadillo` 的低版壳，但还是值得研究，也能对了解高版本的 `Armadillo` 壳打下基础，没什么技术含量，海涵海涵！  

设置 `Ollydbg` 忽略所有的异常选项，用 `IsDebug 1.4` 插件去掉 `Ollydbg` 的调试器标志。

```
004C9B19 c>/$   55                     push ebp                          ;  进入 OD 后停在这里  
004C9B1A  |.   8BEC                    mov ebp,esp  
004C9B1C  |.   6A FF                   push -1  
004C9B1E  |.   68 385A4E00             push czssgold.004E5A38  
004C9B23  |.   68 00954C00             push czssgold.004C9500                 ;  SE  handler  installation  
004C9B28  |.   64:A1 00000000          mov eax,dword ptr fs:[0]  
004C9B2E  |.   50                     push eax  
```

### 一、变换双进程，使程序把自己当成子进程运行

下断：`BP OpenMutexA`，`F9` 运行

```
77E62391 k>   55                     push ebp                          ;  断在这里，留意堆栈变化  
77E62392       8BEC                     mov ebp,esp  
77E62394       51                       push ecx  
77E62395       51                       push ecx  
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
0012F59C       004B5F8D    /CALL 到 OpenMutexA 来自 czssgold.004B5F87  
0012F5A0       001F0001    |Access = 1F0001  
0012F5A4       00000000    |Inheritable = FALSE  
0012F5A8       0012FBDC    \MutexName = "ED0::DA61B9CD17"               ;  注意 0012FBDC  
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

`Ctrl+G` 输入 `401000`，来到以下空代码地址

```
00401000       0000                    add byte ptr ds:[eax],al  
00401002       0000                    add byte ptr ds:[eax],al  
00401004       0000                    add byte ptr ds:[eax],al  
00401006       0000                    add byte ptr ds:[eax],al  
00401008       0000                    add byte ptr ds:[eax],al  
0040100A       0000                    add byte ptr ds:[eax],al  
0040100C       0000                    add byte ptr ds:[eax],al  
0040100E       0000                    add byte ptr ds:[eax],al  
00401010       0000                    add byte ptr ds:[eax],al  
00401012       0000                    add byte ptr ds:[eax],al  
```

键入欺骗代码

```
00401000       60                      pushad  
00401001       9C                      pushfd  
00401002       68 DCFB1200             push 12FBDC                        ;  堆栈里看到的值  
00401007       33C0                    xor eax,eax  
00401009       50                      push eax  
0040100A       50                      push eax  
0040100B       E8 B5A6A577             call kernel32.CreateMutexA  
00401010       9D                      popfd  
00401011       61                      popad  
00401012  - E9 7A13A677               jmp kernel32.OpenMutexA  
```

在 `401000` 处，右键--->此处新建 `EIP`，即新建起源，`F9` 运行，再次中断在 `OpenMutexA` 处，取消该断点，然后返回 `401000` 处，撤消刚才键入的所有代码，ok，序幕结束，游戏刚刚开始... ...

### 二、避开 IAT 加密

常听见朋友说 `Armadillo` 的壳有时并不难脱，难就难在修复输入表，这次我就针对这输入表修复来进行一轮恶补，希望大家都能及格，或许大家都能来个漂亮的满分。下面介绍三种避开输入表加密的方法，希望能帮到大家^\_^

#### ⑴、方法一

下断：`He GetModuleHandleA`，`F9` 运行

```
77E5AD86 k>   837C24 04 00             cmp dword ptr ss:[esp+4],0          ;  断在这里，留意堆栈  
77E5AD8B       0F84 37010000           je kernel32.77E5AEC8  
77E5AD91       FF7424 04              push dword ptr ss:[esp+4]  
77E5AD95       E8 F8050000            call kernel32.77E5B392  
```

在这里中断十五次，然后 `Alt+F9` 返回程序，并取消硬件断点

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

```
0012ECC8       77C059FC    /CALL 到 GetModuleHandleA 来自 msvcrt.77C059F6  
0012ECCC       77BE31AC    \pModule = "kernel32.dll"  
0012ECB4       770FB124    /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E  
0012ECB8       771722E4    \pModule = "KERNEL32.DLL"  
0012ECB0       770FB124    /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E  
0012ECB4       771722E4    \pModule = "KERNEL32.DLL"  
0012ED8C       772A8663    /CALL 到 GetModuleHandleA 来自 SHLWAPI.772A865D  
0012ED90       772AF8FC    \pModule = "KERNEL32.DLL"  
0012F540       004B50E3    /CALL 到 GetModuleHandleA 来自 czssgold.004B50DD  
0012F544       00000000    \pModule = NULL  
0012BB20       00CFF65E    /CALL 到 GetModuleHandleA 来自 00CFF658  
0012BB24       00D10B58    \pModule = "kernel32.dll"  
0012BB20       00CFF67B    /CALL 到 GetModuleHandleA 来自 00CFF675  
0012BB24       00D10B58    \pModule = "kernel32.dll"  
0012B898       00CE97CD    /CALL 到 GetModuleHandleA 来自 00CE97C7  
0012B89C       0012B9D4    \pModule = "kernel32.dll"  
0012B898       00CE97CD    /CALL 到 GetModuleHandleA 来自 00CE97C7  
0012B89C       0012B9D4    \pModule = "user32.dll"  
0012B898       00CE97CD    /CALL 到 GetModuleHandleA 来自 00CE97C7  
0012B89C       0012B9D4    \pModule = "MSVBVM60.DLL"  
0012B19C       66001BC5    /CALL 到 GetModuleHandleA 来自 66001BBF  
0012B1A0       6600F4D0    \pModule = "kernel32.dll"  
0012B190       6600281E    /CALL 到 GetModuleHandleA 来自 66002818  
0012B194       6600F4FC    \pModule = "KERNEL32"  
0012B188       660031CE    /CALL 到 GetModuleHandleA 来自 660031C8  
0012B18C       00000000    \pModule = NULL  
0012B898       00CE97CD    /CALL 到 GetModuleHandleA 来自 00CE97C7  
0012B89C       0012B9D4    \pModule = "advapi32.dll"  
0012BB24       00D0416F    /CALL 到 GetModuleHandleA 来自 00D04169              ;  返回的时机，GO！  
0012BB28       00000000    \pModule = NULL  
```

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

```
00D04169       FF15 D8B0D000          call dword ptr ds:[D0B0D8]              ; kernel32.GetModuleHandleA  
00D0416F       3985 C0E4FFFF          cmp dword ptr ss:[ebp-1B40],eax         ; 返回到这里  
00D04175       75 0F                  jnz short 00D04186  
00D04177       C785 BCE4FFFF 98F9D000 mov dword ptr ss:[ebp-1B44],0D0F998  
00D04181       E9 C4000000            jmp 00D0424A  
00D04186       83A5 98E2FFFF 00      and dword ptr ss:[ebp-1D68],0  
00D0418D       C785 94E2FFFF C0FFD000 mov dword ptr ss:[ebp-1D6C],0D0FFC0  
00D04197       EB 1C                  jmp short 00D041B5  
00D04199       8B85 94E2FFFF          mov eax,dword ptr ss:[ebp-1D6C]  
00D0419F       83C0 0C               add eax,0C  
00D041A2       8985 94E2FFFF          mov dword ptr ss:[ebp-1D6C],eax  
00D041A8       8B85 98E2FFFF          mov eax,dword ptr ss:[ebp-1D68]  
00D041AE       40                     inc eax  
00D041AF       8985 98E2FFFF          mov dword ptr ss:[ebp-1D68],eax  
00D041B5       8B85 94E2FFFF          mov eax,dword ptr ss:[ebp-1D6C]  
00D041BB       8338 00               cmp dword ptr ds:[eax],0  
00D041BE       0F84 86000000          je 00D0424A                             ; Magic Jump  
00D041C4       8B85 94E2FFFF          mov eax,dword ptr ss:[ebp-1D6C]  
00D041CA       8B40 08               mov eax,dword ptr ds:[eax+8]  
00D041CD       83E0 01              and eax,1  
```

1、找到 Magic Jump，只要将 `je` 改为 `jmp`，就能得到正确的输入表

```
00D041BE       0F84 86000000          je 00D0424A  
```

改为：

```
00D041BE       EB84 86000000          jmp 00D0424A  
```

2、`Ctrl+F` 在当前位置查找命令：`salc`，当看到 `jmp`、`salc`、`salc` 代码连在一起时，OK，这就是 IAT 解密已经完成的地方。

```
00D04883       59                      pop ecx  
00D04884       EB 03                   jmp short 00D04889                  ; 此处下断  
00D04886       D6                      salc  
00D04887       D6                      salc  
```

程序在下面会依据原先的代码进行解码，如没有修改原代码，则解码正确。而直接修改 Magic Jump 后改变了原先的代码，导致解码不正确而异常出错！所以我们在 `00D04884` 断下后，应该恢复原先 Magic Jump 的代码，返回原来的地方，则 `00D041BE` 处，右键－>“撤销选择”则可。现在解码就不会出错了。



## 方法二

下断：`He GetModuleHandleA`，`F9` 运行

```
77E5AD86 k>   837C24 04 00                   cmp dword ptr ss:[esp+4],0                 ;  断在这里，留意堆栈  
77E5AD8B       0F84 37010000                 je kernel32.77E5AEC8  
77E5AD91       FF7424 04                     push dword ptr ss:[esp+4]  
77E5AD95       E8 F8050000                  call kernel32.77E5B392  
```

在这里中断八次，然后 `Alt+F9` 返回程序，并取消硬件断点

∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮

```
0012ECC8     77C059FC    /CALL 到 GetModuleHandleA 来自 msvcrt.77C059F6  
0012ECCC     77BE31AC    \pModule = "kernel32.dll"  
0012ECD0     77C2CA20    msvcrt.77C2CA20  
0012ECB4     770FB124    /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E  
0012ECB8     771722E4    \pModule = "KERNEL32.DLL"  
0012ECBC     770FAD56    返回到 OLEAUT32.770FAD56 来自 OLEAUT32.770FB119  
0012ECB0     770FB124    /CALL 到 GetModuleHandleA 来自 OLEAUT32.770FB11E  
0012ECB4     771722E4    \pModule = "KERNEL32.DLL"  
0012ECB8     770FADAC    返回到 OLEAUT32.770FADAC 来自 OLEAUT32.770FB119  
0012ED8C     772A8663    /CALL 到 GetModuleHandleA 来自 SHLWAPI.772A865D  
0012ED90     772AF8FC    \pModule = "KERNEL32.DLL"  
0012ED94     00000000  
0012F540     004B50E3    /CALL 到 GetModuleHandleA 来自 czssgold.004B50DD  
0012F544     00000000    \pModule = NULL  
0012F548     00000000  
0012BB20     00CFF65E    /CALL 到 GetModuleHandleA 来自 00CFF658  
0012BB24     00D10B58    \pModule = "kernel32.dll"  
0012BB28     00D11BB4    ASCII "VirtualAlloc"  
0012BB20     00CFF67B    /CALL 到 GetModuleHandleA 来自 00CFF675  
0012BB24     00D10B58    \pModule = "kernel32.dll"  
0012BB28     00D11BA8    ASCII "VirtualFree"  
0012B898     00CE97CD    /CALL 到 GetModuleHandleA 来自 00CE97C7                 ;  返回的时机，GO！  
0012B89C     0012B9D4    \pModule = "kernel32.dll"  
0012B8A0     0012EB14  
```

∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮∮

```
00CE97C7     FF15 D8B0D000                 call dword ptr ds:[D0B0D8]              ; kernel32.GetModuleHandleA  
00CE97CD     8B0D A04CD100                 mov ecx,dword ptr ds:[D14CA0]            ; 返回到这里  
00CE97D3     89040E                        mov dword ptr ds:[esi+ecx],eax  
00CE97D6     A1 A04CD100                   mov eax,dword ptr ds:[D14CA0]  
00CE97DB     393C06                        cmp dword ptr ds:[esi+eax],edi  
00CE97DE     75 16                         jnz short 00CE97F6  
00CE97E0     8D85 B4FEFFFF                 lea eax,dword ptr ss:[ebp-14C]  
00CE97E6     50                            push eax  
00CE97E7     FF15 E0B0D000                 call dword ptr ds:[D0B0E0]              ; kernel32.LoadLibraryA  
00CE97ED     8B0D A04CD100                 mov ecx,dword ptr ds:[D14CA0]  
00CE97F3     89040E                        mov dword ptr ds:[esi+ecx],eax  
00CE97F6     A1 A04CD100                   mov eax,dword ptr ds:[D14CA0]  
00CE97FB     393C06                        cmp dword ptr ds:[esi+eax],edi  
00CE97FE     0F84 AD000000                 je 00CE98B1                             ; Magic Jump  
00CE9804     33C9                          xor ecx,ecx  
00CE9806     8B03                          mov eax,dword ptr ds:[ebx]  
```

直接改 `00CE97FE` `jmp` `00CE98B1`，此时修改代码也非常的容易、方便，更不需要在 IAT 处理结束后再还原代码了。

## 方法三

嘿嘿，比较多余，充其量只是玩玩，就算是多一种认识 `Armadillo` 壳脱法的机会吧。这也是继方法二之后的事了，废话少说，`GO`！

来到上面的 `Magic Jump` 那里……

```
00CE97C7     FF15 D8B0D000                 call dword ptr ds:[D0B0D8]              ; kernel32.GetModuleHandleA  
00CE97CD     8B0D A04CD100                 mov ecx,dword ptr ds:[D14CA0]           
00CE97D3     89040E                        mov dword ptr ds:[esi+ecx],eax  
00CE97D6     A1 A04CD100                   mov eax,dword ptr ds:[D14CA0]  
00CE97DB     393C06                        cmp dword ptr ds:[esi+eax],edi  
00CE97DE     75 16                         jnz short 00CE97F6  
00CE97E0     8D85 B4FEFFFF                 lea eax,dword ptr ss:[ebp-14C]  
00CE97E6     50                            push eax  
00CE97E7     FF15 E0B0D000                 call dword ptr ds:[D0B0E0]              ; kernel32.LoadLibraryA  
00CE97ED     8B0D A04CD100                 mov ecx,dword ptr ds:[D14CA0]  
00CE97F3     89040E                        mov dword ptr ds:[esi+ecx],eax  
00CE97F6     A1 A04CD100                   mov eax,dword ptr ds:[D14CA0]  
00CE97FB     393C06                        cmp dword ptr ds:[esi+eax],edi  
00CE97FE     0F84 AD000000                 je 00CE98B1                             ; Magic Jump，在这里中断 3 次  
00CE9804     33C9                          xor ecx,ecx  
00CE9806     8B03                          mov eax,dword ptr ds:[ebx]  
```

在 `00CE97FE` 处，右键--->断点--->"硬断点，执行时"，这次是不改 `je` 为 `jmp`，而是每次断下后改标志 `Z=1`，改 `Z=1`，使其 `JMP`,就能得到未被破坏的输入表了.

其实 `Armadillo` 壳的 `Magic Jump` 有时不止一个，就看你怎么利用了... ...

# 三、寻找 OEP

因为上面有三种方法,所以到底 OEP 也有几种方法,别心急,`FOLLOW ME`!

### ㈠、这是继方法一之后的事

下断 `bp GetCurrentThreadId`，`F9` 运行

```
77E5A7DF k>  64:A1 18000000                mov eax,dword ptr fs:[18]               ; 断在这里，留意堆栈  
77E5A7E5      8B40 24                     mov eax,dword ptr ds:[eax+24]  
77E5A7E8      C3                          retn  
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

```
0012F594     00D07245    /CALL 到 GetCurrentThreadId 来自 00D0723F                ;  返回的时机，GO！  
0012F598     0012FF2C  
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

在这里中断一次，然后 `Alt+F9` 返回程序

```
00D07245     A3 8091D100                   mov dword ptr ds:[D19180],eax           ; 停在这里  
00D0724A     E8 6E28FEFF                   call 00CE9ABD  
00D0724F     6A 00                         push 0  
00D07251     E8 B981FEFF                   call 00CEF40F  
00D07256     6A 00                         push 0  
00D07258     C705 7810D100 041CD100        mov dword ptr ds:[D11078],0D11C04       ; ASCII "RC"  
00D07262     E8 CB22FEFF                   call 00CE9532  
00D07267     59                            pop ecx  
00D07268     59                            pop ecx  
00D07269     E8 310AFFFF                   call 00CF7C9F  
00D0726E     8BF8                          mov edi,eax  
00D07270     A1 6890D100                   mov eax,dword ptr ds:[D19068]  
00D07275     8B48 54                       mov ecx,dword ptr ds:[eax+54]  
00D07278     3348 50                       xor ecx,dword ptr ds:[eax+50]  
00D0727B     3348 34                       xor ecx,dword ptr ds:[eax+34]  
00D0727E     03F9                          add edi,ecx  
00D07280     8B0E                          mov ecx,dword ptr ds:[esi]  
00D07282     85C9                          test ecx,ecx  
00D07284     75 2F                         jnz short 00D072B5                      ; 跳  
00D07286     8B78 50                       mov edi,dword ptr ds:[eax+50]  
00D07289     E8 110AFFFF                   call 00CF7C9F  
00D0728E     8B0D 6890D100                 mov ecx,dword ptr ds:[D19068]           ; czssgold.004E02A0  
00D07294     FF76 14                       push dword ptr ds:[esi+14]  
00D07297     8B51 54                       mov edx,dword ptr ds:[ecx+54]  
00D0729A     FF76 10                       push dword ptr ds:[esi+10]  
00D0729D     3351 34                       xor edx,dword ptr ds:[ecx+34]  
00D072A0     FF76 0C                       push dword ptr ds:[esi+C]  
00D072A3     33D7                          xor edx,edi  
00D072A5     03C2                          add eax,edx  
00D072A7     8B51 30                       mov edx,dword ptr ds:[ecx+30]  
00D072AA     3351 20                       xor edx,dword ptr ds:[ecx+20]  
00D072AD     33D7                          xor edx,edi  
00D072AF     2BC2                          sub eax,edx  
00D072B1     FFD0                          call eax  
00D072B3     EB 25                         jmp short 00D072DA  
00D072B5     83F9 01                       cmp ecx,1  
00D072B8     75 22                         jnz short 00D072DC  
```



## ㈡、这是继方法二之后的事

下断 `bp GetCurrentThreadId`，`F9` 运行

```
77E5A7DF k>   64:A1 18000000             mov eax,dword ptr fs:[18]                ; 断在这里，留意堆栈
77E5A7E5      8B40 24                   mov eax,dword ptr ds:[eax+24]
77E5A7E8      C3                        retn
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

```
0012B198     66001E3A   /CALL 到 GetCurrentThreadId 来自 66001E34
0012B19C     00000001

0012B178     66003505   /CALL 到 GetCurrentThreadId 来自 660034FF
0012B17C     00000000

0012B3F4     761736A1   /CALL 到 GetCurrentThreadId 来自 WININET.7617369B
0012B3F8     00000000

0012F594     00D07245   /CALL 到 GetCurrentThreadId 来自 00D0723F             ;  返回的时机，GO！
0012F598     0012FF2C
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

在这里中断四次，然后 `Alt+F9` 返回程序

```
00D07245     A3 8091D100                mov dword ptr ds:[D19180],eax        ; 停在这里
00D0724A     E8 6E28FEFF                call 00CE9ABD
00D0724F     6A 00                      push 0
00D07251     E8 B981FEFF                call 00CEF40F
00D07256     6A 00                      push 0
00D07258     C705 7810D100 041CD100    mov dword ptr ds:[D11078],0D11C04    ; ASCII "RC"
00D07262     E8 CB22FEFF                call 00CE9532
00D07267     59                         pop ecx
00D07268     59                         pop ecx
00D07269     E8 310AFFFF                call 00CF7C9F
00D0726E     8BF8                       mov edi,eax
00D07270     A1 6890D100                mov eax,dword ptr ds:[D19068]
00D07275     8B48 54                    mov ecx,dword ptr ds:[eax+54]
00D07278     3348 50                    xor ecx,dword ptr ds:[eax+50]
00D0727B     3348 34                    xor ecx,dword ptr ds:[eax+34]
00D0727E     03F9                       add edi,ecx
00D07280     8B0E                       mov ecx,dword ptr ds:[esi]
00D07282     85C9                       test ecx,ecx
00D07284     75 2F                      jnz short 00D072B5
00D07286     8B78 50                    mov edi,dword ptr ds:[eax+50]
00D07289     E8 110AFFFF                call 00CF7C9F
00D0728E     8B0D 6890D100              mov ecx,dword ptr ds:[D19068]        ; czssgold.004E02A0
00D07294     FF76 14                    push dword ptr ds:[esi+14]
00D07297     8B51 54                    mov edx,dword ptr ds:[ecx+54]
00D0729A     FF76 10                    push dword ptr ds:[esi+10]
00D0729D     3351 34                    xor edx,dword ptr ds:[ecx+34]
00D072A0     FF76 0C                    push dword ptr ds:[esi+C]
00D072A3     33D7                       xor edx,edi
00D072A5     03C2                       add eax,edx
00D072A7     8B51 30                    mov edx,dword ptr ds:[ecx+30]
00D072AA     3351 20                    xor edx,dword ptr ds:[ecx+20]
00D072AD     33D7                       xor edx,edi
00D072AF     2BC2                       sub eax,edx
00D072B1     FFD0                       call eax
00D072B3     EB 25                      jmp short 00D072DA
00D072B5     83F9 01                    cmp ecx,1
00D072B8     75 22                      jnz short 00D072DC
00D072BA     FF76 04                    push dword ptr ds:[esi+4]
00D072BD     FF76 08                    push dword ptr ds:[esi+8]
00D072C0     6A 00                      push 0
00D072C2     E8 D809FFFF                call 00CF7C9F
00D072C7     50                         push eax
00D072C8     A1 6890D100                mov eax,dword ptr ds:[D19068]
00D072CD     8B48 50                    mov ecx,dword ptr ds:[eax+50]
00D072D0     3348 30                    xor ecx,dword ptr ds:[eax+30]
00D072D3     3348 20                    xor ecx,dword ptr ds:[eax+20]
00D072D6     2BF9                       sub edi,ecx
00D072D8     FFD7                       call edi                               ; 根据地在现，F7 进入吧
00D072DA     8BD8                       mov ebx,eax
00D072DC     5F                         pop edi
00D072DD     8BC3                       mov eax,ebx
00D072DF     5E                         pop esi
00D072E0     5B                         pop ebx
00D072E1     C3                         retn
```

## ㈢、这是继方法三之后的事

下断 `bp GetCurrentThreadId`，`F9` 运行

```
77E5A7DF k>   64:A1 18000000             mov eax,dword ptr fs:[18]                ; 停在这里
77E5A7E5      8B40 24                    mov eax,dword ptr ds:[eax+24]
77E5A7E8      C3                         retn
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

```
0012B3F4     761736A1   /CALL 到 GetCurrentThreadId 来自 WININET.7617369B
0012B3F8     00000000

0012F594     00D07245   /CALL 到 GetCurrentThreadId 来自 00D0723F                ;  返回的时机，GO！
0012F598     0012FF2C
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※

在这里中断二次，然后 `Alt+F9` 返回程序，之后怎么去到 OEP 可以参考以上两例，我们直接进入下一个环节。

## ㈣、用内存断点大法直抵 OEP

以上三种方法都可以直接用内存断点大法直抵 OEP，这个方法确实很实在，也很好用，省了很多不必要的麻烦。

`Alt+M` 查看内存，在 `401000` 开始的段上 下内存访问断点，`F9` 运行

```
0043DC0D     6A 60                      push 60                                  ;  红色代码，呵呵，直达船，爽！
0043DC0F     68 40A04800               push czssgold.0048A040
0043DC14     E8 BB0C0000               call czssgold.0043E8D4
0043DC19     BF 94000000               mov edi,94
0043DC1E     8BC7                      mov eax,edi
0043DC20     E8 DBF0FFFF               call czssgold.0043CD00
0043DC25     8965 E8                   mov dword ptr ss:[ebp-18],esp
0043DC28     8BF4                      mov esi,esp
0043DC2A     893E                      mov dword ptr ds:[esi],edi
0043DC2C     56                        push esi
0043DC2D     FF15 04944700             call dword ptr ds:[479404]               ;  kernel32.GetVersionExA
```

只要来到这里，就代表胜利之神已降临在我们身边了，继续努力，`GO`！

## 四、dumper、修复输入表和优化

这个很容易了，不用我讲大家都会做得比我好，可省我很多口水 `:)`

```
0043DC0D     6A 60                      push 60                                  ;  在这儿用 LordPE 纠正 ImageSize
                                                                     后完全 DUMP 这个进程
0043DC0F     68 40A04800               push czssgold.0048A040
0043DC14     E8 BB0C0000               call czssgold.0043E8D4
0043DC19     BF 94000000               mov edi,94
0043DC1E     8BC7                      mov eax,edi
0043DC20     E8 DBF0FFFF               call czssgold.0043CD00
0043DC25     8965 E8                   mov dword ptr ss:[ebp-18],esp
0043DC28     8BF4                      mov esi,esp
```



```
0043DC2A  893E                    mov dword ptr ds:[esi],edi  
0043DC2C  56                      push esi  
0043DC2D  FF15 04944700           call dword ptr ds:[479404]               ; kernel32.GetVersionExA  
0043DC33  8B4E 10                 mov ecx,dword ptr ds:[esi+10]  
0043DC36  890D 60DD4900           mov dword ptr ds:[49DD60],ecx  
0043DC3C  8B46 04                 mov eax,dword ptr ds:[esi+4]  
0043DC3F  A3 6CDD4900             mov dword ptr ds:[49DD6C],eax  
0043DC44  8B56 08                 mov edx,dword ptr ds:[esi+8]  
0043DC47  8915 70DD4900           mov dword ptr ds:[49DD70],edx  
0043DC4D  8B76 0C                 mov esi,dword ptr ds:[esi+C]  
```

运行`ImportREC 1.6`，选择这个进程。把`OEP`改为`0003DC0D`，点`IT AutoSearch`，有少数无用的函数，点`Show Invalid`全部选中，然后右键-->`Trace Level1(Disasm)`修复大部分，然后剩下的都是无用的东东哦，`CUT`掉无效函数，`FixDump`，正常运行！用`LordPE`删除`text1`和其下的`adata`、`data1`、`pdata`共4个区段，然后再用`LordPE`重建`PE`，程序就精简了不少。

### 后记
其实`Armadillo`的壳还有两个地方值得关注一下：

1. 用以上方法无法正确找到输入表加密的切入点；
2. 如果加了`Armadillo`壳的软件已经过期，那该怎么处理。

如果找到合适的机会我会在再来详细说明一下。本来这文章昨天就应该做好的，无奈于`BOSS`，和他的机会朋友居然来“暗黑破坏神”，加上家里的电脑突然爆毙，所以拖到今天，现在又要去和`BOSS`们联机去玩暗黑了，呵呵，闪咯... ...

---

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
