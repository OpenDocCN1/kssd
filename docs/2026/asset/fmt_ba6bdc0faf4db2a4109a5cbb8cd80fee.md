
# [原创]Yoda's Crypter v1.2脱壳分析

## 前言

时断时续的学习破解，这是我在本坛贴出的第二篇学习笔记，比前一篇稍有了点进步，与刚学破解的朋友们交流，大牛们看了别笑话水平太低，给我指点就好了![](https://bbs.kanxue.com/view/img/face/12.gif)

## 软件

加了yoda's cryptor 1.2壳的win98记事本程序，由天草老师教程中提供。

## 工具

OD、PEiD、lordPE、ImportREC

用PEiD查一下，是yoda's cryptor 1.2壳，下面OD载入，开始分析。

首先是壳引导部分，功能是解密壳主体部分代码，解密完成后就将控制权交给主体代码。

```
0040D060 Y>  60                pushad                                   ; 程序入口，载入时停在此处
0040D061     E8 00000000       call Yoda's_C.0040D066
0040D066     5D                pop ebp                                  ; 这两行是壳代码自定位
0040D067     81ED F31D4000     sub ebp,Yoda's_C.00401DF3                ; 计算偏移量，用于取加壳时保存的信息的重定位
0040D06D     B9 7B090000       mov ecx,97B                              ; 要解密代码的字节数为97Bh
0040D072     8DBD 3B1E4000     lea edi,dword ptr ss:[ebp+401E3B]        ; 被解密代码的起始位置，解密完成全就被执行
0040D078     8BF7              mov esi,edi
0040D07A  /  AC                lods byte ptr ds:[esi]                   ; 逐字节取出进行解密，以下就是解密代码
0040D07B  |  2AC1              sub al,cl
0040D07D  |  EB 01             jmp short Yoda's_C.0040D080
0040D07F  |  C2 348F           retn 8F34                                ; C2花指令，可将C2改为90（nop）。以下几处同此。
0040D080  |  34 8F             xor al,8F
0040D082  |  F9                stc
0040D083  |  02C1              add al,cl
0040D085  |  F8                clc
0040D086  |  34 AE             xor al,0AE
0040D088  |  2C E9             sub al,0E9
0040D08A  |  FEC8              dec al
0040D08C  |  02C1              add al,cl
0040D08E  |  FEC8              dec al
0040D090  |  04 CD             add al,0CD
0040D092  |  90                nop
0040D093  |  EB 01             jmp short Yoda's_C.0040D096
0040D095  |  C2 2AC1           retn 0C12A                                ; 又是C2花指令
0040D096  |  2AC1              sub al,cl
0040D098  |  C0C8 5E           ror al,5E
0040D09B  |  FEC8              dec al
0040D09D  |  2AC1              sub al,cl
0040D09F  |  04 7D             add al,7D
0040D0A1  |  EB 01             jmp short Yoda's_C.0040D0A4
0040D0A3  |  C2 2AC1           retn 0C12A                                ; 依然是C2花指令
0040D0A4  |  2AC1              sub al,cl
0040D0A6  |  34 58             xor al,58
0040D0A8  |  C0C0 B1           rol al,0B1
0040D0AB  |  AA                stos byte ptr es:[edi]
0040D0AC  \^ E2 CC             loopd short Yoda's_C.0040D07A
```

因为有代码自解密，所以会给静态调试带来干扰，但分析上面所有解密方法，可以看出都是对 `al` 进行运算，另两个参数就是 `cl`（剩余字节数的）和 `edi`（代码地址），如果用IDA来调试的话，也很容易能写出脚本函数来解密这段代码。解密完成后，我们在数据窗口看到被解密的末尾赫然出现了 `IsDebuggerPresent`，这里埋下伏笔。

以下就是解密后的代码，位置是紧接上面的代码之后，解密完成后就开始执行之。

```
0040D0AE     8B4424 20                mov eax,dword ptr ss:[esp+20]        ; kernel32.7C817077
0040D0B2     40                       inc eax                              ; 程序出口点--kernel32.ExitThread
0040D0B3     78 0A                    js short Yoda's_C.0040D0BF
0040D0B5     C785 78254000 01000000   mov dword ptr ss:[ebp+402578],1      ; 设置标志，后面会用到。
0040D0BF     8D85 ED1D4000            lea eax,dword ptr ss:[ebp+401DED]    ; 地址=0040D060，程序入口点
0040D0C5     B9 2A060000              mov ecx,62A                          ; 自校验的代码字节数
0040D0CA     E8 41020000              call Yoda's_C.0040D310               ; 壳代码自校验

        {

        0040D310     8BF8                     mov edi,eax                  ; 从入口点0040D060起62Ah字节纳入校验
        0040D312     33C0                     xor eax,eax
        0040D314     33DB                     xor ebx,ebx
        0040D316     33D2                     xor edx,edx
        0040D318     8A07                     mov al,byte ptr ds:[edi]
        0040D31A     F7E2                     mul edx
        0040D31C     03D8                     add ebx,eax
        0040D31E     42                       inc edx
        0040D31F     47                       inc edi
        0040D320   ^ E2 F6                    loopd short Yoda's_C.0040D318

        }
```

这个函数校验从壳入口起62A字节，实际上第1字节不影响结果，因这它乘0还是=0。

```
0040D0CF     8985 74254000            mov dword ptr ss:[ebp+402574],eax    ; 保存校验和（0002494C），后面会用到。
```

我们可以看到 `[ebp+402574]` 上下是一张信息表，其中存放了加壳前原程序的一些数据信息，如 `[0040D7DB]` 处就是存放的原程序 `OEP` －－ `004010CC`，以及 `[0040D7DB]` 处的 `00400000`，加壳后的程序运行时用这些数据进行一系列处理。

```
0040D0D5     8B85 6C254000            mov eax,dword ptr ss:[ebp+40256C]    ; ss:[0040D7DF]=0000003C
0040D0DB     83E0 01                  and eax,1
0040D0DE     74 40                    je short Yoda's_C.0040D120           ; 偶数，跳
0040D120     8B85 64254000            mov eax,dword ptr ss:[ebp+402564]    ; 取ImageBaseAddress
0040D126     0340 3C                  add eax,dword ptr ds:[eax+3C]        ; 定位PE Header
0040D129     05 80000000              add eax,80                           ; 取输入表数据目录
0040D12E     8B08                     mov ecx,dword ptr ds:[eax]           ; 取输入表RVA（加壳时已经被改指向壳中的新位置）
0040D130     038D 64254000            add ecx,dword ptr ss:[ebp+402564]    ; +ImageBaseAddress定位输入表（位于壳中，只有1个IID）
0040D136     83C1 10                  add ecx,10                           ; FirstThunk
0040D139     8B01                     mov eax,dword ptr ds:[ecx]           ;
0040D13B     0385 64254000            add eax,dword ptr ss:[ebp+402564]    ; 函数指针
0040D141     8B18                     mov ebx,dword ptr ds:[eax]           ; 函数kernel32.LoadLibraryA地址
0040D143     899D F0264000            mov dword ptr ss:[ebp+4026F0],ebx    ; 保存LoadLibraryA函数地址
0040D149     83C0 04                  add eax,4                            ; 下一个函数
0040D14C     8B18                     mov ebx,dword ptr ds:[eax]           ; kernel32.GetProcAddress
0040D14E     899D F4264000            mov dword ptr ss:[ebp+4026F4],ebx    ; 保存GetProcAddress函数地址
0040D154     8D85 F8264000            lea eax,dword ptr ss:[ebp+4026F8]    ; ASCII "Kernel32.dll"
0040D15A     50                       push eax
0040D15B     FF95 F0264000            call dword ptr ss:[ebp+4026F0]       ; 动态调入Kernel32.dll
0040D161     8BF0                     mov esi,eax                          ; Kernel32.dll句柄
0040D163     8985 05274000            mov dword ptr ss:[ebp+402705],eax    ; 保存句柄
0040D169     8D85 09274000            lea eax,dword ptr ss:[ebp+402709]    ; ASCII "GetModuleHandleA"
0040D16F     E8 96000000              call Yoda's_C.0040D20A               ; 简单封装的GetProcAddress函数。获取kernel32.GetModuleHandleA地址

        {

        0040D20A     50                       push eax
        0040D20B     56                       push esi
        0040D20C     FF95 F4264000            call dword ptr ss:[ebp+4026F4] ; kernel32.GetProcAddress

        }

0040D174     8985 1A274000            mov dword ptr ss:[ebp+40271A],eax    ; 保存函数地址
0040D17A     8D85 1E274000            lea eax,dword ptr ss:[ebp+40271E]    ; ASCII "VirtualProtect"
0040D180     E8 85000000              call Yoda's_C.0040D20A               ; 获取kernel32.VirtualProtect地址
0040D185     8985 2D274000            mov dword ptr ss:[ebp+40272D],eax    ; 保存函数地址
0040D18B     8D85 31274000            lea eax,dword ptr ss:[ebp+402731]
0040D191     E8 74000000              call Yoda's_C.0040D20A               ; 获取kernel32.GetModuleFileNameA地址
0040D196     8985 44274000            mov dword ptr ss:[ebp+402744],eax    ; 保存函数地址
0040D19C     8D85 48274000            lea eax,dword ptr ss:[ebp+402748]
0040D1A2     E8 63000000              call Yoda's_C.0040D20A               ; 获取kernel32.CreateFileA地址
0040D1A7     8985 54274000            mov dword ptr ss:[ebp+402754],eax    ; 保存函数地址
0040D1AD     8D85 58274000            lea eax,dword ptr ss:[ebp+402758]
0040D1B3     E8 52000000              call Yoda's_C.0040D20A               ; 获取kernel32.GlobalAlloc地址
0040D1B8     8985 64274000            mov dword ptr ss:[ebp+402764],eax    ; 保存函数地址
0040D1BE     8D85 68274000            lea eax,dword ptr ss:[ebp+402768]
0040D1C4     E8 41000000              call Yoda's_C.0040D20A               ; 获取kernel32.GlobalFree地址
0040D1C9     8985 73274000            mov dword ptr ss:[ebp+402773],eax    ; 保存函数地址
0040D1CF     8D85 77274000            lea eax,dword ptr ss:[ebp+402777]
```
