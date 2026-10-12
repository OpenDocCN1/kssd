
# 幻影之旅

> 原文：[`www.pediy.com/kssd/pediy06/pediy6729.htm`](https://www.pediy.com/kssd/pediy06/pediy6729.htm)

【实例下载】点击此处下载（或鼠标右键另存为）

【原文链接】[`bbs.pediy.com/showthread.php?s=&threadid=359`](http://bbs.pediy.com/showthread.php?s=&threadid=359)

\[part 1\]

幻影之旅

by forgot/uS

转载保持完整![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

```
00420000  > /EB 20          jmp     short Try.00420022  
00420002   |0000           add     byte ptr ds:[eax], al  
00420004   |40            inc     eax  
00420005   |0000          add     byte ptr ds:[eax], al  
00420007   |0040 00       add     byte ptr ds:[eax], al  
0042000A   |0000          add     byte ptr ds:[eax], al  
0042000C   |0000          add     byte ptr ds:[eax], al  
0042000E   |0000          add     byte ptr ds:[eax], al  
00420010   |0000          add     byte ptr ds:[eax], al  
00420012   |0200          add     al, byte ptr ds:[eax]  
00420014   |0B00          or      eax, dword ptr ds:[eax]  
00420016   |0000          add     byte ptr ds:[eax], al  
00420018   |0230          add     dh, byte ptr ds:[eax]  
0042001A   |0000          add     byte ptr ds:[eax], al  
0042001C   |0000          add     byte ptr ds:[eax], al  
0042001E   |0000          add     byte ptr ds:[eax], al  
00420020   |0000          add     byte ptr ds:[eax], al  
00420022   \9C           pushfd                           ; 保存标志  
00420023    55           push    ebp                       ; 保存寄存器  
00420024    57           push    edi  
00420025    56           push    esi  
00420026    52           push    edx  
00420027    51           push    ecx  
00420028    53           push    ebx  
00420029    9C           pushfd  
0042002A    E8 00000000  call    Try.0042002F              ; 取 delta  
0042002F    5D           pop     ebp  
00420030    81ED FC584000 sub   ebp, Try.004058FC  
00420040    50           push    eax                       ; 保存 eax  
00420058    8B85 F1344200 mov   eax, dword ptr ss:[ebp+4234F1]  
00420063    83F8 00      cmp     eax, 0                    ; DLL 重入?  
0042007D    0F85 33030000 jnz   Try.004203B6               ; 第二次进入,无需解压缩,直接跳去执行  
0042008D    B8 C7D70100   mov   eax, 1D7C7  
00420097    83C0 10      add     eax, 10                   ; 计算所需长度  
0042009F    6A 04        push    4  
004200A1    68 00100000  push    1000  
004200A6    50           push    eax                       ; 所需长度  
004200A7    6A 00        push    0  
004200A9    FF95 B93D4200 call  dword ptr ss:[ebp+423DB9]    ; 分配内存, VirtualAlloc  
004200DC    8BF0         mov     esi, eax                  ; esi -> 分配的内存  
00420110    8D9D 835C4000 lea   ebx, dword ptr ss:[ebp+405C83]  ; ebx -> 被压缩的数据  
0042011B    50           push    eax                       ; 目的地  
0042011C    53           push    ebx                       ; 数据来源  
0042011D    E8 15010000  call    Try.00420237              ; APLib 解码, depack  
00420127    83C4 08      add     esp, 8                    ; 平衡堆栈  
00420134    56           push    esi                       ; 保存内存首地址  
0042013A    8BC8         mov     ecx, eax                  ; 数据长度  
00420141    8DBD 835C4000 lea   edi, dword ptr ss:[ebp+405C83]  ; 目的地  
0042014C    F3:A4        rep     movsb                     ; 传送数据  
0042017B    5E           pop     esi                       ; 恢复内存首地址  
004201D6    68 00800000  push    8000  
004201DB    6A 00        push    0  
004201DD    56           push    esi  
004201DE    FF95 B53D4200 call  dword ptr ss:[ebp+423DB5]    ; 释放申请的内存, VirtualFree  
; * forgot : 干吗不在内存里执行……成了固定地址, 让偶们有机会.哈  
00420216    E9 9B010000  jmp     Try.004203B6              ; 跳向解压缩后的数据  
004203B6    90           nop  
004203CD    58           pop     eax                       ; 恢复上边保存的 eax  
004203D3    EB 43        jmp     short Try.00420418        ; 下边放的是一些函数名与其 DLL 名,跳过他们  
---------------------------------------------------------------------------  
004203D5  6B 65 72 6E 65 6C 33 32 2E 64 6C 6C 00 53 6C 65   kernel32.dll.Sle  
004203E5  65 70 00 47 65 74 54 69 63 6B 43 6F 75 6E 74 00   ep.GetTickCount.  
004203F5  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................  
00420405  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................  
00420415  00 00 00                                     ...  
---------------------------------------------------------------------------  
; * forgot: 这一段真无聊  
00420418    60           pushad                            ; 保存寄存器  
00420419    83BD CA5C4000 0>cmp dword ptr ss:[ebp+405CCA], 0    ; Sleep 地址  
00420420    75 2C        jnz     short Try.0042044E        ; 非零表示已经取得, 不再执行这一段  
00420422    8D85 A25C4000 lea   eax, dword ptr ss:[ebp+405CA2]  ; eax -> "kernel32.dll"  
00420428    50           push    eax                       ; push FileName  
00420429    FF95 C13D4200 call  dword ptr ss:[ebp+423DC1]    ; 载入动态链接库, LoadLibraryA  
0042042F    8DB5 AF5C4000 lea   esi, dword ptr ss:[ebp+405CAF]  ; esi -> "Sleep"  
00420435    56           push    esi                       ; ProcNameOrOrdinal = "Sleep"  
00420436    50           push    eax                       ; hModule = 77E40000 (kernel32)  
00420437    FF95 C53D4200 call  dword ptr ss:[ebp+423DC5]    ; 取 Sleep 地址, GetProcAddress  
0042043D    8985 CA5C4000 mov   dword ptr ss:[ebp+405CCA], eax  ; 存入 Sleep 地址, 隐式链接  
00420443    68 F4010000  push    1F4                       ; 500 ms  
00420448    FF95 CA5C4000 call  dword ptr ss:[ebp+405CCA]    ; 调用 Sleep 延时  
0042044E    61           popad                             ; 恢复寄存器  
00420498    8985 F5344200 mov   dword ptr ss:[ebp+4234F5], eax  ; 这是啥玩艺?十里香-_-;;  
004204A8    BE 5C344200 mov     esi, Try.0042345C          ; esi -> 指向一个 OSVERSIONINFO 结构  
004204B2    03F5         add     esi, ebp  
; * D.Boy 真奇怪, lea esi, 0042345C[ebp] 不行吗?很多都这么写,请大虾们不吝赐教.  
004204E1    C706 94000000 mov   dword ptr ds:[esi], 94h    ; sizeof OSVERSIONINFO  
004204EC    56           push    esi  
004204ED    FF95 BD3D4200 call  dword ptr ss:[ebp+423DBD]    ; 取系统版本.GetVersionExA  
004204F8    837E 10 02   cmp     dword ptr ds:[esi+10], 2   ; Windows 98 系列? 这里有个 if...else...endif 结构  
004204FC    75 13        jnz     short Try.00420511  
00420503    C685 F0344200 0>mov byte ptr ss:[ebp+4234F0], 1  ; 设置 NT 标记  
0042050F   /EB 39        jmp     short Try.0042054A  
00420511    90           nop  
00420516    C685 F0344200 0>mov byte ptr ss:[ebp+4234F0], 0  ; 清零 NT 标记  
0042054A    90           nop  
00420559    8D85 CF5C4000 lea   eax, dword ptr ss:[ebp+405CCF]  ; eax -> 上边的一小段 Buffer,就是刚才放 Sleep 等的地方  
004205A9    80BD F0344200 0>cmp byte ptr ss:[ebp+4234F0], 1    ; 某个标记  
004205B0    0F85 114C0000 jnz   Try.004251C7  
004205D2    E8 D40D0000  call    Try.004213AB              ; 取函数地址  
            {  
            004213AB    90           nop  
            004213D8    60           pushad  
            004213DE    BB 04714000  mov     ebx, Try.00407104  
            004213E8    03DD         add     ebx, ebp  
            ; * forgot : 又来了-_-;;  
            00421433    807B 08 00   cmp     byte ptr ds:[ebx+8], 0       ; "ntdll.dll"存在?  
```



```
00421464   /0F84 5D030000   je      Try.004217C7                  ; 跳过  
0042149C    8BC3            mov     eax, ebx                      ; eax = ebx  
004214CB    83C0 08         add     eax, 8                          ; eax -> "ntdll.dll"  
004214D3    50              push    eax  
004214D4    FF95 C13D4200   call    dword ptr ss:[ebp+423DC1]       ; kernel32.LoadLibraryA  
; * 载入"NTDLL.DLL"动态链接库, 我觉得是 Bug,应该先 GetModuleHandleA 才对![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg),有些默认都映射了.  
00421507    8985 FC704000   mov     dword ptr ss:[ebp+4070FC], eax  ; 写入 NTDLL 基地址  
0042153A    83F8 00         cmp     eax, 0                          ; 获得 NTDLL 模块了?  
0042153D    75 46          jnz     short Try.00421585              ; Good...走人  
00421544    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0   ; 设置失败标记  
00421553    E9 6F020000     jmp     Try.004217C7  
00421585    90              nop  
004215A1    8B33            mov     esi, dword ptr ds:[ebx]        ; Try.00407150  
004215A8    8B7B 04         mov     edi, dword ptr ds:[ebx+4]      ; Try.0040719F  
004215B0    03F5            add     esi, ebp                        ; 指向 dll 函数 名表  
004215B7    03FD            add     edi, ebp                        ; 指向 函数地址表  
004215EC    803E 00         cmp     byte ptr ds:[esi], 0  
004215EF    75 58          jnz     short Try.00421649              ; 处理完了吗?  
004215F6    83C3 08         add     ebx, 8  
004215FE    803B 00         cmp     byte ptr ds:[ebx], 0            ; NULL?  
00421601    74 1F          je      short Try.00421622  
0042161A    43              inc     ebx  
00421620    EB DC          jmp     short Try.004215FE  
00421622    90              nop  
0042167B    56              push    esi                             ; 函数名  
0042167C    FFB5 FC704000   push    dword ptr ss:[ebp+4070FC]       ; NTDLL  
00421682    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]       ; kernel32.GetProcAddress  
0042169F    83F8 00         cmp     eax, 0  
004216A2    75 30          jnz     short Try.004216D4  
004216A9    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0  
004216CA   /E9 F8000000     jmp     Try.004217C7  
00421701    8907            mov     dword ptr ds:[edi], eax        ; 写入地址  
00421730    83C7 04         add     edi, 4                          ; 下一个地址  
0042173D    803E 00         cmp     byte ptr ds:[esi], 0            ; NULL?  
00421740    74 31          je      short Try.00421773  
00421759    46              inc     esi  
00421771    EB CA          jmp     short Try.0042173D  
0042178A    46              inc     esi                             ; 上边到达 NULL,这里跳过 NULL  
004217B8    E9 18FEFFFF     jmp     Try.004215D5                  ; 循环处理 -> 004215EC  
; * 所有要处理的函数:  
---------------------------------------------------------------------------  
00421883  52 74 6C 49 6E 69 74 55 6E 69 63 6F 64 65 53 74  RtlInitUnicodeSt  
00421893  72 69 6E 67 00 5A 77 4F 70 65 6E 53 65 63 74 69  ring.ZwOpenSecti  
004218A3  6F 6E 00 5A 77 55 6E 6D 61 70 56 69 65 77 4F 66  on.ZwUnmapViewOf  
004218B3  53 65 63 74 69 6F 6E 00 5A 77 4D 61 70 56 69 65  Section.ZwMapVie  
004218C3  77 4F 66 53 65 63 74 69 6F 6E 00 00 00 00 00 F8  wOfSection.....  
004218D3  41 F9 77 00 00 00 00 00 00 00 00 00 00 00 00 00  A 鵺.............  
004218E3  00 00 00 52 65 6C 65 61 73 65 4D 75 74 65 78 00  ...ReleaseMutex.  
004218F3  43 72 65 61 74 65 4D 75 74 65 78 41 00 44 65 6C  CreateMutexA.Del  
00421903  65 74 65 46 69 6C 65 41 00 44 65 76 69 63 65 49  eteFileA.DeviceI  
00421913  6F 43 6F 6E 74 72 6F 6C 00 47 65 74 4C 61 73 74  oControl.GetLast  
00421923  45 72 72 6F 72 00 47 65 74 53 79 73 74 65 6D 44  Error.GetSystemD  
00421933  69 72 65 63 74 6F 72 79 41 00 43 72 65 61 74 65  irectoryA.Create  
00421943  46 69 6C 65 41 00 57 72 69 74 65 46 69 6C 65 00  FileA.WriteFile.  
00421953  43 6C 6F 73 65 48 61 6E 64 6C 65 00 00 00 00 00  CloseHandle.....  
00421963  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00421973  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00421983  00 00 00 00 00 00 00 00 53 74 61 72 74 53 65 72  ........StartSer  
00421993  76 69 63 65 41 00 44 65 6C 65 74 65 53 65 72 76  viceA.DeleteServ  
004219A3  69 63 65 00 4F 70 65 6E 53 43 4D 61 6E 61 67 65  ice.OpenSCManage  
004219B3  72 41 00 43 72 65 61 74 65 53 65 72 76 69 63 65  rA.CreateService  
004219C3  41 00 4F 70 65 6E 53 65 72 76 69 63 65 41 00 43  A.OpenServiceA.C  
004219D3  6F 6E 74 72 6F 6C 53 65 72 76 69 63 65 00 43 6C  ontrolService.Cl  
004219E3  6F 73 65 53 65 72 76 69 63 65 48 61 6E 64 6C 65  oseServiceHandle  
004219F3  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00421A03  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00421A13  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  
00421A23  00                                                    .  
---------------------------------------------------------------------------  
0042167B    56              push    esi  
0042167C    FFB5 FC704000   push    dword ptr ss:[ebp+4070FC]  
00421682    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]       ; kernel32.GetProcAddress, 最后一个函数  
0042169F    83F8 00         cmp     eax, 0                          ; 失败?  
004216A2    75 30          jnz     short Try.004216D4  
004216A9    C785 00714000 0>mov     dword ptr ss:[ebp+407100], 0    ; 失败标记  
004216CA   /E9 F8000000     jmp     Try.004217C7  
00421701    8907            mov     dword ptr ds:[edi], eax         ; 写入地址  
00421730    83C7 04         add     edi, 4  
0042173D    803E 00         cmp     byte ptr ds:[esi], 0  
00421740    74 31          je      short Try.00421773  
00421759    46              inc     esi  
00421771  ^\EB CA          jmp     short Try.0042173D               ; 循环到遇到 NULL  
0042178A    46              inc     esi                             ; 跳过 NULL,指向下一个函数  
; * forgot :一大堆取函数地址,不说了  
004217CC    61              popad                                   ; 终于结束了  
004217E4    8B85 00714000   mov     eax, dword ptr ss:[ebp+407100]  ; 设置返回值  
00421801    C3              ret  
}
004205E1    8D85 2E6C4000   lea     eax, dword ptr ss:[ebp+406C2E]   ; 告诉我们要干坏事了;-), 互斥体名称  
\---------------------------------------------------------------------------  
00421361  4C 6F 61 64 69 6E 67 44 42 50 45 00 00 00 00 00  LoadingDBPE.....  
```



00421371  43 64 73 79 73 00 5C 5C 2E 5C 44 62 70 65 44 65  Cdsys.\\\\.\\DbpeDe  

00421381  76 69 63 65 30 00 00 00 00 00 00 00 00 00 00 00  vice0...........  

00421391  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................  

004213A1  00 00 00 00 00 00 00 00 00 00                 ..........  

---------------------------------------------------------------------------

```
00420614    50            push    eax
00420615    6A 01         push    1
00420617    6A 00         push    0
00420619    FF95 34724000 call    dword ptr ss:[ebp+407234]        ; 呵呵,创建互斥体,kernel32.CreateMutexA
00420636    83F8 00       cmp     eax, 0
00420666   /0F84 CB490000 je     Try.00425037                     ; 大概装载过了,跳过装载吧
0042069E    8985 2A6C4000 mov    dword ptr ss:[ebp+406C2A], eax   ; 保存互斥体
004206E8    FF95 40724000 call   dword ptr ss:[ebp+407240]        ; ntdll.RtlGetLastWin32Error
00420705    3D B7000000   cmp     eax, 0B7                         ; 忘记是啥了
0042070A    0F85 9B000000 jnz    Try.004207AB
0042073D    FFB5 2A6C4000 push   dword ptr ss:[ebp+406C2A]
00420743    FF95 50724000 call   dword ptr ss:[ebp+407250]        ; 我没到这里,猜想是 CloseHandle
0042074E    C785 2A6C4000 0>mov  dword ptr ss:[ebp+406C2A], 0     ; 互斥体清零
00420785    B8 00000000   mov     eax, 0                           ; 返回 FALSE
0042078F   /E9 A3480000    jmp    Try.00425037
004207AB    90            nop
004207B5    83BD 706C4000 0>cmp  dword ptr ss:[ebp+406C70], 0
004207BC    75 5C         jnz     short Try.0042081A
```

`[part 2]`

```
004207D5    8D85 E15C4000 lea    eax, dword ptr ss:[ebp+405CE1]
004207EB    E8 34120000 call    Try.00421A24

                    00421A68    60            pushad
                    00421A80    B8 92764000   mov     eax, Try.00407692
                    00421A9C    03C5          add     eax, ebp                     ; buffer
                    00421AA3    8BF0          mov     esi, eax
                    00421AAA    68 C8000000   push    MAX_PATH
                    00421AAF    50            push    eax
                    00421AB0    FF95 44724000 call    dword ptr ss:[ebp+407244]     ; kernel32.GetSystemDirectoryA
                    00421ABB    90            nop
                    00421AC0    8A06          mov     al, byte ptr ds:[esi]
                    00421AC7    46            inc     esi
                    00421ADF    3C 00         cmp     al, 0
                    00421B0E  ^\75 AB       jnz     short Try.00421ABB
                    00421B15    C646 FF 5C    mov     byte ptr ds:[esi-1], 5C      ; 目录后边加个"\\"
                    00421B1E    C706 63646364 mov     dword ptr ds:[esi], 64636463  ; cdcd
                    00421B29    C746 04 2E73797>mov dword ptr ds:[esi+4], 7379732E  ; .sys
                    00421B5D    C746 08 0000000>mov dword ptr ds:[esi+8], 0         ; NULL, 铺张浪费,byte 就足够了
                    ; 我这里的文件名
                    ---------------------------------------------------------------------------
                    00421DB9                       ** 3A 5C 57       *:\\W
                    00421DC9  49 4E 44 4F 57 53 5C 53 79 73 74 65 6D 33 32 5C  WINDOWS\\System32\\
                    00421DD9  63 64 63 64 2E 73 79 73                           cdcd.sys
                    ---------------------------------------------------------------------------
                    00421B92    B8 86764000   mov     eax, Try.00407686
                    00421B9C    03C5          add     eax, ebp
                    00421BCB    BB 92764000   mov     ebx, Try.00407692
                    00421BFD    03DD          add     ebx, ebp
                    00421C2C    6A 00         push    0
                    00421C2E    6A 20         push    20
                    00421C30    6A 04         push    4
                    00421C32    6A 00         push    0
                    00421C34    6A 02         push    2
                    00421C36    68 00000040   push    40000000
                    00421C3B    53            push    ebx
                    00421C3C    FF95 48724000 call    dword ptr ss:[ebp+407248]     ; kernel32.CreateFileA
                    ; 参数,清晰一些![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)
                    ---------------------------------------------------------------
                    0012FF60   00421C42  /CALL to CreateFileA from Try.00421C3C
                    0012FF64   00421DC5  |FileName = "E:\\WINDOWS\\System32\\cdcd.sys"
                    0012FF68   40000000  |Access = GENERIC_WRITE
                    0012FF6C   00000002  |ShareMode = FILE_SHARE_WRITE
                    0012FF70   00000000  |pSecurity = NULL
                    0012FF74   00000004  |Mode = OPEN_ALWAYS
                    0012FF78   00000020  |Attributes = ARCHIVE
                    0012FF7C   00000000  \hTemplateFile = NULL
                    ---------------------------------------------------------------
                    00421C4C    83F8 FF       cmp     eax, -1
                    00421C54   /0F84 32010000 je     Try.00421D8C                   ; 坏事了...
                    00421C76    8985 8A764000 mov    dword ptr ss:[ebp+40768A], eax ; 保存句柄
                    00421CC0    B9 8C310000   mov     ecx, 318C                      ; 驱动文件长度
                    00421CCA    B8 8E764000   mov     eax, Try.0040768E
                    00421CE6    03C5          add     eax, ebp                       ; WriteFile 返回结构,名字忘了
                    00421CED    BB 5A774000   mov     ebx, Try.0040775A
                    00421D09    03DD          add     ebx, ebp                       ; 可爱的驱动程序
                    00421D10    6A 00         push    0
                    00421D12    50            push    eax
                    00421D13    51            push    ecx
                    00421D14    53            push    ebx
                    00421D15    FFB5 8A764000 push    dword ptr ss:[ebp+40768A]      ; hWnd
                    00421D1B    FF95 4C724000 call    dword ptr ss:[ebp+40724C]      ; kernel32.WriteFile
                    ; 写出驱动,如果想把这个东西拷贝出来研究要关闭程序,它独占了文件
```



```
    0012FF6C   00000024  |hFile = 00000024  
    0012FF70   00421E8D  |Buffer = Try.00421E8D  
    0012FF74   0000318C  |nBytesToWrite = 318C (12684.)  
    0012FF78   00421DC1  |pBytesWritten = Try.00421DC1  
    0012FF7C   00000000  \pOverlapped = NULL  
```

```
    00421D3D    FFB5 8A764000   push    dword ptr ss:[ebp+40768A]  
    00421D43    FF95 50724000   call    dword ptr ss:[ebp+407250]        ; kernel32.CloseHandle  
```

```
    0012FF78   00421D49  /CALL to CloseHandle from Try.00421D43  
    0012FF7C   00000024  \hObject = 00000024  
```

```
    00421D76    61              popad  
    00421D7C    B8 01000000     mov     eax, TRUE                     ; 收工  
    00421D86    C3              ret                                     ; 回家  
    00421D8C    90              nop  
    00421D91    61              popad  
    00421D7C    B8 01000000     mov     eax, FALSE  
    00421D86    C3              ret  
```

```
00420807    83F8 00         cmp     eax, FALSE                      ; 失败?  
0042080F   /0F84 22480000  je      Try.00425037  
```

```
    00425037    50              push    eax  
    00425038    FFB5 2A6C4000   push    dword ptr ss:[ebp+406C2A]  
    0042503E    FF95 30724000   call    dword ptr ss:[ebp+407230]        ; kernel32.ReleaseMutex,释放互斥体  
    00425044    FFB5 2A6C4000   push    dword ptr ss:[ebp+406C2A]  
    0042504A    FF95 50724000   call    dword ptr ss:[ebp+407250]        ; kernel32.CloseHandle,关闭互斥体句柄  
    00425050    58              pop     eax  
    00425068    83F8 00         cmp     eax, 0  
    0042506B    0F85 4C010000  jnz     Try.004251BD  
    ; ..............我没走这条线,不跟了  
```

```
0042085E    8D85 E25C4000   lea     eax, dword ptr ss:[ebp+405CE2]  
00420874    E8 7F060000     call    Try.00420EF8  
```

```
    -----------------------------  
    以后不跟那么多线了,受不了  
```

```
    00420F2A    68 3F000F00     push    0F003F  
    00420F2F    6A 00           push    0  
    00420F31    6A 00           push    0  
    00420F33    FF95 CD724000   call    dword ptr ss:[ebp+4072CD]    ; advapi32.OpenSCManagerA  
    00420F50    83F8 00         cmp     eax, 0  
    00420F58   /0F84 8F030000  je      Try.004212ED  
    00420F75    8985 3A6C4000   mov     dword ptr ss:[ebp+406C3A], eax  
    00420F85    B8 3E6C4000     mov     eax, Try.00406C3E  
    00420FA1    03C5            add     eax, ebp  
    00420FBA    68 FF010F00     push    0F01FF  
    00420FBF    50              push    eax  
    00420FC0    FFB5 3A6C4000   push    dword ptr ss:[ebp+406C3A]  
    00420FC6    FF95 D5724000   call    dword ptr ss:[ebp+4072D5]        ; advapi32.OpenServiceA  
    00420FD1    83F8 00         cmp     eax, 0  
    00420FD4    0F85 97000000  jnz     Try.00421071                     ; Jump  
    0042109E    83F8 00         cmp     eax, 0  
    004210A6   /0F84 41020000  je      Try.004212ED  
    004210B1    8985 586C4000   mov     dword ptr ss:[ebp+406C58], eax  
    004210C1    6A 00           push    0  
    004210C3    6A 00           push    0  
    004210C5    FFB5 586C4000   push    dword ptr ss:[ebp+406C58]  
    004210CB    FF95 C5724000   call    dword ptr ss:[ebp+4072C5]        ; advapi32.StartServiceA  
    ; 启动服务,不好玩^^,我这里服务开启,所以产生了 ERROR_SERVICE_ALREADY_RUNNING (00000420)  
    004210E4    FF95 40724000   call    dword ptr ss:[ebp+407240]        ; ntdll.RtlGetLastWin32Error  
    00421101    3D E5030000     cmp     eax, 3E5  
    00421106   /0F84 DD000000  je      Try.004211E9  
    00421123    3D 20040000     cmp     eax, 420  
    00421128   /0F84 B6000000  je      Try.004211E4                       ; GoGoGo,否则 Game Over 就不好玩了  
    0042120A    B8 446C4000     mov     eax, Try.00406C44  
    00421226    03C5            add     eax, ebp  
    ; Oh God save me!!!  
```

```
    ---------------------------------------------------------------------------  
    00421377  5C 5C 2E 5C 44 62 70 65 44 65 76 69 63 65 30 00  \\.\DbpeDevice0.  
    ---------------------------------------------------------------------------  
```

```
    0042123F    6A 00           push    0  
    00421241    6A 00           push    0  
    00421243    6A 03           push    3  
    00421245    6A 00           push    0  
    00421247    6A 01           push    1  
    00421249    68 000000C0     push    C0000000  
    0042124E    50              push    eax  
    0042124F    FF95 48724000   call    dword ptr ss:[ebp+407248]        ; kernel32.CreateFileA  
```

```
    0012FF7C   0012FFE0  
    0012FF80   00421255  /CALL to CreateFileA from Try.0042124F  
    0012FF84   00421377  |FileName = "\\\\.\\DbpeDevice0"  
    0012FF88   C0000000  |Access = GENERIC_READ|GENERIC_WRITE  
    0012FF8C   00000001  |ShareMode = FILE_SHARE_READ  
    0012FF90   00000000  |pSecurity = NULL  
    0012FF94   00000003  |Mode = OPEN_EXISTING  
```



```
                        0012FF98   00000000  |Attributes = 0

                        0012FF9C   00000000  \|hTemplateFile = NULL

                        ; 调查一下服务![](img/26ed9fa78f7d02ee5a28c57dc1e1e2a8.jpg)

                        0042126C    83F8 FF         cmp     eax, -1

                        00421274   /74 77           je      short Try.004212ED ; 没启动走人,加载错误

                        0042127B    8985 546C4000   mov     dword ptr ss:\[ebp+406C54\], eax; save handle

                        004212AE    B8 01000000     mov     eax, TRUE

                        004212B8    8985 706C4000   mov     dword ptr ss:\[ebp+406C70\], eax

                        004212D5    C3              ret ; 回去了

004208A6    83F8 00         cmp     eax, 0 ; over?

004208AE   /0F84 83470000   je      Try.00425037 ; over

004208C3    C785 5C6C4000 0>mov     dword ptr ss:\[ebp+406C5C\], 3

004208FF    B8 5C3A4200     mov     eax, Try.00423A5C

00420909    03C5            add     eax, ebp

; forgot: 老是这个,我$@%@%@^%@$^

00420922    8985 606C4000   mov     dword ptr ss:\[ebp+406C60\], eax   ; Try.0043E18F

00420932    B8 746C4000     mov     eax, Try.00406C74

0042093C    03C5            add     eax, ebp

0042096B    8985 646C4000   mov     dword ptr ss:\[ebp+406C64\], eax   ; Try.004213A7

0042097B    B8 5C6C4000     mov     eax, Try.00406C5C

00420985    03C5            add     eax, ebp

0042098C    6A 00           push    0

0042098E    6A 00           push    0

00420990    6A 00           push    0

00420992    6A 00           push    0

00420994    6A 0C           push    0C

00420996    50              push    eax

00420997    6A 04           push    4

00420999    FFB5 546C4000   push    dword ptr ss:\[ebp+406C54\]

0042099F    FF95 3C724000   call    dword ptr ss:\[ebp+40723C\]        ; kernel32.DeviceIoControl

                        0012FF80   004209A5  /CALL to **DeviceIoControl** from Try.0042099F

                        0012FF84   0000004C  |hDevice = 0000004C

                        0012FF88   00000004  |IoControlCode = 4

                        0012FF8C   0042138F  |InBuffer = Try.0042138F

                        0012FF90   0000000C  |InBufferSize = C (12.)

                        0012FF94   00000000  |OutBuffer = NULL

                        0012FF98   00000000  |OutBufferSize = 0

                        0012FF9C   00000000  |pBytesReturned = NULL

                        0012FFA0   00000000  \\pOverlapped = NULL

00420A55    B8 00000000     mov     eax, 0

00420A5F    81BD 746C4000 8>cmp     dword ptr ss:\[ebp+406C74\], FFFF8888

00420A6E   /0F84 C3450000   je      Try.00425037     ; 好像是 Over

00420AB8    C785 5C6C4000 0>mov     dword ptr ss:\[ebp+406C5C\], 3

00420AF4    B8 5C3A4200     mov     eax, Try.00423A5C

00420AFE    03C5            add     eax, ebp

00420B17    8985 606C4000   mov     dword ptr ss:\[ebp+406C60\], eax                ; Try.0043E18F

00420B61    B8 746C4000     mov     eax, Try.00406C74

00420B6B    03C5            add     eax, ebp

00420B9A    8985 646C4000   mov     dword ptr ss:\[ebp+406C64\], eax                ; Try.004213A7

00420BBC    C785 746C4000 0>mov     dword ptr ss:\[ebp+406C74\], 0

00420BE2    B8 5C6C4000     mov     eax, Try.00406C5C

00420BEC    03C5            add     eax, ebp

00420BF3    6A 00           push    0

00420BF5    6A 00           push    0

00420BF7    6A 00           push    0

00420BF9    6A 00           push    0

00420BFB    6A 0C           push    0C

00420BFD    50              push    eax

00420BFE    6A 01           push    1

00420C00    FFB5 546C4000   push    dword ptr ss:\[ebp+406C54\]

00420C06    FF95 3C724000   call    dword ptr ss:\[ebp+40723C\]                    ; kernel32.DeviceIoControl

                        0012FF80   00420C0C  /CALL to **DeviceIoControl** from Try.00420C06

                        0012FF84   0000004C  |hDevice = 0000004C

                        0012FF88   00000001  |IoControlCode = 1

                        0012FF8C   0042138F  |InBuffer = Try.0042138F

                        0012FF90   0000000C  |InBufferSize = C (12.)

                        0012FF94   00000000  |OutBuffer = NULL

                        0012FF98   00000000  |OutBufferSize = 0

                        0012FF9C   00000000  |pBytesReturned = NULL

                        0012FFA0   00000000  \\pOverlapped = NULL

; 中断向量被 xxx 了,不敢用 int3 乱动阿!用 F4 走,或者在 Debug 里设置用 hardware breakpoint 进行 step,否则\*\*\*\*自己想象吧

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

;                     千  万  小  心

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

00420C0C    90              nop    ; F4

00420D07    8B85 746C4000   mov     eax, dword ptr ss:\[ebp+406C74\]

00420D3A    83F8 00         cmp     eax, 0

00420D42   /0F84 EF420000   je      Try.00425037    ; over

00420D7A    66:8985 5434420>mov     word ptr ss:\[ebp+423454\], ax

00420D86    C1E8 10         shr     eax, 10

00420D8E    66:8985 5634420>mov     word ptr ss:\[ebp+423456\], ax

00420DC7    B8 01000000     mov     eax, 1

00420DD1    8985 6C6C4000   mov     dword ptr ss:\[ebp+406C6C\], eax

00420E09    50              push    eax

00420E0F    B8 92764000     mov     eax, Try.00407692

00420E2B    03C5            add     eax, ebp

; 对驱动进行惨无人道的毁尸灭迹……

00420E5A    50              push    eax                                       ; Try.00421DC5

00420E5B    FF95 38724000   call    dword ptr ss:\[ebp+407238\]               ; kernel32.DeleteFileA

                        0012FF98   00420E61  /CALL to DeleteFileA from Try.00420E5B

                        0012FF9C   00421DC5  \\FileName = "E:\\WINDOWS\\System32\\cdcd.sys"

00420E66    58              pop     eax

00420E6C   /E9 C6410000     jmp     Try.00425037

00425037    50              push    eax

00425038    FFB5 2A6C4000   push    dword ptr ss:\[ebp+406C2A\]

0042503E    FF95 30724000   call    dword ptr ss:\[ebp+407230\]               ; kernel32.ReleaseMutex

                        0012FF98   00425044  /CALL to **ReleaseMutex** from Try.0042503E

                        0012FF9C   00000020  \\hMutex = 00000020

00425044    FFB5 2A6C4000   push    dword ptr ss:\[ebp+406C2A\]

0042504A    FF95 50724000   call    dword ptr ss:\[ebp+407250\]               ; kernel32.CloseHandle

                        0012FF98   00425050  /CALL to **CloseHandle** from Try.0042504A

                        0012FF9C   00000020  \\hObject = 00000020

; 没人性，互斥体也惨遭毒手……

00425050    58              pop     eax

00425068    83F8 00         cmp     eax, 0

0042506B    0F85 4C010000   jnz     Try.004251BD    ; Go

004251C2   /E9 39010000     jmp     Try.00425300

; 花絮：跳过的这段代码还真惹眼：）

\----------------------------------------------------------------

004251CC    FA              cli

004251D2    BE 4E344200     mov     esi, Try.0042344E

004251DC    03F5            add     esi, ebp

004251E3    0F010E          sidt    fword ptr ds:\[esi\]

004251EB    8B76 02         mov     esi, dword ptr ds:\[esi+2\]

0042520A    66:8B46 18      mov     ax, word ptr ds:\[esi+18\]
```



### 汇编代码分析

#### 第一部分：内存操作与初始化

```
00425213    66:8B5E 1E      mov     bx, word ptr ds:[esi+1E]  
00425244    66:8985 5434420>mov     word ptr ss:[ebp+423454], ax  
00425250    66:899D 5634420>mov     word ptr ss:[ebp+423456], bx  
00425289    B8 5C3A4200     mov     eax, Try.00423A5C  
00425293    03C5            add     eax, ebp  
004252AC    66:8946 18      mov     word ptr ds:[esi+18], ax  
004252B5    C1E8 10         shr     eax, 10  
004252E5    66:8946 1E      mov     word ptr ds:[esi+1E], ax  
```

; 好在我不是 9x....哈

---

#### 第二部分：解密逻辑（Part 3）

; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
; 当当当~~~~~ 醒醒啦！！！
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
0042532D    8D85 D05C4000   lea     eax, dword ptr ss:[ebp+405CD0] ; 没弄明白，也许是参数  
00425334    B0 94           mov     al, 94        ; 后边用来 xor 的 key  
00425336    E8 00000000     call    Try.0042533B  
0042533B    5E              pop     esi      ; 取这条信令地址  
0042533C    81C6 A6000000   add     esi, 0A6 ; 加上这段解密代码的长度，指向下一部分，进行 SMC 解密  
00425347    B9 68010000     mov     ecx, 168h        ; 解码长度  
00425356    8A26            mov     ah, byte ptr ds:[esi] ; 读取一个字节  
0042535D    32E0            xor     ah, al  
00425376    F6D4            not     ah  
0042538F    8826            mov     byte ptr ds:[esi], ah ; 解密后存回去  
004253A8    46              inc     esi ; 下一个字节  
004253AE    49              dec     ecx ; 计数器--  
004253B4    83F9 00         cmp     ecx, 0  
004253B7    75 98           jnz     short Try.00425351 ; 循环直到都解密完成  
```

; 精彩片断！！！！！！！！！！

```
004253BE    FEC8            dec     al     ; 换一个 key  
004253C5    BB 4F434E55     mov     ebx, 554E434F ; 不清楚，自己猜吧  
0042540E    B9 68010000     mov     ecx, 168     ; 这一块长度  
0042542A    8A240E          mov     ah, byte ptr ds:[esi+ecx]    ; 读取一个字节  
00425444    88240E          mov     byte ptr ds:[esi+ecx], ah   ; 再写进去，什么名堂？  
0042545E    CC              int     3 ; 别乱动，int 3 解码  
0042545F    E9 E5000000     jmp     Try.00425549     ; 按 F4 ，int3 的向量处理好解码了，过去看看  
                        ; 跳过的好像是下一次 int3 的向量,俺可不敢进去胡闹  
00425464    3D 534E5552     cmp     eax, 52554E53  
00425469    75 24           jnz     short Try.0042548F  
00425482    FFD3            call    ebx  
00425489    CF              iretd  
```

#### 第三部分：重复的 Int3 陷阱循环

以下代码块展示了多次类似的 `int3` 和跳转逻辑，用于调试或混淆。

```
00425576    B9 68010000     mov     ecx, 168 ; TMD,没完没了了  
00425592    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004255AC    88240E          mov     byte ptr ds:[esi+ecx], ah  
004255C6    CC              int3  
004255C7    E9 E5000000     jmp     Try.004256B1     ; F4  
004256DE    B9 68010000     mov     ecx, 168 ; KAO....  
004256FA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425714    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042572E    CC              int3  
0042572F    E9 E5000000     jmp     Try.00425819  
00425846    B9 68010000     mov     ecx, 168 ; ……  
00425862    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042587C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425896    CC              int3  
00425897    E9 E5000000     jmp     Try.00425981  
004259AE    B9 68010000     mov     ecx, 168     ; my heart will go on  
004259CA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004259E4    88240E          mov     byte ptr ds:[esi+ecx], ah  
004259FE    CC              int3  
004259FF    E9 E5000000     jmp     Try.00425AE9  
00425B16    B9 68010000     mov     ecx, 168 ; 没完没了  
00425B32    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425B4C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425B66    CC              int3  
00425B67    E9 E5000000     jmp     Try.00425C51  
00425C7E    B9 68010000     mov     ecx, 168 ; 不见不散  
00425C9A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425CB4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425CCE    CC              int3  
00425CCF    E9 E5000000     jmp     Try.00425DB9  
```

#### 第四部分：后续 Int3 序列

```
00425C7E    B9 68010000     mov     ecx, 168 ; 有话好好说  
00425C9A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425CB4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425CCE    CC              int3  
00425CCF    E9 E5000000     jmp     Try.00425DB9  
00425DE6    B9 68010000     mov     ecx, 168 ; 声声慢……  
00425E02    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425E1C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425E36    CC              int3  
00425E37    E9 E5000000     jmp     Try.00425F21  
00425F4E    B9 68010000     mov     ecx, 168  
00425F6A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00425F84    88240E          mov     byte ptr ds:[esi+ecx], ah  
00425F9E    CC              int3  
00425F9F    E9 E5000000     jmp     Try.00426089  
004260B6    B9 68010000     mov     ecx, 168  
004260D2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004260EC    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426106    CC              int3  
00426107    E9 E5000000     jmp     Try.004261F1  
0042621E    B9 68010000     mov     ecx, 168  
0042623A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426254    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042626E    CC              int3  
0042626F    E9 E5000000     jmp     Try.00426359  
00426386    B9 68010000     mov     ecx, 168  
004263A2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004263BC    88240E          mov     byte ptr ds:[esi+ecx], ah  
004263D6    CC              int3  
004263D7    E9 E5000000     jmp     Try.004264C1  
004264EE    B9 68010000     mov     ecx, 168  
0042650A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426524    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042653E    CC              int3  
0042653F    E9 E5000000     jmp     Try.00426629  
00426656    B9 68010000     mov     ecx, 168  
00426672    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042668C    88240E          mov     byte ptr ds:[esi+ecx], ah  
004266A6    CC              int3  
004266A7    E9 E5000000     jmp     Try.00426791  
004267BE    B9 68010000     mov     ecx, 168  
004267DA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004267F4    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042680E    CC              int3  
0042680F    E9 E5000000     jmp     Try.004268F9  
00426926    B9 68010000     mov     ecx, 168  
00426942    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042695C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426976    CC              int3  
00426977    E9 E5000000     jmp     Try.00426A61  
00426A8E    B9 68010000     mov     ecx, 168  
00426AAA    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426AC4    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426ADE    CC              int3  
00426ADF    E9 E5000000     jmp     Try.00426BC9  
00426BF6    B9 68010000     mov     ecx, 168  
00426C12    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426C2C    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426C46    CC              int3  
00426C47    E9 E5000000     jmp     Try.00426D31  
00426D5E    B9 68010000     mov     ecx, 168  
00426D7A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426D94    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426DAE    CC              int3  
00426DAF    E9 E5000000     jmp     Try.00426E99  
```

#### 第五部分：尾部代码

```
00426D5E    B9 68010000     mov     ecx, 168  
00426D7A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
```



```
00426D94    88240E            mov     byte ptr ds:[esi+ecx], ah  
00426DAE    CC              int3  
00426DAF    E9 E5000000      jmp     Try.00426E99  
00426EC6    B9 68010000      mov     ecx, 168  
00426EE2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00426EFC    88240E          mov     byte ptr ds:[esi+ecx], ah  
00426F16    CC              int3  
00426F17    E9 E5000000      jmp     Try.00427001  
0042702E    B9 68010000      mov     ecx, 168  
0042704A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00427064    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042707E    CC              int3  
0042707F    E9 E5000000      jmp     Try.00427169  
00427196    B9 68010000      mov     ecx, 168  
004271B2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
004271CC    88240E          mov     byte ptr ds:[esi+ecx], ah  
004271E6    CC              int3  
004271E7    E9 E5000000      jmp     Try.004272D1  
004272FE    B9 68010000      mov     ecx, 168  
0042731A    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
00427334    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042734E    CC              int3  
0042734F    E9 E5000000      jmp     Try.00427439  
```

```
00429D84    3D 534E5552      cmp     eax, 52554E53  
00429D89    75 24            jnz     short Try.00429DAF  
00429DA2    FFD3            call    ebx  
00429DA9    CF              iretd  
00429DAF    81FB 4F434E55   cmp     ebx, 554E434F  
00429DB5    0F85 AD000000   jnz     Try.00429E68  
00429DBB    8A26            mov     ah, byte ptr ds:[esi]  
00429DC2    32E0            xor     ah, al  
00429DDB    F6D4            not     ah  
00429DE2    8826            mov     byte ptr ds:[esi], ah  
00429DE9    46              inc     esi  
00429DEF    49              dec     ecx  
00429DF5    83F9 00         cmp     ecx, 0  
00429DF8    75 C1           jnz     short Try.00429DBB  
00429DFF    FEC8            dec     al  
00429E06    8DBD 4E344200   lea     edi, dword ptr ss:[ebp+42344E]  
00429E11    0F010F          sidt    fword ptr ds:[edi]  
00429E19    8B7F 02         mov     edi, dword ptr ds:[edi+2]  
00429E1C    8B1C24          mov     ebx, dword ptr ss:[esp]  
00429E24    803B E9         cmp     byte ptr ds:[ebx], 0E9  
00429E27    75 05           jnz     short Try.00429E2E  
00429E29    83C3 05         add     ebx, 5  
00429E2C    EB 03           jmp     short Try.00429E31  
00429E2E    83C3 02         add     ebx, 2  
00429E36    66:895F 18      mov     word ptr ds:[edi+18], bx  
00429E3F    C1EB 10         shr     ebx, 10  
00429E47    66:895F 1E      mov     word ptr ds:[edi+1E], bx  
00429E50    BB 55010000      mov     ebx, 155  
00429E5A    0F23FB          mov     dr7, ebx          ; Privileged command  
00429E62    BB 4F434E55      mov     ebx, 554E434F  
00429E67    CF              iretd  
00429E68    CF              iretd  
```

```
0042CB96    B9 68010000      mov     ecx, 168  
0042CBB2    8A240E          mov     ah, byte ptr ds:[esi+ecx]  
0042CBCC    88240E          mov     byte ptr ds:[esi+ecx], ah  
0042CBE6    CC              int3  
0042CBE7    E9 E5000000      jmp     Try.0042CCD1  
0042EFF9    E8 00000000      call    Try.0042EFFE  
0042EFFE    5D              pop     ebp  
0042EFFF    81ED CB484100   sub     ebp, Try.004148CB     ; 长征过去……再来取 delta  
0042F005    58              pop     eax  
0042F038    80E4 01         and     ah, 1  
0042F052    32C0            xor     al, al  
0042F081    66:3185 2DD6410>xor     word ptr ss:[ebp+41D62D], ax  
0042F0BF    8D85 D15C4000   lea     eax, dword ptr ss:[ebp+405CD1]  
0042F0FE    BE 944B4100      mov     esi, Try.00414B94  
0042F130    03F5            add     esi, ebp  
0042F149    B9 B6E80000      mov     ecx, 0E8B6  
0042F158    03F1            add     esi, ecx  
0042F187    4E              dec     esi  
0042F1B5    BB 01000000      mov     ebx, 1  

; 又解码  
0042F1C4    8A06            mov     al, byte ptr ds:[esi]  
0042F1F3    3246 01         xor     al, byte ptr ds:[esi+1]  
0042F1FB    32C3            xor     al, bl  
0042F219    3285 02AC4000   xor     al, byte ptr ss:[ebp+40AC02]  
0042F251    8806            mov     byte ptr ds:[esi], al  
0042F258    4E              dec     esi  
0042F25E    43              inc     ebx  
0042F264    49              dec     ecx  
0042F26A    83F9 00         cmp     ecx, 0  
0042F26D    0F85 4CFFFFFF   jnz     Try.0042F1BF  
0042F278    66:81BD 944B410>cmp     word ptr ss:[ebp+414B94], 9090  
0042F2AE    75 FE           jnz     short Try.0042F2AE    ; 搞怪？  
0042F2C9    8B85 F1344200   mov     eax, dword ptr ss:[ebp+4234F1]  
0042F2CF    83F8 00         cmp     eax, 0  
0042F2D2    B9 7DDB0100      mov     ecx, 1DB7D  
0042F2D7    0F85 E0020000   jnz     Try.0042F5BD  
0042F2DD    E8 00000000      call    Try.0042F2E2  
0042F2E2    5A              pop     edx  
0042F310    2B95 F4D54100   sub     edx, dword ptr ss:[ebp+41D5F4]  
0042F31B    81EA E2F20000   sub     edx, 0F2E2  
0042F326    8995 DCD34100   mov     dword ptr ss:[ebp+41D3DC], edx  
0042F3B3    8D85 D25C4000   lea     eax, dword ptr ss:[ebp+405CD2]  
0042F458    B8 87CC4100      mov     eax, Try.0041CC87  
0042F462    03C5            add     eax, ebp    ; eax -> IID  
0042F469    E8 10890000      call    Try.00437D7E    ; 取函数地址  
```

```
00437DAB    60              pushad  
00437DB1    8BD8            mov     ebx, eax  

; 大循环开始  
00437DC2    807B 08 00      cmp     byte ptr ds:[ebx+8], 0  
00437DCB    0F84 1B030000   je      Try.004380EC          ; over  
00437DED    8BC3            mov     eax, ebx  
00437DF4    83C0 08         add     eax, 8  
00437DFC    50              push    eax  
```



```asm
    00437DFD    FF95 C13D4200   call    dword ptr ss:[ebp+423DC1]        ; kernel32.LoadLibraryA  
    00437E08    8985 7FCC4100   mov     dword ptr ss:[ebp+41CC7F], eax    ; 保存 hModule  
    00437E13    83F8 00         cmp     eax, 0                        ; faint  
    00437E16   /75 46          jnz     short Try.00437E5E    ; JUMP  
    00437E1D    C785 83CC4100 0>mov     dword ptr ss:[ebp+41CC83], 0    ; failed....  
    00437E54   /E9 93020000     jmp     Try.004380EC    ; Game Over  
    00437EA2    8B33            mov     esi, dword ptr ds:[ebx]          ; Try.0041CCD4  
    00437EBB    8B7B 04         mov     edi, dword ptr ds:[ebx+4]  
    00437ED5    03F5            add     esi, ebp    ; 修正地址  
    00437EDC    03FD            add     edi, ebp  
    00437EFF    803E 00         cmp     byte ptr ds:[esi], 0 ; 表处理完了？  
    00437F02    0F85 BE000000   jnz     Try.00437FC6    ;没有就处理 -> ***  
    00437F35    83C3 08         add     ebx, 8  
    ; ebx 指向下一个 DLL 名称  
    00437F65    803B 00         cmp     byte ptr ds:[ebx], 0  
    00437F68    74 1F           je      short Try.00437F89  
    00437F81    43              inc     ebx  
    00437F87  ^\EB DC          jmp     short Try.00437F65  
    00437FB6    43              inc     ebx                           ; Try.004373CE  
    00437FBC  ^\E9 FCFDFFFF     jmp     Try.00437DBD  
    ; 大循环结束  
    ;********************************************************************  
    ;***:  
    00437FD0    56              push    esi  
    00437FD1    FFB5 7FCC4100   push    dword ptr ss:[ebp+41CC7F]  
    00437FD7    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]        ; kernel32.GetProcAddress,取地址  
    00437FE2    83F8 00         cmp     eax, 0  
    00437FE5    75 30          jnz     short Try.00438017    ; GO  
    00437FEC    C785 83CC4100 0>mov     dword ptr ss:[ebp+41CC83], 0    ; failed again  
                        ..........  
    00438038    8907            mov     dword ptr ds:[edi], eax          ; 写入函数地址  
    00438051    83C7 04         add     edi, 4    ; 下一个 Thunk  
    ; 使 esi 指向下一个函数名  
    0043805E   /803E 00         cmp     byte ptr ds:[esi], 0  
    00438061  | 74 0D           je      short Try.00438070  
    00438068  | 46              inc     esi  
    0043806E   \EB EE          jmp     short Try.0043805E  
    0043809D    46              inc     esi                           ; skip NULL  
    004380CB  ^\E9 2AFEFFFF     jmp     Try.00437EFA  
    004380F1    61              popad  
    004380F7    8B85 83CC4100   mov     eax, dword ptr ss:[ebp+41CC83]    ; 返回标志  
    00438102    C3              ret  
    ----------------------------------------------------------------------  
```

```asm
0042F473    BB 3AD34100     mov     ebx, Try.0041D33A  
0042F47D    03DD            add     ebx, ebp    ; 出错信息  
0042F484    66:3D 0000      cmp     ax, 0  
0042F48D   /0F84 3D2B0000   je      Try.00431FD0    ; 某个错误  
0042F4C5    B9 60E50100     mov     ecx, 1E560  
0042F4E1    83C1 20         add     ecx, 20  
0042F516    6A 00           push    0  
0042F518    51              push    ecx  
0042F519    6A 00           push    0  
0042F51B    6A 04           push    4  
0042F51D    6A 00           push    0  
0042F51F    6A FF          push    -1  
0042F521    FF95 C3CE4100   call    dword ptr ss:[ebp+41CEC3]        ; kernel32.CreateFileMappingA  
                0012FF8C   0042F527  /CALL to CreateFileMappingA from Try.0042F521  
                0012FF90   FFFFFFFF  |hFile = FFFFFFFF  
                0012FF94   00000000  |pSecurity = NULL  
                0012FF98   00000004  |Protection = PAGE_READWRITE  
                0012FF9C   00000000  |MaximumSizeHigh = 0  
                0012FFA0   0001E580  |MaximumSizeLow = 1E580  
                0012FFA4   00000000  \MapName = NULL  
0042F52C    6A 00           push    0  
0042F52E    6A 00           push    0  
0042F530    6A 00           push    0  
0042F532    6A 02           push    2  
0042F534    50              push    eax  
0042F535    FF95 BFCE4100   call    dword ptr ss:[ebp+41CEBF]        ; kernel32.MapViewOfFile  
                0012FF90   0042F53B  /CALL to `MapViewOfFile` from Try.0042F535  
                0012FF94   00000060  |hMapObject = 00000060 (window)  
                0012FF98   00000002  |AccessMode = FILE_MAP_WRITE  
                0012FF9C   00000000  |OffsetHigh = 0  
                0012FFA0   00000000  |OffsetLow = 0  
                0012FFA4   00000000  \MapSize = 0  
; map it...  

0042F557    8985 F1344200   mov     dword ptr ss:[ebp+4234F1], eax    ; 保存  
0042F574    B9 60E50100     mov     ecx, 1E560  
0042F5C2    8BD0            mov     edx, eax  
0042F5C9    8BF8            mov     edi, eax    ; 要解一部分 loader 代码到内存  
0042F5D0    BE CD584000     mov     esi, Try.004058CD  
0042F5DA    03F5            add     esi, ebp    ; 未还原数据  
0042F5F3    F3:A4           rep     movsb  
0042F5FF    8D85 D35C4000   lea     eax, dword ptr ss:[ebp+405CD3]  
0042F615    B8 804F4100     mov     eax, Try.00414F80  
0042F61F    2D CD584000     sub     eax, Try.004058CD  
0042F629    03C2            add     eax, edx  
0042F642    8B9D F5344200   mov     ebx, dword ptr ss:[ebp+4234F5]  
0042F64D    50              push    eax  
0042F67B    C3              ret    ; funy jump...  

0042F67B    C3              ret  ; 二步瞬移~~~ ==> 003AF6B3
```

`[part 4]`

```asm
003AF6B3    90              nop  
003AF6B8    8BC5            mov     eax, ebp  
003AF6BA    E8 00000000     call    003AF6BF  
003AF6BF    5D              pop     ebp  
003AF6C0    81ED 8C4F4100   sub     ebp, 414F8C  
003AF6DD    50              push    eax  
003AF70B    53              push    ebx  
003AF711    B8 534E5552     mov     eax, 52554E53  
003AF743    8D9D 71504100   lea     ebx, dword ptr ss:[ebp+415071]    ; 通过 int3 调用地址  
003AF760    CC              int3    ; 改中断  
003AF766    5B              pop     ebx  
003AF794    58              pop     eax  
```



```
003AF79A   /E9 DF000000     jmp     003AF87E    ; 走人  

-----------------------------------------------------  

; 9x 的处理  

003AF7A9    8DB5 4E344200   lea     esi, dword ptr ss:[ebp+42344E]  

003AF7B4    0F010E          sidt    fword ptr ds:[esi]  

003AF7E4    8B76 02         mov     esi, dword ptr ds:[esi+2]  

003AF803    8D85 5C3A4200   lea     eax, dword ptr ss:[ebp+423A5C]  

003AF836    66:8946 18      mov     word ptr ds:[esi+18], ax  

003AF83F    C1E8 10         shr     eax, 10  

003AF847    66:8946 1E      mov     word ptr ds:[esi+1E], ax  

003AF850    C3              retn  

-----------------------------------------------------  

003AF8DD    8985 3FD64100   mov     dword ptr ss:[ebp+41D63F], eax  

003AF8E8    C785 43D64100 C>mov     dword ptr ss:[ebp+41D643], 4058CD  

003AF8F7    0185 43D64100   add     dword ptr ss:[ebp+41D643], eax    ; 修正 base  

003AF92A    899D F5344200   mov     dword ptr ss:[ebp+4234F5], ebx  

003AF951    BF CD584000     mov     edi, 4058CD  

003AF95B    03F8            add     edi, eax    ; 还是修正 base  

003AF962    B9 7DDB0100     mov     ecx, 1DB7D    ; 长度  

003AF96C    32C0            xor     al, al  

003AF973    F3:AA          rep     stosb    ; 擦除外壳区段里的东西(我们在内存里!)  

003AF991    8D85 D45C4000   lea     eax, dword ptr ss:[ebp+405CD4]  

003AF9F6    60              pushad    ; 看到这个肯定要干坏事  

003AFA0E    B8 534E5552     mov     eax, 52554E53  

003AFA18    BB C0534100     mov     ebx, 4153C0  

003AFA22    03DD            add     ebx, ebp  

003AFA29    CC              int3    ; F4 到这里  

; 我这里 int3 的处理是：3be18f  

---------------------------------------------------  

003BE194    3D 534E5552     cmp     eax, 52554E53  

003BE199    75 74           jnz     short 003BE20F  

003BE1B2    FFD3            call    ebx  

003BE1E1    CF              iretd  

---------------------------------------------------  

; 可见是通过 int3 执行 ebx, ebx 似乎是通过向量反跟踪  

003AFA2F    66:3D 0000      cmp     ax, 0    ; 再 F4 到这里  

003AFA38    61              popad  

003AFA55    9C              pushfd    ; 害怕 ZF 出事, 看来一定有鬼：）  

003AFA5B    BB 8AD34100     mov     ebx, 41D38A  

003AFA77    03DD            add     ebx, ebp  

003AFA7E    9D              popfd  

003AFE1F    83BD 47D64100 0>cmp     dword ptr ss:[ebp+41D647], 0  

003AFE26    0F84 CA000000   je      003AFEF6  

003AFF12    83BD 041E4200 0>cmp     dword ptr ss:[ebp+421E04], 1  

003AFF19    0F84 9D000000   je      003AFFBC  

003AFFEE    8D85 D55C4000   lea     eax, dword ptr ss:[ebp+405CD5]  

003B0031    83BD 001E4200 0>cmp     dword ptr ss:[ebp+421E00], 1  

003B0038    0F85 9F000000   jnz     003B00DD  

003B0043    8DB5 5C344200   lea     esi, dword ptr ss:[ebp+42345C]    ; 开始用 GetVersionExA 获得的版本  

003B004E    837E 10 02      cmp     dword ptr ds:[esi+10], 2  

003B0052   /75 31           jnz     short 003B0085    ; 不是 9x?  

003B0059    837E 04 04      cmp     dword ptr ds:[esi+4], 4  

003B005D    77 0F           ja      short 003B006E ; 不是未知？ 那就是 NT  

003B00B2    89AD 8A204200   mov     dword ptr ss:[ebp+42208A], ebp  

003B00BD    8D9D 88204200   lea     ebx, dword ptr ss:[ebp+422088]    ; 怕~~~~  

---------------------------------------------------------------------------  

003B7ABD  20 20 20 45 72 72 6F 72 20 28 33 29 3A 20 44 65     Error (3): De  

003B7ACD  62 75 67 67 65 72 20 64 65 74 65 63 74 69 6F 6E  bugger detection  

003B7ADD  20 2C 20 41 62 6F 72 74 21 00 20 20 20 45 72     , Abort! .   Er  

003B7AED  72 6F 72 20 28 34 29 3A 20 46 69 6C 65 20 43 52  oror (4): File CR  

003B7AFD  43 20 45 72 72 6F 72 2C 20 20 20 41 62 6F 72 74  C Error,   Abort  

003B7B0D  21 00                                             !.  

---------------------------------------------------------------------------  

; 反跟踪定时器,不好玩,干掉他===>how?看下边  

003B00C8    53              push    ebx  

003B00C9    68 F4010000     push    1F4  

003B00CE    6A 00           push    0  

003B00D0    6A 00           push    0  

003B00D2    FF95 68CF4100   call    dword ptr ss:[ebp+41CF68]        ; user32.SetTimer  

---> F7 进入  

            77D19160 >   B8 1E120000     mov     eax, 121E    ; 修改为 retn    10 了事  

            77D19165    BA 0003FE7F     mov     edx, 7FFE0300  

            77D1916A    FFD2            call    edx  

            77D1916C    C2 1000         retn    10  

                                             0012FF94   003B00D8  /CALL to `SetTimer` from 003B00D2  

                                             0012FF98   00000000  |hWnd = NULL  

                                             0012FF9C   00000000  |TimerID = 0  

                                             0012FFA0   000001F4  |Timeout = 500. ms  

                                             0012FFA4   003BC7BB  \\Timerproc = 003BC7BB ;干坏事,发生 `mov eax, \eax\`异常有兴趣可以研究一下  

; 恢复刚才的 SetTimer 代码...小心至上  

003B0142    8D85 D65C4000   lea     eax, dword ptr ss:[ebp+405CD6]    ; 无聊  

; ==============================================================================  

;                   A n t i - D u m p i n g  

;  

;      我只知道是 Anti-Dumping,原理大概是修改 ImageSize,有兴趣自己研究吧.  

; ==============================================================================  

003B016A    64:67:A1 3000   mov     eax, dword ptr fs:[30]  

003B0174    85C0            test    eax, eax  

003B01A3    78 78           js      short 003B021D    ; else...根据系统不同决定  

003B01C1    8B40 0C         mov     eax, dword ptr ds:[eax+C]  

003B01DB    8B40 0C         mov     eax, dword ptr ds:[eax+C]  

003B01F5    C740 20 0010000  mov     dword ptr ds:[eax+20], 1000  

003B0201    E9 9E000000     jmp     003B02A4            ; `anti_dump_over`  

003B022C    6A 00           push    0  

003B022E    FF95 F7CE4100   call    dword ptr ss:[ebp+41CEF7]    ; `GetModuleHandleA`  

003B023E    85D2            test    edx, edx  

003B0245    79 5D           jns     short 003B02A4    ; `anti_dump_over`  

003B024C    837A 08 FF      cmp     dword ptr ds:[edx+8], -1  

003B0267    75 3B           jnz     short 003B02A4    ; `anti_dump_over`  

003B026E    8B52 04         mov     edx, dword ptr ds:[edx+4]  

003B028D    C742 50 0010000>mov     dword ptr ds:[edx+50], 1000  

003B0299    66:C742 06 1010  mov     word ptr ds:[edx+6], 1010    ; 没见过:P,NumberOfSections?  

; ==============================================================================  

003B02DB    8D85 D75C4000   lea     eax, dword ptr ss:[ebp+405CD7]; 错误消息  

; ==============================================================================  

;                             M e l t I C E  

;  

;                                  地球人都知道  

; ==============================================================================  

003B0303    BE 26CC4100     mov     esi, 41CC26  

003B031F    03F5            add     esi, ebp            ; esi -> 指向 MeltICE 驱动表  

; 不幸儿童:  

---------------------------------------------------------------------------  

003B7359  5C 5C 2E 5C 62 77 32 6B 00 5C 5C 2E 5C 53 55 50   \\\.\\bw2k.\\\.\\SUP  

003B7369  45 52 42 50 4D 00 5C 5C 2E 5C 49 43 45 44 55 4D  ERBPM.\\\.\\ICEDUM  

003B7379  50 00 5C 5C 2E 5C 52 45 47 56 58 44 00 5C 5C 2E  P.\\\.\\REGVXD.\\\.  

003B7389  5C 4E 54 49 43 45 00 5C 5C 2E 5C 53 49 57 56 49  \\NTICE.\\\.\\SIWVI  

003B7399  44 00 5C 5C 2E 5C 53 49 43 45 00 5C 5C 2E 5C 46  D.\\\.\\SICE.\\\.\\F  

003B73A9  49 4C 45 56 58 44 00                             ILEVXD.  

---------------------------------------------------------------------------  

003B037C    FFB5 DBCE4100   push    dword ptr ss:[ebp+41CEDB]        ; `kernel32.CreateFileA`  

003B0382    8F85 F9344200   pop     dword ptr ss:[ebp+4234F9]        ; `kernel32.CreateFileA`  

; 无聊的倒腾~~~  

003B03CC    6A 00           push    0  

003B03CE    6A 00           push    0  

003B03D0    6A 00           push    0  

003B03D2    6A 00           push    0  

003B03D4    6A 00           push    0  
```



# 代码分析与反汇编

## 1. 文件操作与校验

### 1.1 打开设备文件
```assembly
003B03D6    6A 00             push    0
003B03D8    56                push    esi
003B03D9    FF95 DBCE4100     call    dword ptr ss:[ebp+41CEDB]        ; kernel32.CreateFileA
```
**调用参数细节：**
```text
0012FF88   003B03DF  /CALL to CreateFileA from 003B03D9
0012FF8C   003B7359  |FileName = "\\\\.\\theif"    ; -_--;
0012FF90   00000000  |Access = 0
0012FF94   00000000  |ShareMode = 0
0012FF98   00000000  |pSecurity = NULL
0012FF9C   00000000  |Mode = 0
0012FFA0   00000000  |Attributes = 0
0012FFA4   00000000  \\hTemplateFile = NULL
```

### 1.2 检查返回值与循环
```assembly
003B03E9    83F8 FF           cmp     eax, -1
003B03F1    /0F85 8E000000    jnz     003B0485        ; 事情不好办啦^\_^
```
**循环使 `esi` 指向下一个名称：**
```assembly
003B041D    46                inc     esi
003B0423    | 803E 00         cmp     byte ptr ds:[esi], 0
003B042B    \ 75 EB          jnz     short 003B0418 ; ===> 003B041D
003B0432    46                inc     esi    ; Skip NULL char
003B044A    803E 00           cmp     byte ptr ds:[esi], 0    ; 表示用完了吗?
003B047A    ^\\0F85 E5FEFFFF  jnz     003B0365             ; 没完?接着来……
```

### 1.3 错误处理信息
```assembly
; ==============================================================================
003B04DF    8D9D 5FD34100     lea     ebx, dword ptr ss:[ebp+41D35F] ; 我又怕了
```
**相关字符串数据：**
```text
003B7A92  20 20 20 45 72 72 6F 72 20 28 32 29 3A 20 44 65      Error (2): De
003B7AA2  62 75 67 67 65 72 20 64 65 74 65 63 74 69 6F 6E  bugger detection
003B7AB2  20 2C 20 41 62 6F 72 74 21 20 00 20 20 20 45 72    , Abort! .   Er
003B7AC2  72 6F 72 20 28 33 29 3A 20 44 65 62 75 67 67 65  ror (3): Debugge
003B7AD2  72 20 64 65 74 65 63 74 69 6F 6E 20 2C 20 41 62   r detection , Ab
003B7AE2  6F 72 74 21 20 00                                 ort! .
```

```assembly
003B04FC    83F8 FF           cmp     eax, -1
003B0504    /0F85 C61A0000    jnz     003B1FD0 ; 送你去取经
```

## 2. 校验和检查 (Checksum Checking)

### 2.1 算法逻辑
```assembly
; ==============================================================================
;               C h e c k s u m - C h e c k i n g
;
;              外星人也都知道,但是算法叫什么我不清楚(脸红)
; ==============================================================================

003B055D    8B85 18D64100     mov     eax, dword ptr ss:[ebp+41D618]    ; 我想是 CRC protection flag
003B0568    83F8 00           cmp     eax, 0
003B056B    /74 51            je      short 003B05BE    ; 我这里 JUMP
003B05F5    8BBD FCD54100     mov     edi, dword ptr ss:[ebp+41D5FC]    ; 好像是资源段
003B0628    03BD DCD34100     add     edi, dword ptr ss:[ebp+41D3DC]    ; +ImageBase
003B0633    8B8D 00D64100     mov     ecx, dword ptr ss:[ebp+41D600]    ; size
003B063E    83F9 00           cmp     ecx, 0
003B0641    0F84 C5000000     je      003B070C    ; Nothing?
003B064C    33C0              xor     eax, eax
003B0665    33DB              xor     ebx, ebx
003B066C    33D2              xor     edx, edx
003B068A    /8A1F             mov     bl, byte ptr ds:[edi]    ; get a byte
003B0691    | 32D9            xor     bl, cl
003B0698    | 03C3            add     eax, ebx
003B069F    | 47              inc     edi
003B06A5    | 49              dec     ecx
003B06BD    | 83F9 00         cmp     ecx, 0
003B06C0    ^\\75 C3          jnz     short 003B0685    ; 计算校验和
003B06C7    8D9D B5D34100     lea     ebx, dword ptr ss:[ebp+41D3B5]
```

### 2.2 错误处理与信息
```assembly
\---------------------------------------------------------------------------
```
**相关字符串数据：**
```text
003B7AE8  20 20 20 45 72 72 6F 72 20 28 34 29 3A 20 46 69      Error (4): Fi
003B7AF8  6C 65 20 43 52 43 20 45 72 72 6F 72 2C 20 20 20    le CRC Error,
003B7B08  41 62 6F 72 74 21 00                               Abort!.
```
```assembly
\---------------------------------------------------------------------------
003B06E4    3985 04D64100     cmp     dword ptr ss:[ebp+41D604], eax ; 比较校验和
003B06EF    /0F85 DB180000    jnz     003B1FD0                     ; 你敢跳？
```

## 3. Part 5: 注册表与校验

```assembly
; ==============================================================================
[part 5]

003B073A    E8 4A2A0000       call    003B3189    ; 进去观光
```

**调用流程与控制流：**
```assembly
            003B31A5    B8 01000000       mov     eax, 1
            003B31D7    83BD 9ED24100     >cmp    dword ptr ss:[ebp+41D29E], 0
            003B31E3    /0F84 C6120000    je      003B44AF    ; 不需要注册？跳吧========〉下边在极度困倦下走了弯路

            003B31F3    8D85 DD5C4000     lea     eax, dword ptr ss:[ebp+405CDD]; ??
            003B3209    83BD FC1D4200     >cmp    dword ptr ss:[ebp+421DFC], 0
            003B3210    /75 54            jnz     short 003B3266; 不知道是啥,我这里没跳
            003B3217    E8 A51D0000       call    003B4FC1; ?
            003B3249    3C 01             cmp     al, 1
            003B324B    74 14             je      short 003B3261

            003B3270    83BD FC1D4200     >cmp    dword ptr ss:[ebp+421DFC], 2
            003B3277    75 66             jnz     short 003B32DF

            003B32FB    83BD FC1D4200     >cmp    dword ptr ss:[ebp+421DFC], 1
            003B3302    75 37             jnz     short 003B333B

            003B3345    83BD FC1D4200     >cmp    dword ptr ss:[ebp+421DFC], 3
            003B334C    75 26             jnz     short 003B3374

            003B33A6    E8 17270000       call    003B5AC2
```

**子程序 `003B5AC2` 内部逻辑（读取注册标记）：**
```assembly
                                003B5B06    60                pushad
                                003B5B0C    C785 EAD24100     >mov     dword ptr ss:[ebp+41D2EA], 0
                                003B5B32    BB CCD24100       mov     ebx, 41D2CC
                                003B5B3C    03DD              add     ebx, ebp    ; buffer
                                003B5B43    B9 AAD24100       mov     ecx, 41D2AA
                                003B5B4D    03CD              add     ecx, ebp    ; subkey
                                ; 猜想是读注册标记
                                003B5B54    53                push    ebx
                                003B5B55    68 1F000200       push    2001F
                                003B5B5A    6A 00             push    0
                                003B5B5C    51                push    ecx
                                003B5B5D    68 00000080       push    80000000
                                003B5B62    FF95 D2CF4100     call    dword ptr ss:[ebp+41CFD2]        ; advapi32.RegOpenKeyExA
```
**`RegOpenKeyExA` 调用参数细节：**
```text
                                                0012FF68   003B5B68  /CALL to RegOpenKeyExA from 003B5B62
                                                0012FF6C   80000000  |hKey = HKEY_CLASSES_ROOT
                                                0012FF70   003B79DD  |Subkey = "wrifile\\shell\\open\\command\\config"
                                                0012FF74   00000000  |Reserved = 0
                                                0012FF78   0002001F  |Access = KEY_QUERY_VALUE|KEY_SET_VALUE|KEY_CREATE_SUB_KEY|KEY_ENUMERATE_SUB_KEYS|KEY_NOTIFY|20000
```



```
0012FF7C   003B79FF  \\pHandle = 003B79FF

003B5B9A    83F8 00         cmp     eax, 0
003B5B9D    0F84 2A010000   je      003B5CCD    ; JUMP
003B5D09    BE A6D24100     mov     esi, 41D2A6
003B5D13    03F5            add     esi, ebp
003B5D1A    BF A2D24100     mov     edi, 41D2A2
003B5D24    03FD            add     edi, ebp
003B5D30    B8 FECF4100     mov     eax, 41CFFE
003B5D3A    03C5            add     eax, ebp
003B5D53    BB DCD24100     mov     ebx, 41D2DC
003B5D85    03DD            add     ebx, ebp
; 用这么多 regs，气势逼人~~~
003B5D8C    8B8D CCD24100   mov     ecx, dword ptr ss:[ebp+41D2CC]    ; Reg 的句柄
003B5D97    56              push    esi
003B5D98    50              push    eax
003B5D99    57              push    edi
003B5D9A    6A 00           push    0
003B5D9C    53              push    ebx    ; 好奇怪的名字，大概是自动计算的
003B5D9D    51              push    ecx
003B5D9E    FF95 DECF4100   call    dword ptr ss:[ebp+41CFDE]         ; advapi32.RegQueryValueExA
0012FF64   003B5DA4  /CALL to RegQueryValueExA from 003B5D9E
0012FF68   0000006A  |hKey = 6A
0012FF6C   003B7A0F  |ValueName = ""9B,"",80,"",8B,"",80,"?,80,"?,80,"",84,"",81,""
0012FF70   00000000  |Reserved = NULL
0012FF74   003B79D5  |pValueType = 003B79D5
0012FF78   003B7731  |Buffer = 003B7731
0012FF7C   003B79D9  \\pBufSize = 003B79D9
; *** 如果做的 DBPE Cleaner，大概就是清理 HKEY\_CLASSES\_ROOT\\wrifile\\shell\\open\\command\\config\\所有子项目
003B5DC0    66:3D 0000      cmp     ax, 0
003B5DC9   /0F85 EE020000   jnz     003B60BD    ; 做注册部分
003B60C2    8B85 CCD24100   mov     eax, dword ptr ss:[ebp+41D2CC]
003B60CD    50              push    eax
003B60CE    FF95 DACF4100   call    dword ptr ss:[ebp+41CFDA]         ; advapi32.RegCloseKey
003B6106    61              popad
003B611E    8B85 EAD24100   mov     eax, dword ptr ss:[ebp+41D2EA]
003B6129    C3              ret
003B33CC    8B85 7CD24100   mov     eax, dword ptr ss:[ebp+41D27C]
003B33FF    BB 8CD24100     mov     ebx, 41D28C
003B341B    03DD            add     ebx, ebp
003B3422    B9 27354200     mov     ecx, 423527
003B343E    03CD            add     ecx, ebp
003B3445    50              push    eax
003B3446    53              push    ebx
003B3447    51              push    ecx
003B3448    FF95 64CF4100   call    dword ptr ss:[ebp+41CF64]         ; user32.wsprintfA
0012FF64   0000006A
0012FF94   003B344E  /CALL to wsprintfA from 003B3448
0012FF98   003BDC5A  |s = 003BDC5A
0012FF9C   003B79BF  |Format = "%08lX"
0012FFA0   06781F8B  \\<%08lX> = 6781F8B
003B3453    83C4 0C         add     esp, 0C                     ; 平衡堆栈
; ecx ->
-------------------------------------------------------------------
003BDC5D  38 31 46 38 42 00 00 00                             81F8B...
-------------------------------------------------------------------
003B3488    BE 0AD04100     mov     esi, 41D00A
003B34A4    03F5            add     esi, ebp
---------------------------------------------------------------------------
003BDC5A  30 36 37 38 31 46 38 42 00 00 00 00 00 00 00 00  06781F8B........
---------------------------------------------------------------------------
003B34BD    BF 2CFA4100     mov     edi, 41FA2C
003B34EF    03FD            add     edi, ebp
003B34F6    B9 16000000     mov     ecx, 16
003B3500    F3:A4           rep     movsb    ; 传送...不明飞行物
003B3534    BE 27354200     mov     esi, 423527
003B353E    03F5            add     esi, ebp
003B3545    BF 42FA4100     mov     edi, 41FA42
003B354F    03FD            add     edi, ebp
003B3556    B9 0A000000     mov     ecx, 0A
003B3588    F3:A4           rep     movsb
003B3599    E8 A1140000     call    003B4A3F
; 传送某种非 0 的数据
003B4A5B    8DB5 78D04100   lea     esi, dword ptr ss:[ebp+41D078]
003B4A8E    8DBD 0AD04100   lea     edi, dword ptr ss:[ebp+41D00A]
003B4AD8    8A07            mov     al, byte ptr ds:[edi]
003B4AF1    8806            mov     byte ptr ds:[esi], al
003B4AF8    47              inc     edi
```



```
003B4AFE    46              inc     esi  

003B4B2C    3C 00          cmp     al, 0  

003B4B2E    ^\0F85 77FFFFFF jnz     003B4AAB  

003B4B8E    4E              dec     esi  

003B4B94    8DBD 27354200 lea     edi, dword ptr ss:[ebp+423527]  

---------------------------------------------------------------------------  

003BDC5A    30 36 37 38 31 46 38 42 00 00 00 00 00 00 00 00  06781F8B........  

---------------------------------------------------------------------------  

003B4BDE    8A07              mov     al, byte ptr ds:[edi]  

003B4BF7    8806              mov     byte ptr ds:[esi], al  

003B4BFE    47              inc     edi  

003B4C16    46              inc     esi  

003B4C2E    3C 00          cmp     al, 0  

003B4C30    ^\0F85 7BFFFFFF jnz     003B4BB1  

003B4C3B    8806              mov     byte ptr ds:[esi], al  

003B4C6F    8D9D 02D34100 lea     ebx, dword ptr ss:[ebp+41D302]  

003B4C7A    8D85 78D04100 lea     eax, dword ptr ss:[ebp+41D078]  

003B4C85    8BF5              mov     esi, ebp  

003B4CB4    60              pushad  

003B4CBA    53              push    ebx  

003B4CBB    50              push    eax  

003B4CBC    E8 0C810000     call    003BCDCD    ; 奇怪的算法解码出奇怪的信息  

003B4CC6    61              popad  

; 这里的代码把一堆东西拷来拷去，我没耐心也没兴致看算法，不做解说了。  

003B4CF9    8DB5 02D34100 lea     esi, dword ptr ss:[ebp+41D302]  

003B4D04    8B06              mov     eax, dword ptr ds:[esi]  

003B4D0B    3385 9ED24100 xor     eax, dword ptr ss:[ebp+41D29E]  

003B4D16    8985 EED24100 mov     dword ptr ss:[ebp+41D2EE], eax  

003B4D4E    8B46 04         mov     eax, dword ptr ds:[esi+4]  

003B4D56    3385 9AD24100 xor     eax, dword ptr ss:[ebp+41D29A]  

003B4D61    8985 F2D24100 mov     dword ptr ss:[ebp+41D2F2], eax  

003B4D71    8DBD 20D04100 lea     edi, dword ptr ss:[ebp+41D020]  

003B4D7C    8DB5 F6D24100 lea     esi, dword ptr ss:[ebp+41D2F6]  

003B4DDC    8B07              mov     eax, dword ptr ds:[edi]  

003B4DE3    8906              mov     dword ptr ds:[esi], eax  

003B4DEA    8B47 04         mov     eax, dword ptr ds:[edi+4]  

003B4DF2    8946 04         mov     dword ptr ds:[esi+4], eax  

003B4E0C    E8 1B1A0000     call    003B682C    ; 从代码给我的印象来看，像是检查非法字符  

003B4E28    3B85 EED24100 cmp     eax, dword ptr ss:[ebp+41D2EE]  

003B4E45    /0F85 D7000000  jnz     003B4F22  

003B4F39    B8 00000000     mov     eax, 0  

003B4F43    8985 F2D24100 mov     dword ptr ss:[ebp+41D2F2], eax  

003B4F76    8985 EED24100 mov     dword ptr ss:[ebp+41D2EE], eax  

003B4F93    C3              ret  

003B35CB    83F8 01         cmp     eax, 1  

003B35D3    /0F84 D60E0000  je      003B44AF  

003B3622    8D85 DE5C4000 lea     eax, dword ptr ss:[ebp+405CDE]  

003B3660    E8 AF330000     call    003B6A14  

003B6A58    60              pushad  

003B6A5E    8D85 8FC74100 lea     eax, dword ptr ss:[ebp+41C78F]  

003B6A69    6A 00           push    0  

003B6A6B    6A 20           push    20  

003B6A6D    6A 03           push    3  

003B6A6F    6A 00           push    0  

003B6A71    6A 00           push    0  

003B6A73    68 00000080     push    80000000  

003B6A78    50              push    eax  

003B6A79    FF95 DBCE4100 call    dword ptr ss:[ebp+41CEDB]                 ; kernel32.CreateFileA  

                                                                          0012FF60   003B6A7F  /CALL to CreateFileA from 003B6A79  

                                                                          0012FF64   003B6EC2  |FileName = "regdial.dat"  

                                                                          0012FF68   80000000  |Access = GENERIC_READ  

                                                                          0012FF6C   00000000  |ShareMode = 0  

                                                                          0012FF70   00000000  |pSecurity = NULL  

                                                                          0012FF74   00000003  |Mode = OPEN_EXISTING  

                                                                          0012FF78   00000020  |Attributes = ARCHIVE  

                                                                          0012FF7C   00000000  \hTemplateFile = NULL  

; 搞注册界面库，我们到沙罗双树园啦;-)  

003B6A84    83F8 FF         cmp     eax, -1  

003B6A87    /74 2D          je      short 003B6AB6    ; 读取失败？我们给他行行好，翻转 ZF 不跳
```



```
            003B6AA0    50               push    eax  
            003B6AA1    FF95 E3CE4100   call    dword ptr ss:[ebp+41CEE3]           ; kernel32.CloseHandle  

            003B6AAC   /E9 14030000     jmp     003B6DC5  

            003B6DDC    61               popad  

            003B6DE2    B8 01000000     mov     eax, 1  

            003B6DFE    C3               ret  
```

```
    003B36A9    8D85 DF5C4000   lea     eax, dword ptr ss:[ebp+405CDF]    ; 无聊！！！  
    003B36E7    B8 87C74100     mov     eax, 41C787  
    003B3703    03C5            add     eax, ebp  
    003B370A    E8 6F460000     call    003B7D7E  
    ; 用来得到 2 个注册功能的窗口函数 `ShowTryWindow` & `GetRegister`  
```

```
            003B7DAB    60               pushad  
            003B7DB1    8BD8            mov     ebx, eax  
            003B7DC2    807B 08 00      cmp     byte ptr ds:[ebx+8], 0  
            003B7DCB   /0F84 1B030000   je      003B80EC    ; nj  
            003B7DED    8BC3            mov     eax, ebx  
            003B7DF4    83C0 08         add     eax, 8  
            003B7DFC    50               push    eax  
            003B7DFC    50               push    eax  
            003B7DFD    FF95 C13D4200   call    dword ptr ss:[ebp+423DC1]           ; kernel32.LoadLibraryA  
```

```
                                                                                                                                                                                                                                                    0012FF78   003B7E03  /CALL to `LoadLibraryA` from 003B7DFD  
                                                                                                                                                                                                                                                    0012FF7C   003B6EC2  \FileName = "regdial.dat"  
            ; 考，居然是 lib  
            003B7E08    8985 7FCC4100   mov     dword ptr ss:[ebp+41CC7F], eax  
            003B7E13    83F8 00         cmp     eax, 0  
            003B7E16   /75 46           jnz     short 003B7E5E    ; 翻转 ZF  
            003B7EA2    8B33            mov     esi, dword ptr ds:[ebx]               ; `Try.0041C863`  
            003B7EBB    8B7B 04         mov     edi, dword ptr ds:[ebx+4]             ; `Try.0041C881`  
            003B7ED5    03F5            add     esi, ebp  
            003B7EDC    03FD            add     edi, ebp  
            ; 看名字也知道是要干啥  
            ---------------------------------------------------------------------------  
```

```
            003B6F96  53 68 6F 77 54 72 79 57 69 6E 64 6F 77 00 47 65  003B6FA6  
            74 52 65 67 69 73 74 65 72 00 00 00 00 00 00 00  003B6FA6  
            ---------------------------------------------------------------------------  
            003B7EFF    803E 00         cmp     byte ptr ds:[esi], 0  
            003B7F02    0F85 BE000000   jnz     003B7FC6    ; 2 个都得到了？ 为了好看，完成后代码下放  
            003B7FD0    56               push    esi  
            003B7FD1    FFB5 7FCC4100   push    dword ptr ss:[ebp+41CC7F]  
            003B7FD7    FF95 C53D4200   call    dword ptr ss:[ebp+423DC5]           ; kernel32.GetProcAddress  
```

```
                                                                                                                                                                                                                                      0012FF74   003B7FDD  /CALL to `GetProcAddress` from 003B7FD7  
                                                                                                                                                                                                                                      0012FF78   00000000  |hModule = NULL ; 我从中作梗  
                                                                                                                                                                                                                                      0012FF7C   003B6F96  \ProcNameOrOrdinal = "ShowTryWindow"  
            003B7FE2    83F8 00         cmp     eax, 0  
            003B7FE5   /75 30           jnz     short 003B8017    ; 翻转 ZF,j  
            003B8038    8907            mov     dword ptr ds:[edi], eax  
            003B8051    83C7 04         add     edi, 4  
            ; 指向 `GetRegister`  
            003B805E    803E 00         cmp     byte ptr ds:[esi], 0  
            003B8061   /74 0D           je      short 003B8070  
            003B8068    46               inc     esi  
            003B806E  ^\EB EE           jmp     short 003B805E  
            003B809D    46               inc     esi  
            003B80CB  ^\E9 2AFEFFFF     jmp     003B7EFA  
            --------------------------------------------------------------------------  
```

```
            ; 完成以后的代码，接 003B7F02    0F85 BE000000   jnz     003B7FC6  ; 2 个都得到了？ 为了好看，完成后代码下放  
            003B7F35    83C3 08         add     ebx, 8  
            ; 移动指针  
            003B7F65    803B 00         cmp     byte ptr ds:[ebx], 0  
            003B7F68   /74 1F           je      short 003B7F89  
            003B7F81    43               inc     ebx  
            003B7F87  ^\EB DC           jmp     short 003B7F65  
            003B7FB6    43               inc     ebx  
            003B7FBC  ^\E9 FCFDFFFF     jmp     003B7DBD; 回去〉〉〉  
            003B7DC2    807B 08 00      cmp     byte ptr ds:[ebx+8], 0  
            003B7DCB   /0F84 1B030000   je      003B80EC    ; 结束,j  
            003B80F1    61               popad  
            003B80F7    8B85 83CC4100   mov     eax, dword ptr ss:[ebp+41CC83]  
            003B8102    C3               ret  
```

```
    003B373C    B8 FFFFFFFF     mov     eax, -1  
    003B3746    83BD 81C84100 0>cmp     dword ptr ss:[ebp+41C881], 0  
    003B3752   /0F84 570D0000   je      003B44AF; nj  
    003B376F    83BD 85C84100 0>cmp     dword ptr ss:[ebp+41C885], 0  
    003B37A3   /0F84 060D0000   je      003B44AF; 还是 nj  
    003B37D7    E8 060D0000     call    003B44E2  
    003B44E7    8D85 C1A04100   lea     eax, dword ptr ss:[ebp+41A0C1]  
```



### Assembly Code Snippet

```asm
003B44F2  68 C8000000     push    0C8  
003B44F7  50             push    eax    ; buffer  
003B44F8  FF95 BBCE4100  call    dword ptr ss:[ebp+41CEBB]             ; kernel32.GetWindowsDirectoryA  
                                0012FF94   003B44FE  /CALL to GetWindowsDirectoryA from 003B44F8  
                                0012FF98   003B47F4  |Buffer = 003B47F4  
                                0012FF9C   000000C8  \BufSize = C8 (200.)  
003B4503  33C0           xor     eax, eax  
003B451C  B9 C8000000    mov     ecx, 0C8  
003B4526  8DBD C1A04100  lea     edi, dword ptr ss:[ebp+41A0C1]  
003B4531  F2:AE          repne   scas byte ptr es:[edi]   ; 复制目录  
003B4538  C647 FF 5C    mov     byte ptr ds:[edi-1], 5C   ; \  
003B4541  C707 75736572 mov     dword ptr ds:[edi], 72657375   ; user  
003B455E  C747 04 2E64617>mov  dword ptr ds:[edi+4], 7461642E   ; .dat  
003B4592  C647 08 00    mov     byte ptr ds:[edi+8], 0  
003B45C4  8D85 C1A04100 lea     eax, dword ptr ss:[ebp+41A0C1]  
003B45CF  8D9D C1A14100 lea     ebx, dword ptr ss:[ebp+41A1C1]  
003B45DA  53             push    ebx  
003B45DB  50             push    eax  
003B45DC  FF95 B7CE4100 call    dword ptr ss:[ebp+41CEB7]             ; kernel32.FindFirstFileA  
003B45F9  8D9D C1A14100 lea     ebx, dword ptr ss:[ebp+41A1C1]  
003B462C  8D43 14        lea     eax, dword ptr ds:[ebx+14]  
003B4646  8D9D FCA24100 lea     ebx, dword ptr ss:[ebp+41A2FC]  
003B4679  53             push    ebx  
003B467A  50             push    eax  
003B467B  FF95 B3CE4100 call    dword ptr ss:[ebp+41CEB3]             ; kernel32.FileTimeToSystemTime  
                                0012FF94   003B4681  /CALL to `FileTimeToSystemTime` from 003B467B  
                                0012FF98   003B4908  |pFileTime = 003B4908  
                                0012FF9C   003B4A2F  \pSystemTime = 003B4A2F  
003B469D  8D9D FCA24100 lea     ebx, dword ptr ss:[ebp+41A2FC]  
; 计算一个 sum  
003B46A8  33C0           xor     eax, eax  
003B46AF  33D2           xor     edx, edx  
003B46CD  66:8B03        mov     ax, word ptr ds:[ebx]  
003B46E7  B9 6D010000    mov     ecx, 16D  
003B46F1  F7E1           mul     ecx  
003B470A  8985 80D24100 mov     dword ptr ss:[ebp+41D280], eax  
003B4742  33C0           xor     eax, eax  
003B475B  66:8B43 02     mov     ax, word ptr ds:[ebx+2]  
003B4764  B9 1E000000    mov     ecx, 1E  
003B476E  F7E1           mul     ecx  
003B4775  0185 80D24100 add     dword ptr ss:[ebp+41D280], eax  
003B47AD  66:8B43 06     mov     ax, word ptr ds:[ebx+6]  
003B47B6  0185 80D24100 add     dword ptr ss:[ebp+41D280], eax  
003B47EE  C3             ret      ; 班师回朝  
003B380E  C785 88D24100 0>mov  dword ptr ss:[ebp+41D288], 0  
003B381D  C785 84D24100 0>mov  dword ptr ss:[ebp+41D284], 0  
003B3836  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 1  
003B383D  /0F85 10010000 jnz     003B3953   ; nj  
003B3848  FFB5 FECF4100 push    dword ptr ss:[ebp+41CFFE]  
003B384E  8F85 42344200 pop     dword ptr ss:[ebp+423442]  
003B3859  FFB5 E41D4200 push    dword ptr ss:[ebp+421DE4]  
003B385F  8F85 46344200 pop     dword ptr ss:[ebp+423446]  
003B387C  8B9D E41D4200 mov     ebx, dword ptr ss:[ebp+421DE4]  
003B3899  399D FECF4100 cmp     dword ptr ss:[ebp+41CFFE], ebx  
003B389F  /0F83 95000000 jnb     003B393A   ; j  
003B393F  C785 84D24100 0>mov  dword ptr ss:[ebp+41D284], 1  
003B3974  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 2  
003B397B  /0F85 E0000000 jnz     003B3A61   ; j  
003B3A98  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 3  
003B3A9F  0F85 C3000000 jnz     003B3B68  
003B3B9F  33C0           xor     eax, eax  
003B3BA6  83BD 88D24100 0>cmp  dword ptr ss:[ebp+41D288], 1  
003B3BAD  74 0D          je      short 003B3BBC  
003B3BAF  83BD 84D24100 0>cmp  dword ptr ss:[ebp+41D284], 1  
003B3BB6  0F85 F5000000 jnz     003B3CB1  
003B3BD3  8D85 081E4200 lea     eax, dword ptr ss:[ebp+421E08] ; caption  
003B3BDE  8D9D 261E4200 lea     ebx, dword ptr ss:[ebp+421E26] ; http...  
003B3BE9  8D8D 441E4200 lea     ecx, dword ptr ss:[ebp+421E44] ; message  
; 此时发现这两个牛插手了……  
----------------------------------------------------------------------------  
003BC777              20 45 6E 63 72 79 70 72   Encrypt  
003BC787  65 64 20 62 79 20 7A 65 72 30 21 00 00 00 00  ed by zer0!.....  
003BC797  00 00 00 00 00 00 63 6F 64 65 5F 69 6E 6A 65  ......code_injec  
003BC7A7  74 00 00 00 00 00 00 00 00                   t........  
----------------------------------------------------------------------------  
003B3C1C  89A5 74894100 mov     dword ptr ss:[ebp+418974], esp  
003B3C27  FFB5 81C84100 push    dword ptr ss:[ebp+41C881]  
003B3C2D  8F85 F9344200 pop     dword ptr ss:[ebp+4234F9]  
003B3C38  51             push    ecx  
003B3C39  53             push    ebx  
003B3C3A  50             push    eax  
003B3C3B  FFB5 46344200 push    dword ptr ss:[ebp+423446]  
003B3C41  FFB5 42344200 push    dword ptr ss:[ebp+423442]  
003B3C47  E8 39810000    call    003BBD85   ; int3 调用，都 nop  
003B3C7E  8BA5 74894100 mov     esp, dword ptr ss:[ebp+418974]  
003B3D22  83BD 84D24100 0>cmp  dword ptr ss:[ebp+41D284], 1  
003B3D29  74 09          je      short 003B3D34  
003B3D2B  83F8 02        cmp     eax, 2  
003B3D2E  0F85 C0040000 jnz     003B41F4  
003B3D66  8D85 FD344200 lea     eax, dword ptr ss:[ebp+4234FD]  
003B3D99  8D9D 13354200 lea     ebx, dword ptr ss:[ebp+423513]  
003B3DA4  8D8D 27354200 lea     ecx, dword ptr ss:[ebp+423527]  
; 003BDC5A  30 36 37 38 31 46 38 42                      06781F8B  
003B3DAF  8D95 38204200 lea     edx, dword ptr ss:[ebp+422038]  
003B3DF9  89A5 74894100 mov     dword ptr ss:[ebp+418974], esp  
003B3E2C  FFB5 85C84100 push    dword ptr ss:[ebp+41C885]  
003B3E32  8F85 F9344200 pop     dword ptr ss:[ebp+4234F9]  
003B3E4F  52             push    edx  
003B3E50  51             push    ecx  
003B3E51  53             push    ebx  
003B3E52  50             push    eax  
003B3E53  E8 2D7F0000    call    003BBD85; 还是他，nop  
003B3E74  8BA5 74894100 mov     esp, dword ptr ss:[ebp+418974]  
003B3E91  83F8 02        cmp     eax, 2  
```



## 003B3E99 - 003B3F41

```
003B3E99    B8 00000000     mov     eax, 0  
003B3EA3   /0F84 C4040000   je      003B436D  
003B3EDB    B9 2A000000     mov     ecx, 2A  
003B3EE5    BF 0AD04100     mov     edi, 41D00A  
003B3EEF    03FD            add     edi, ebp  
003B3F08    BE FD344200     mov     esi, 4234FD  
003B3F3A    03F5            add     esi, ebp  
003B3F41    F3:A4           rep     movs byte ptr es:[edi], byte ptr ds:[esi]  
```

## Part 6

; ================================================================= 

; 返回到 `Ch e c k s u m - C h e c k i n g` 之后的位置

; 有个稍微快一点的方法解决注册

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

```
003B073A    E8 4A2A0000     call    003B3189 ; nop 掉然后置 eax =1 即可  
```

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

```
003B076C    66:3D 0100      cmp     ax, 1  
003B0775   /0F84 E9000000   je      003B0864    ; j,否则 Over  
003B087B    8D85 D85C4000   lea     eax, dword ptr ss:[ebp+405CD8]  
003B08F8    BE E4D34100     mov     esi, 41D3E4  
003B0902    03F5            add     esi, ebp    ; esi -> 块表  
```

; `esi:`

;-----------------------

```
003B7B17  00001000    ; VOffset  
003B7B1B  00006000    ; RSize  
003B7B1F  0000F342    ; VSize  
003B7B23  E0000021    ; Flags  
003B7B27  00017000   ; 一样....  
003B7B2B  00001000  
003B7B2F  00000FB8  
003B7B33  C0000041  
```

;-----------------------

```
003B090E    833E 00         cmp     dword ptr ds:[esi], 0    ; 区块解码结束了?  
003B0916   /0F84 7E060000   je      003B0F9A                   ; 结束就走人  
003B0921    8B9D DCD34100   mov     ebx, dword ptr ss:[ebp+41D3DC]   ; ImageBase  
003B092C    031E            add     ebx, dword ptr ds:[esi]    ; ebx -> 指向一个 Section  
003B0945    8B4E 04         mov     ecx, dword ptr ds:[esi+4]    ; ecx -> Size  
003B0952    83F9 00         cmp     ecx, 0  
003B0955   /75 4E           jnz     short 003B09A5    ; 长度非零,处理--->  
```

(((((((((((((((((((((((((((((((((((((((((((((((((((

; 跳过处理大小为零的 Section

```
003B096E    83C6 10         add     esi, 10  
003B0976  ^\EB 91          jmp     short 003B0909  
```

)))))))))))))))))))))))))))))))))))))))))))))))))

```
003B09EE    D1E9            shr     ecx, 1    ; ecx /2  
003B09F5    66:8B85 2DD6410>mov     ax, word ptr ss:[ebp+41D62D]  
003B0A01    66:35 2111      xor     ax, 1121  
003B0A37    66:C1C8 02      ror     ax, 2  
003B0A40    66:05 1A00      add     ax, 1A  
003B0A49    66:05 A100      add     ax, 0A1  
```

; 计算 key

```
003B0A84    66:3103         xor     word ptr ds:[ebx], ax  
003B0A8C    66:48           dec     ax  
003B0A93    43              inc     ebx                     ; 指向下一 byte  
003B0A99    43              inc     ebx                     ; 移动指针  
003B0AC7    49              dec     ecx  
003B0ACD    83F9 00         cmp     ecx, 0  
003B0AD0  ^\75 AD          jnz     short 003B0A7F  
```

; 循环解码

```
003B0B09    8B46 0C         mov     eax, dword ptr ds:[esi+C]    ; flag  
003B0B11    83E0 01         and     eax, 1    ; 隐藏着什么标记呢?  
003B0B30    83BD F81D4200 0>cmp     dword ptr ss:[ebp+421DF8], 1  
003B0B37   /0F85 60030000   jnz     003B0E9D    ; 不知道,nj  
003B0B6A    83F8 01         cmp     eax, 1  
003B0B6D    0F85 25030000   jnz     003B0E98; 还是不知道,nj,$$#@%@#$  
```

```
003B0C18    60              pushad  
003B0C23    8B46 04         mov     eax, dword ptr ds:[esi+4]; RSize  
003B0C2B    83F8 00         cmp     eax, 0  
003B0C33   /0F84 03020000   je      003B0E3C    ; RSize==0 就不分配空间了  
003B0C43    8B46 08         mov     eax, dword ptr ds:[esi+8]    ;VSize  
003B0C73    6A 04           push    4  
003B0C75    68 00100000     push    1000  
003B0C7A    50              push    eax  
003B0C7B    6A 00           push    0  
003B0C7D    FF95 13CF4100   call    dword ptr ss:[ebp+41CF13]        ; 分配空间存放 Section  
```

```
            0012FF74   003B0C83  /CALL to `VirtualAlloc` from 003B0C7D  
            0012FF78   00000000  |Address = NULL  
            0012FF7C   0000F342  |Size = F342 (62274.)  
            0012FF80   00001000  |AllocationType = MEM_COMMIT  
            0012FF84   00000004  \Protect = PAGE_READWRITE  
```

```
003B0CC7    8985 E0D34100   mov     dword ptr ss:[ebp+41D3E0], eax ; 得到的空间  
003B0CFF    56              push    esi    ; 保存 Section Table Pointer  
003B0D32    8B1E            mov     ebx, dword ptr ds:[esi]    ; ebx -> VOffset  
003B0D39    039D DCD34100   add     ebx, dword ptr ss:[ebp+41D3DC]   ; +ImageBase, 得到 VA  
003B0D44    50              push    eax  
003B0D45    53              push    ebx  
003B0D46    E8 AE620000     call    003B6FF9    ; aplib_depack_asm.解压缩到刚才 VirtualAlloc 得到的空间  
003B0D50    83C4 08         add     esp, 8                   ; 平衡堆栈，大概用的不是 stdcall  
003B0D5D    8BC8            mov     ecx, eax    ; 解压出来的长度  
003B0D64    8B3E            mov     edi, dword ptr ds:[esi]    ; edi -> VOffset  
003B0D93    03BD DCD34100   add     edi, dword ptr ss:[ebp+41D3DC]   ; +ImageBase, Get VA  
```

; 要把解压得代码传送回去了

```
003B0DC6    8BB5 E0D34100   mov     esi, dword ptr ss:[ebp+41D3E0]; 解压缩后的数据  
003B0DE3    F3:A4           rep     movs byte ptr es:[edi], byte ptr ds:[esi]     ; copy all  
003B0DEF    5E              pop     esi                                             ; 恢复 Section Table Pointer  
```

; 这里有点无聊。根本没有修改 esi 也没用用它，最后都 popad

```
003B0E0C    8B85 E0D34100   mov     eax, dword ptr ss:[ebp+41D3E0]    ; 申请到的地址  
003B0E29    68 00800000     push    8000  
003B0E2E    6A 00           push    0  
003B0E30    50              push    eax  
003B0E31    FF95 17CF4100   call    dword ptr ss:[ebp+41CF17]         ; kernel32.VirtualFree  
```

; 满门抄斩……55555555

```
003B0E53    61              popad  
```

; -------- 修正页面访问权限 ----------

```
003B0EAC    60              pushad  
003B0EB2    8B9D DCD34100   mov     ebx, dword ptr ss:[ebp+41D3DC]            ; ImageBase  
003B0ECF    031E            add     ebx, dword ptr ds:[esi]    ; 块表 VA  
003B0EE8    8B4E 04         mov     ecx, dword ptr ds:[esi+4]    ; 长度  
003B0EF5    B8 74894100     mov     eax, 418974  
003B0EFF    03C5            add     eax, ebp    ; buffer for VirtualProtect  
003B0F2E    50              push    eax  
003B0F2F    6A 04           push    4  
003B0F31    51              push    ecx  
003B0F32    53              push    ebx  
003B0F33    FF95 27CF4100   call    dword ptr ss:[ebp+41CF27]         ; kernel32.VirtualProtect  
```

```
            0012FF74   003B0F39  /CALL to `VirtualProtect` from 003B0F33  
            0012FF78   00401000  |Address = Try.00401000  
            0012FF7C   00006000  |Size = 6000 (24576.)  
            0012FF80   00000004  |NewProtect = PAGE_READWRITE  
            0012FF84   003B30A7  \pOldProtect = 003B30A7  
```

```
003B0F3E    61              popad  
```

; -------------- 移动指针，到下一个 Section ------------------

```
003B0F83    83C6 10         add     esi, 10  
003B0F8B  ^\E9 79F9FFFF     jmp     003B0909    ; 循环直到所有 Sections 都还原  
```

; ------------------------------------

; 循环完成到这里，这里其实可以 Dump 了：

```
003B0F9A    90              nop  
003B0FAE    8D85 D95C4000   lea     eax, dword ptr ss:[ebp+405CD9] ; 这东西很无聊，碰到好几次，猜不出来，大概是 buffer  
```

![](img/620448a77d884bd6b2c4a7cfef437501.jpg) ![](img/620448a77d884bd6b2c4a7cfef437501.jpg)

## Part 7: 输入表处理

; ************************************************************

;               阿 赖 耶 识 —— 输 入 表 处 理 觉醒




#### 逆向工程分析

**003B0FC4** `E8 34140000` **call** `003B23FD` ; 当然要进去了

**003B242F** `60` **pushad**

**003B2474** `FF95 1BCF4100` **call** `dword ptr ss:[ebp+41CF1B]` ; `kernel32.GetCurrentProcessId`

**003B24A8** `8985 20FA4100` **mov** `dword ptr ss:[ebp+41FA20], eax` ; save process Id

**003B24B3** `8B85 1BCF4100` **mov** `eax, dword ptr ss:[ebp+41CF1B]` ; `kernel32.GetCurrentProcessId`

**003B24D0** `8985 24FA4100` **mov** `dword ptr ss:[ebp+41FA24], eax` ; `kernel32.GetCurrentProcessId`

; 不知道搞什么名堂

**003B2508** `B8 B8FA4100` **mov** `eax, 41FAB8`

**003B2512** `03C5` **add** `eax, ebp`

**003B2519** `8985 ACDC4100` **mov** `dword ptr ss:[ebp+41DCAC], eax`

**003B254C** `8B9D 08D64100` **mov** `ebx, dword ptr ss:[ebp+41D608]` ; Import Table RVA

-------------------

`0041F023 78750000`

`0041F027 00000000`

`0041F02B 00000000`

`0041F02F 82750000`

`0041F033 7D910000`

`0041F037 25350000`

`............`

-------------------

**003B2557** `83FB 00` **cmp** `ebx, 0` ; 没有 import 么？

**003B2587** `/0F84 1F0A0000` **je** `003B2FAC` ; 当然有，不跳

**003B25D1** `039D DCD34100` **add** `ebx, dword ptr ss:[ebp+41D3DC]` ; +ImageBase,得到 IT VA

; `ebx` -> `IID(s)`

**003B25E6** `8B43 0C` **mov** `eax, dword ptr ds:[ebx+C]` ; pointer to DLL asciz name

**003B25EE** `83F8 00` **cmp** `eax, 0`

**003B25F6** `/0F84 B0090000` **je** `003B2FAC` ; Game Over?

**003B2601** `53` **push** `ebx` ; Try.0041F023

**003B262F** `51` **push** `ecx`

**003B2647** `52` **push** `edx`

**003B2648** `33D2` **xor** `edx, edx`

**003B264F** `B9 20000000` **mov** `ecx, 20`

**003B2654** `33DB` **xor** `ebx, ebx` ; Try.0041F023

**003B2656** `D1F8` **sar** `eax, 1`

**003B265D** `0F92C3` **setb** `bl`

**003B2660** `D3E3` **shl** `ebx, cl`

**003B2667** `03D3` **add** `edx, ebx`

**003B2669** `^\\E2 E9` **loopd** `short 003B2654`

; DLL Name 的 RVA 解码,结果在 `edx` 输出

**003B266B** `8BC2` **mov** `eax, edx` ; `eax = edx = dll name rva`

**003B2684** `5A` **pop** `edx`

**003B2685** `59` **pop** `ecx`

**003B268B** `5B` **pop** `ebx`

**003B2696** `0385 DCD34100` **add** `eax, dword ptr ss:[ebp+41D3DC]` ; +ImageBase = VA

**003B26C9** `8BF0` **mov** `esi, eax`

**003B26D5** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 0`

; -------------- 是否是特殊 DLL, 包括 Windows `Kernel32` & `User32`, 与 VB 的 `MSVB` ------------------------

**003B2709** `50` **push** `eax`

**003B270F** `8B00` **mov** `eax, dword ptr ds:[eax]`; 取 dll name 开头 4 个字节

**003B2716** `25 DFDFDFDF` **and** `eax, DFDFDFDF`; 转换为大写

**003B2720** `3D 4B45524E` **cmp** `eax, 4E52454K` ; `KERN****`

**003B2725** `74 07` **je** `short 003B272E`

**003B2727** `3D 55534552` **cmp** `eax, 52455355` ; `USER****`

**003B272C** `75 11` **jnz** `short 003B273F`

**003B2733** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 1` ; 特殊 dll 标记

**003B2744** `58` **pop** `eax`

**003B2777** `50` **push** `eax` ; Try.00415C82

**003B277D** `8B00` **mov** `eax, dword ptr ds:[eax]`

**003B2784** `25 DFDFDFDF` **and** `eax, DFDFDFDF`

**003B278E** `3D 4D535642` **cmp** `eax, 4256534D` ; `MSVB****`

**003B2793** `75 11` **jnz** `short 003B27A6`

**003B279A** `C685 78894100 0>` **mov** `byte ptr ss:[ebp+418978], 1` ; 特殊 dll 标记

**003B27D3** `58` **pop** `eax` ; Try.00415C82

; 为何不一起判断? D.boy 大概写到这里喝多了:0

; ---------------------------------------------------------------------------

**003B27DE** `50` **push** `eax`

**003B27DF** `FF95 C13D4200` **call** `dword ptr ss:[ebp+423DC1]` ; `kernel32.LoadLibraryA`

; 载入 dll

**003B27EA** `8985 7FCC4100` **mov** `dword ptr ss:[ebp+41CC7F], eax` ; 保存 dll module base

**003B2822** `33C0` **xor** `eax, eax`

**003B2851** `8703` **xchg** `dword ptr ds:[ebx], eax` ; 清掉 IID 的 OriginalFirstThunk,然后把它读到 `eax`

; 如果改为 `mov eax, dword ptr [ebx]`可以避免...我偷懒,不改了,下面也是这样

**003B2858** `53` **push** `ebx`

**003B2886** `51` **push** `ecx`

**003B289E** `52` **push** `edx`

**003B289F** `33D2` **xor** `edx, edx`

**003B28A6** `B9 20000000` **mov** `ecx, 20`

**003B28AB** `33DB` **xor** `ebx, ebx`

**003B28AD** `D1F8` **sar** `eax, 1`

**003B28B4** `0F92C3` **setb** `bl`

**003B28B7** `D3E3` **shl** `ebx, cl`

**003B28BE** `03D3` **add** `edx, ebx`

**003B28C0** `^\\E2 E9` **loopd** `short 003B28AB`

**003B28C2** `8BC2` **mov** `eax, edx`

**003B28DB** `5A` **pop** `edx`

**003B28DC** `59` **pop** `ecx`

**003B28E2** `5B` **pop** `ebx`

**003B28E8** `8BF0` **mov** `esi, eax`

; OriginalFirstThunk 解码 -> `esi`

**003B292E** `33C0` **xor** `eax, eax`

**003B295D** `8743 10` **xchg** `dword ptr ds:[ebx+10], eax` ; `eax -> FirstThunk, & erase rva`

**003B2965** `53` **push** `ebx`

**003B2993** `51` **push** `ecx`

**003B29AB** `52` **push** `edx`

**003B29AC** `33D2` **xor** `edx, edx`

**003B29B3** `B9 20000000` **mov** `ecx, 20`

**003B29B8** `33DB` **xor** `ebx, ebx`

**003B29BA** `D1F8` **sar** `eax, 1`

**003B29C1** `0F92C3` **setb** `bl`

**003B29C4** `D3E3` **shl** `ebx, cl`

**003B29CB** `03D3` **add** `edx, ebx`

**003B29CD** `^\\E2 E9` **loopd** `short 003B29B8`

**003B29CF** `8BC2` **mov** `eax, edx`

**003B29E8** `5A` **pop** `edx`

**003B29E9** `59` **pop** `ecx`

**003B29EF** `5B` **pop** `ebx`

**003B29F5** `8BF8` **mov** `edi, eax`

; FirstThunk 解码 -> `edi`

**003B2A01** `83FE 00` **cmp** `esi, 0` ; OriginalFirstThunk 不能用?

**003B2A04** `/75 34` **jnz** `short 003B2A3A`; 能就走

**003B2A33** `8BF7` **mov** `esi, edi` ; 它不行就使用第二个表 FirstThunk

**003B2A67** `03B5 DCD34100` **add** `esi, dword ptr ss:[ebp+41D3DC]` ; +ImageBase,Get VA



`003B2A72` `03BD DCD34100` `add` `edi, dword ptr ss:[ebp+41D3DC]` `+ImageBase,Get VA`
`; offset to function name ( and hint)`
`003B2A99` `8B06` `mov` `eax, dword ptr ds:[esi]` `; 要 do hint...`
`003B2AA0` `83F8 00` `cmp` `eax, 0`
`003B2AA3` `/75 29` `jnz` `short 003B2ACE` `; 未完?`
`003B2ABC` `83C3 14` `add` `ebx, 14`
`003B2AC4` `^\\E9 13FBFFFF` `jmp` `003B25DC` `; 下一个 IID`
`003B2B00` `807E 03 80` `cmp` `byte ptr ds:[esi+3], 80` `; is imported by ordinal?`
`003B2B04` `75 6C` `jnz` `short 003B2B72`
`; ord...`
`003B2B0B` `33C0` `xor` `eax, eax`
`003B2B12` `66:8706` `xchg` `word ptr ds:[esi], ax`
`003B2B42` `50` `push` `eax`
`003B2B43` `FFB5 7FCC4100` `push` `dword ptr ss:[ebp+41CC7F]`
`003B2B49` `FF95 C53D4200` `call` `dword ptr ss:[ebp+423DC5]`
`003B2B54` `8BC8` `mov` `ecx, eax`
`003B2B6D` `/E9 84000000` `jmp` `003B2BF6`
`; str...`
`003B2B77` `33C0` `xor` `eax, eax`
`003B2B7E` `8706` `xchg` `dword ptr ds:[esi], eax`
`003B2B85` `0385 DCD34100` `add` `eax, dword ptr ss:[ebp+41D3DC]` `+ImageBase,这句话打过无数次了,累!!!`
`003B2BA2` `83C0 02` `add` `eax, 2` `; 跳过 hint 值(sizeof word == 2)`
`003B2BD2` `50` `push` `eax`
`003B2BD3` `FFB5 7FCC4100` `push` `dword ptr ss:[ebp+41CC7F]`
`003B2BD9` `FF95 C53D4200` `call` `dword ptr ss:[ebp+423DC5]` `kernel32.GetProcAddress`
`003B2C00` `80BD 78894100 0>cmp` `byte ptr ss:[ebp+418978], 1` `; 特殊函数?`
`003B2C07` `/0F85 87020000` `jnz` `003B2E94` `; magic jump...改为 jmp 吧,省得心烦`
`; --------------------------- 加密输入表 ----------------------------------------`
`; 具体不说了,跟到这里有点筋疲力尽了`
`003B2C3F` `60` `pushad`
`003B2C45` `8BF8` `mov` `edi, eax`
`003B2C63` `8B85 A7DC4100` `mov` `eax, dword ptr ss:[ebp+41DCA7]`
`003B2C6E` `3D F4010000` `cmp` `eax, 1F4`
`003B2C73` `/75 1A` `jnz` `short 003B2C8F`
`003B2C7A` `89BD 70894100` `mov` `dword ptr ss:[ebp+418970], edi`
`003B2C85` `/E9 C9010000` `jmp` `003B2E53`
`003B2CC1` `B9 04000000` `mov` `ecx, 4`
`003B2CCB` `33D2` `xor` `edx, edx`
`003B2CD2` `F7E1` `mul` `ecx`
`003B2CD9` `BE B1DC4100` `mov` `esi, 41DCB1`
`003B2CE3` `03F5` `add` `esi, ebp`
`003B2CFC` `33BD 20FA4100` `xor` `edi, dword ptr ss:[ebp+41FA20]`
`003B2D07` `893C06` `mov` `dword ptr ds:[esi+eax], edi`
`003B2D26` `8B85 A7DC4100` `mov` `eax, dword ptr ss:[ebp+41DCA7]`
`003B2D31` `B9 0B000000` `mov` `ecx, 0B`
`003B2D3B` `33D2` `xor` `edx, edx`
`003B2D42` `F7E1` `mul` `ecx`
`003B2D49` `BF 85E44100` `mov` `edi, 41E485`
`003B2D7B` `03FD` `add` `edi, ebp`
`003B2DAA` `03F8` `add` `edi, eax`
`003B2DB6` `89BD 70894100` `mov` `dword ptr ss:[ebp+418970], edi`
`003B2DEE` `BE A6DC4100` `mov` `esi, 41DCA6`
`003B2DF8` `03F5` `add` `esi, ebp`
`003B2E27` `B9 0B000000` `mov` `ecx, 0B`
`003B2E31` `F3:A4` `rep movs` `byte ptr es:[edi], byte ptr ds:[esi]`
`003B2E38` `FF85 A7DC4100` `inc` `dword ptr ss:[ebp+41DCA7]` `; 计数器,被加密的应该是 push count...push address...ret`
`003B2E43` `8BBD 70894100` `mov` `edi, dword ptr ss:[ebp+418970]`
`003B2E6A` `61` `popad`
`003B2E70` `8B8D 70894100` `mov` `ecx, dword ptr ss:[ebp+418970]`
`003B2E92` `/EB 1E` `jmp` `short 003B2EB2`
`; -----------------------------------------------------------------------------`
`003B2EAB` `8BC8` `mov` `ecx, eax` `; NETAPI32.Netbios`
`; 函数地址`
`003B2ECE` `39BD 33D64100` `cmp` `dword ptr ss:[ebp+41D633], edi`
`003B2ED4` `76 10` `jbe` `short 003B2EE6` `; 不知道,没跳`
`003B2EDB` `89BD 33D64100` `mov` `dword ptr ss:[ebp+41D633], edi`
`003B2EEB` `39BD 2FD64100` `cmp` `dword ptr ss:[ebp+41D62F], edi`
`003B2EF1` `73 10` `jnb` `short 003B2F03` `; 不知道,还是没跳`
`003B2EF8` `89BD 2FD64100` `mov` `dword ptr ss:[ebp+41D62F], edi`
`003B2F24` `890F` `mov` `dword ptr ds:[edi], ecx` `; 填充 IAT`
`003B2F3D` `83C6 04` `add` `esi, 4`
`003B2F45` `83C7 04` `add` `edi, 4`
`; 移动 Thunk 指针`
`003B2F4D` `^\\E9 42FBFFFF` `jmp` `003B2A94` `; 循环---`
`; ---------------------------------------------------------------------------`
`; 从老上边的 003B25F6 /0F84 B0090000 je 003B2FAC 出来,完成 patch iat`
`003B2FB1` `E8 16000000` `call` `003B2FCC` `; 取 User32.GetClassNameA...居然专门弄个 call...`
`003B2FBB` `61` `popad` `; 解放了`
`003B2FC1` `C3` `ret`

[part 8]
`img/620448a77d884bd6b2c4a7cfef437501.jpg`

`; --------------------------- 不明飞行物 -----------------------------------`
`003B0FD3` `8D85 DA5C4000` `lea` `eax, dword ptr ss:[ebp+405CDA]`
`003B1011` `E8 96200000` `call` `003B30AC` `; 没看懂`

`; 处理 调用 表 & 跳转 表 ( 还原 指针 )`

`; --------------call dword ptr ds:[xxxxxxxx]，其中 xxxxxxxx 是 80xxxxxxxx----------------`
`003B108C` `60` `pushad`
`003B10A4` `8B9D 08D64100` `mov` `ebx, dword ptr ss:[ebp+41D608]`
`003B10AF` `039D DCD34100` `add` `ebx, dword ptr ss:[ebp+41D3DC]` `; 还是 Imagebase`
`003B110F` `BE E4D34100` `mov` `esi, 41D3E4` `; 块表`
`003B1119` `03F5` `add` `esi, ebp`
`003B1120` `8B9D DCD34100` `mov` `ebx, dword ptr ss:[ebp+41D3DC]` `; Try.00400000`
`003B112B` `031E` `add` `ebx, dword ptr ds:[esi]`
`; ebx -> a section,应该是代码段`
`003B1132` `8BFB` `mov` `edi, ebx` `; Try.00401000`
`003B1139` `8B4E 08` `mov` `ecx, dword ptr ds:[esi+8]` `; VSize`
`003B1141` `83F9 00` `cmp` `ecx, 0`



# 反汇编代码分析

## 主循环与指针还原

```
003B115B   /0F84 93050000   je      003B16F4     ; 0?不玩了  

003B1178    D1E9             shr     ecx, 1                         ;  ecx/2  

003B11A7    49               dec     ecx  

003B11AD    66:B8 FF15       mov     ax, 15FF                       ;  call    dword ptr ds:\[xxxxxxxx\]  

003B11CD    BE 31354200      mov     esi, 423531  

003B11FF    03F5             add     esi, ebp  

003B124A    F2:66:AF         repne   scas word ptr es:\[edi\]   ; 寻找 call    dword ptr ds:\[xxxxxxxx\]  

; forgot:如果没有就……不好玩了，看 `Ding Boy` 怎么收场  

; `edi -> xxxxxxxx`,而不是 `call dword ptr`  

003B1264    8B1F             mov     ebx, dword ptr ds:\[edi\]; 要 `call` 的地址->Ebx  

003B127D    81E3 00000080    and     ebx, 80000000    ; 取高位  

003B129A    81FB 00000080    cmp     ebx, 80000000    ; 是 80xxxxxxx 的形势？  

003B12A0   /0F85 13010000    jnz     003B13B9            ; 不是算了  

003B12AB    8B1F             mov     ebx, dword ptr ds:\[edi\]; 加密过的指针  

003B12B2    81E3 FFFFFFFF7F  and     ebx, 7FFFFFFF    ; 去掉 31st bit,还原指针  

003B12BD    3B9D 33D64100    cmp     ebx, dword ptr ss:\[ebp+41D633\]  

003B12C3    0F82 C3000000    jb      003B138C    ; 判断地址范围  

003B12CE    3B9D 2FD64100    cmp     ebx, dword ptr ss:\[ebp+41D62F\]                 ; `Try.0041F01B`  

003B12D4   /0F87 AD000000    ja      003B1387  

003B12DF    833E FF          cmp     dword ptr ds:\[esi\], -1    ; ???  

003B130F   /0F84 DF030000    je      003B16F4  

003B131A    8B1B             mov     ebx, dword ptr ds:\[ebx\]                        ; 把指针指向的内容读出来  

003B1321    891E             mov     dword ptr ds:\[esi\], ebx  

; 写到 `esi`  

003B1350    8937             mov     dword ptr ds:\[edi\], esi    ; 地址重定位,nop 掉,下同  

003B1357    83C6 04         add     esi, 4    ; 移动指针  

003B13BE    83F9 00         cmp     ecx, 0  

003B13C1  ^ 0F85 56FEFFFF    jnz     003B121D    ; 循环直到所有都搞定  
```

## `jmp` 指令处理

```
; ----------------------- jmp     dword ptr ds:\[xxxxxxxx\]----------------------------------  

; 方法如出一辙，不多解释  

003B140B    56               push    esi;剩下的 buffer  

003B1416    8B9D 08D64100    mov     ebx, dword ptr ss:\[ebp+41D608\]  

003B1449    039D DCD34100    add     ebx, dword ptr ss:\[ebp+41D3DC\]                 ; `Try.00400000`  

003B1459    BE E4D34100      mov     esi, 41D3E4  

003B148B    03F5             add     esi, ebp  

003B1492    8B9D DCD34100    mov     ebx, dword ptr ss:\[ebp+41D3DC\]                 ; `Try.00400000`  

003B149D    031E             add     ebx, dword ptr ds:\[esi\]  

003B14A4    8BFB             mov     edi, ebx                                        ; `Try.00401000`  

003B14AB    8B4E 08         mov     ecx, dword ptr ds:\[esi+8\]  

003B14C5    D1E9             shr     ecx, 1  

003B14DE    49               dec     ecx  

003B150C    66:B8 FF25       mov     ax, 25FF    ;   jmp     dword ptr ds:\[xxxxxxxx\]  

003B151A    5E               pop     esi  

003B152A    F2:66:AF         repne   scas word ptr es:\[edi\]  

003B1532    8B1F             mov     ebx, dword ptr ds:\[edi\]  

003B1539    81E3 00000080    and     ebx, 80000000  

003B1556    81FB 00000080    cmp     ebx, 80000000  

003B155C   /0F85 5B010000    jnz     003B16BD  

003B158F    8B1F             mov     ebx, dword ptr ds:\[edi\]  

003B15BE    81E3 FFFFFFFF7F  and     ebx, 7FFFFFFF  

003B15F1    3B9D 33D64100    cmp     ebx, dword ptr ss:\[ebp+41D633\]                 ; `Try.00411000`  

003B15F7   /0F82 A9000000    jb      003B16A6  

003B1614    3B9D 2FD64100    cmp     ebx, dword ptr ss:\[ebp+41D62F\]                 ; `Try.0041F01B`  

003B161A   /77 5D            ja      short 003B1679  

003B1621    833E FF          cmp     dword ptr ds:\[esi\], -1  

003B1629   /0F84 C5000000    je      003B16F4  

003B1634    8B1B             mov     ebx, dword ptr ds:\[ebx\]  

003B163B    891E             mov     dword ptr ds:\[esi\], ebx  

003B166A    8937             mov     dword ptr ds:\[edi\], esi  

003B166A    8937             mov     dword ptr ds:\[edi\], esi  

003B16D4    83F9 00         cmp     ecx, 0  

003B16D7  ^ 0F85 48FEFFFF    jnz     003B1525    ; 循环处理所有  

003B170B    61               popad     ; 文革结束啦![](img/620448a77d884bd6b2c4a7cfef437501.jpg) 

\[最后的战役\]:) 
```

## 最后的战役：恢复内核函数指针

```
; ------------------------------------------------------最后的战役  

003B174E    8B85 DBCE4100    mov     eax, dword ptr ss:\[ebp+41CEDB\]                 ; `kernel32.CreateFileA`  

003B1759    8985 A0FA4100    mov     dword ptr ss:\[ebp+41FAA0\], eax                 ; `kernel32.CreateFileA`  

003B177B    8B85 CFCE4100    mov     eax, dword ptr ss:\[ebp+41CECF\]                 ; `kernel32.ReadFile`  

003B1786    8985 A4FA4100    mov     dword ptr ss:\[ebp+41FAA4\], eax                 ; `kernel32.ReadFile`  

003B1796    8B85 DFCE4100    mov     eax, dword ptr ss:\[ebp+41CEDF\]                 ; `kernel32.WriteFile`  

003B17A1    8985 A8FA4100    mov     dword ptr ss:\[ebp+41FAA8\], eax                 ; `kernel32.WriteFile`  

003B17D9    8B85 CBCE4100    mov     eax, dword ptr ss:\[ebp+41CECB\]                 ; `kernel32.SetFilePointer`  

003B17F6    8985 ACFA4100    mov     dword ptr ss:\[ebp+41FAAC\], eax                 ; `kernel32.SetFilePointer`  

003B182E    8B85 E3CE4100    mov     eax, dword ptr ss:\[ebp+41CEE3\]                 ; `kernel32.CloseHandle`  

003B184B    8985 B0FA4100    mov     dword ptr ss:\[ebp+41FAB0\], eax                 ; `kernel32.CloseHandle`  

003B1895    8B85 EFCE4100    mov     eax, dword ptr ss:\[ebp+41CEEF\]                 ; `kernel32.DeleteFileA`  

003B18A0    8985 B4FA4100    mov     dword ptr ss:\[ebp+41FAB4\], eax                 ; `kernel32.DeleteFileA`  

003B18F4    8D85 DC5C4000    lea     eax, dword ptr ss:\[ebp+405CDC\]  

; 最后来个 `int3`,还原中断向量  

003B1949    60               pushad  

003B194F    B8 534E5552      mov     eax, 52554E53  

003B1959    BB 0ECA4100      mov     ebx, 41CA0E  

003B1975    03DD             add     ebx, ebp  

003B197C    CC               int3; 收尾  

003B1982    61               popad  

003B198D    80BD F0344200 0>cmp     byte ptr ss:\[ebp+4234F0\], 1    ; `NT?` 释放驱动，我们用来夺取 `ring0` 的嘛  

003B1994   /0F85 93010000    jnz     003B1B2D  

003B19C7    8D85 E35C4000    lea     eax, dword ptr ss:\[ebp+405CE3\]; 垃圾  

003B1A37    FFB5 546C4000    push    dword ptr ss:\[ebp+406C54\]    ; 驱动句柄  

003B1A3D    FF95 50724000    call    dword ptr ss:\[ebp+407250\]                 ; `kernel32.CloseHandle`  

003B1A89    FFB5 586C4000    push    dword ptr ss:\[ebp+406C58\] ; 服务  

003B1A8F    FF95 DD724000    call    dword ptr ss:\[ebp+4072DD\]                 ; `advapi32.CloseServiceHandle`  

003B1AEF    FFB5 3A6C4000    push    dword ptr ss:\[ebp+406C3A\]  

003B1AF5    FF95 DD724000    call    dword ptr ss:\[ebp+4072DD\]                 ; `advapi32.CloseServiceHandle`  

; 解密 `OEP`~~~  

003B1B5F    8B85 F8D54100    mov     eax, dword ptr ss:\[ebp+41D5F8\]  

003B1B81    53               push    ebx                                          ; `Try.00418000`  

003B1BAF    51               push    ecx                                          ; `advapi32.77DA214E`  

003B1BC7    52               push    edx  

003B1BC8    33D2             xor     edx, edx  

003B1BCF    B9 20000000      mov     ecx, 20  

003B1BD4    33DB             xor     ebx, ebx                                       ; `Try.00418000`  

003B1BD6    D1F8             sar     eax, 1  

003B1BDD    0F92C3           setb    bl  

003B1BE0    D3E3             shl     ebx, cl  

003B1BE7    03D3             add     edx, ebx  

003B1BE9  ^\\E2 E9           loopd   short 003B1BD4  

003B1BEB    8BC2             mov     eax, edx  

003B1C04    5A               pop     edx  

003B1C05    59               pop     ecx  

003B1C0B    5B               pop     ebx                                          ; `Try.00418000`  

003B1C43    8B9D DCD34100    mov     ebx, dword ptr ss:\[ebp+41D3DC\]                 ; `Try.00400000`  

003B1C76    03C3             add     eax, ebx                                       ; `Try.00400000`  

; now `eax` is `0040E2FD`, the `OEP` :),可以直接 `Dump` 修正...我们走完吧  
```



### 反汇编代码片段

```
003B1C7D    8985 3BD64100   mov     dword ptr ss:[ebp+41D63B], eax             ; Try.0040E2FD  
003B1CD1    80BD 24D64100 0>cmp     byte ptr ss:[ebp+41D624], 1  
003B1CD8    0F85 04010000   jnz     003B1DE2    ; 跳了，不知道跳过什么冬冬  
003B1DF9   /E9 C4000000     jmp     003B1EC2  
003B1EE3    8D85 E45C4000   lea     eax, dword ptr ss:[ebp+405CE4]  
003B1F0B    C785 55384200 0>mov     dword ptr ss:[ebp+423855], 0  
003B1F1A    8BC5            mov     eax, ebp  
003B1F21    5B              pop     ebx  
003B1F22    59              pop     ecx  
003B1F23    5A              pop     edx  
003B1F24    5E              pop     esi  
003B1F25    5F              pop     edi  
003B1F26    5D              pop     ebp  
003B1F3E    9D              popfd  
003B1F6D    FFB0 3BD64100   push    dword ptr ds:[eax+41D63B]  
003B1F78    C780 3BD64100 0>mov     dword ptr ds:[eax+41D63B], 0  
003B1FAF   /E9 CE620000     jmp     003B8282  
003B8287    56              push    esi                             ; ntdll.77F57D70  
003B82B5    51              push    ecx  
003B82BB    BE CD584000     mov     esi, 4058CD  
003B82C5    03F0            add     esi, eax  
003B82CC    B9 82820100     mov     ecx, 18282  
003B8330    C606 00         mov     byte ptr ds:[esi], 0            ; erase the loader  
003B8338    46              inc     esi  
003B833E    49              dec     ecx  
003B8356    83F9 00         cmp     ecx, 0  
003B8359   ^ 75 A8         jnz     short 003B8303  
003B839F    59              pop     ecx                             ; 0012FFB0  
003B83B7    5E              pop     esi  
003B83CF   /E9 81390000     jmp     003BBD55  
003BBD55    60              pushad  
003BBD56    8BF0            mov     esi, eax  
003BBD58    B8 4A344200     mov     eax, 42344A  
003BBD5D    03C6            add     eax, esi  
003BBD5F    BB 59384200     mov     ebx, 423859  
003BBD64    03DE            add     ebx, esi  
003BBD66    803B 00         cmp     byte ptr ds:[ebx], 0  
003BBD69    74 0C          je      short 003BBD77  
003BBD6B    6A 00          push    0  
003BBD6D    50              push    eax  
003BBD6E    53              push    ebx  
003BBD6F    6A 00          push    0  
003BBD71    FF96 55384200   call    dword ptr ds:[esi+423855]  
003BBD77    61              popad  
003BBD78    58              pop     eax  
003BBD79    83F8 FF         cmp     eax, -1  
003BBD7C    75 05          jnz     short 003BBD83  
003BBD7E    33C0            xor     eax, eax  
003BBD80    C2 0C00         retn    0C  
003BBD83    FFE0            jmp     eax     ; 飞向光明之巅！！！  
0040E2FD    55              push    ebp     ; Dump & Fix Dump .优化一下即可  
0040E2FE    8BEC            mov     ebp, esp  
0040E300    6A FF          push    -1  
0040E302    68 F83A4100     push    Try.00413AF8  
0040E307    68 84E44000     push    Try.0040E484                    ; jmp to msvcrt._except_handler3  
0040E30C    64:A1 00000000  mov     eax, dword ptr fs:[0]  
```

### IAT 导入表信息

这是 `IAT`, `ImportRec` 可以全部找到，因为修改了 `Magic Jump`，所以都有效。

`OEP: 0000E2FD`    `IATRVA: 00011000`    `IATSize: 000003B8`

`FThunk: 00011000`    `NbFunc: 00000005`
1    `00011000`    `advapi32.dll`    `01CD`    `RegCreateKeyExA`
1    `00011004`    `advapi32.dll`    `01E2`    `RegOpenKeyExA`
1    `00011008`    `advapi32.dll`    `01D9`    `RegEnumValueA`
1    `0001100C`    `advapi32.dll`    `01F9`    `RegSetValueExA`
1    `00011010`    `advapi32.dll`    `01C9`    `RegCloseKey`

`FThunk: 00011018`    `NbFunc: 00000003`
1    `00011018`    `gdi32.dll`    `0196`    `GetObjectA`
1    `0001101C`    `gdi32.dll`    `002E`    `CreateCompatibleDC`
1    `00011020`    `gdi32.dll`    `0013`    `BitBlt`

`FThunk: 00011028`    `NbFunc: 00000017`
1    `00011028`    `kernel32.dll`    `0385`    `WriteFile`
1    `0001102C`    `kernel32.dll`    `0374`    `WaitForSingleObject`
1    `00011030`    `kernel32.dll`    `0185`    `GetOverlappedResult`
1    `00011034`    `kernel32.dll`    `004A`    `CreateEventA`
1    `00011038`    `kernel32.dll`    `0030`    `CloseHandle`
1    `0001103C`    `kernel32.dll`    `004E`    `CreateFileA`
1    `00011040`    `kernel32.dll`    `0162`    `GetLastError`
1    `00011044`    `kernel32.dll`    `00AF`    `EscapeCommFunction`
1    `00011048`    `kernel32.dll`    `0101`    `GetCommState`
1    `0001104C`    `kernel32.dll`    `0284`    `PurgeComm`
1    `00011050`    `kernel32.dll`    `02CB`    `SetCommMask`
1    `00011054`    `kernel32.dll`    `02CC`    `SetCommState`
1    `00011058`    `kernel32.dll`    `02CD`    `SetCommTimeouts`
1    `0001105C`    `kernel32.dll`    `0334`    `SetupComm`
1    `00011060`    `kernel32.dll`    `0084`    `DeviceIoControl`
1    `00011064`    `kernel32.dll`    `01D5`    `GetVersionExA`
1    `00011068`    `kernel32.dll`    `016D`    `GetModuleFileNameA`
1    `0001106C`    `kernel32.dll`    `016F`    `GetModuleHandleA`
1    `00011070`    `kernel32.dll`    `01D7`    `GetVolumeInformationA`
1    `00011074`    `kernel32.dll`    `01B5`    `GetSystemTime`
1    `00011078`    `kernel32.dll`    `01A6`    `GetStartupInfoA`
1    `0001107C`    `kernel32.dll`    `002E`    `ClearCommError`
1    `00011080`    `kernel32.dll`    `029D`    `ReadFile`

`FThunk: 00011088`    `NbFunc: 0000009D`
1    `00011088`    `mfc42.dll`    `0BA9`    
1    `0001108C`    `mfc42.dll`    `0BA6`    
1    `00011090`    `mfc42.dll`    `13C9`    
1    `00011094`    `mfc42.dll`    `06BF`    
1    `00011098`    `mfc42.dll`    `148D`    
1    `0001109C`    `mfc42.dll`    `098E`    
1    `000110A0`    `mfc42.dll`    `084C`    
1    `000110A4`    `mfc42.dll`    `1479`    
1    `000110A8`    `mfc42.dll`    `0BA6`    
1    `000110AC`    `mfc42.dll`    `0BA6`    
1    `000110B0`    `mfc42.dll`    `0BA6`    
1    `000110B4`    `mfc42.dll`    `06F0`    
1    `000110B8`    `mfc42.dll`    `0C40`    
1    `000110BC`    `mfc42.dll`    `0807`    
1    `000110C0`    `mfc42.dll`    `18E8`    
1    `000110C4`    `mfc42.dll`    `0C09`    
1    `000110C8`    `mfc42.dll`    `0BA0`    
1    `000110CC`    `mfc42.dll`    `0EF6`    
1    `000110D0`    `mfc42.dll`    `0EF1`    
1    `000110D4`    `mfc42.dll`    `0EF1`    
1    `000110D8`    `mfc42.dll`    `0BA6`    
1    `000110DC`    `mfc42.dll`    `0FF0`    
1    `000110E0`    `mfc42.dll`    `1213`    
1    `000110E4`    `mfc42.dll`    `1149`    
1    `000110E8`    `mfc42.dll`    `0E0D`    
1    `000110EC`    `mfc42.dll`    `0144`    
1    `000110F0`    `mfc42.dll`    `0281`    
1    `000110F4`    `mfc42.dll`    `108A`    
1    `000110F8`    `mfc42.dll`    `0CBE`    
1    `000110FC`    `mfc42.dll`    `08F1`    
1    `00011100`    `mfc42.dll`    `1021`    
1    `00011104`    `mfc42.dll`    `03AB`    
1    `00011108`    `mfc42.dll`    `0B02`    
1    `0001110C`    `mfc42.dll`    `0A36`    
1    `00011110`    `mfc42.dll`    `0490`    
1    `00011114`    `mfc42.dll`    `0219`    
1    `00011118`    `mfc42.dll`    `0486`    
1    `0001111C`    `mfc42.dll`    `03AD`    
1    `00011120`    `mfc42.dll`    `1613`    
1    `00011124`    `mfc42.dll`    `0C37`    
1    `00011128`    `mfc42.dll`    `0E20`    
1    `0001112C`    `mfc42.dll`    `0299`    
1    `00011130`    `mfc42.dll`    `07BB`    
1    `00011134`    `mfc42.dll`    `18F1`    
1    `00011138`    `mfc42.dll`    `1442`    
1    `0001113C`    `mfc42.dll`    `015E`    
1    `00011140`    `mfc42.dll`    `0162`    
1    `00011144`    `mfc42.dll`    `04B0`    
1    `00011148`    `mfc42.dll`    `18BE`    
1    `0001114C`    `mfc42.dll`    `16E0`    
1    `00011150`    `mfc42.dll`    `0A52`    
1    `00011154`    `mfc42.dll`    `175D`    
1    `00011158`    `mfc42.dll`    `0C14`    
1    `0001115C`    `mfc42.dll`    `1542`    
1    `00011160`    `mfc42.dll`    `1837`    
1    `00011164`    `mfc42.dll`    `1A7A`    
1    `00011168`    `mfc42.dll`    `188B`    
1    `0001116C`    `mfc42.dll`    `19F8`    
1    `00011170`    `mfc42.dll`    `0942`    
1    `00011174`    `mfc42.dll`    `12E5`    
1    `00011178`    `mfc42.dll`    `10B2`    
1    `0001117C`    `mfc42.dll`    `18E7`    
1    `00011180`    `mfc42.dll`    `1186`    
1    `00011184`    `mfc42.dll`    `1159`    
1    `00011188`    `mfc42.dll`    `0A58`    
1    `0001118C`    `mfc42.dll`    `1663`    



1 `00011190` `mfc42.dll` `0F52`  

1 `00011194` `mfc42.dll` `0441`  

1 `00011198` `mfc42.dll` `144F`  

1 `0001119C` `mfc42.dll` `095C`  

1 `000111A0` `mfc42.dll` `0D12`  

1 `000111A4` `mfc42.dll` `14B4`  

1 `000111A8` `mfc42.dll` `14B6`  

1 `000111AC` `mfc42.dll` `0AA5`  

1 `000111B0` `mfc42.dll` `0FEF`  

1 `000111B4` `mfc42.dll` `125A`  

1 `000111B8` `mfc42.dll` `14BB`  

1 `000111BC` `mfc42.dll` `14A9`  

1 `000111C0` `mfc42.dll` `1652`  

1 `000111C4` `mfc42.dll` `120E`  

1 `000111C8` `mfc42.dll` `1148`  

1 `000111CC` `mfc42.dll` `0E9A`  

1 `000111D0` `mfc42.dll` `0231`  

1 `000111D4` `mfc42.dll` `032F`  

1 `000111D8` `mfc42.dll` `035C`  

1 `000111DC` `mfc42.dll` `0A3D`  

1 `000111E0` `mfc42.dll` `046E`  

1 `000111E4` `mfc42.dll` `096B`  

1 `000111E8` `mfc42.dll` `0BA6`  

1 `000111EC` `mfc42.dll` `06F0`  

1 `000111F0` `mfc42.dll` `112C`  

1 `000111F4` `mfc42.dll` `14AA`  

1 `000111F8` `mfc42.dll` `0D4A`  

1 `000111FC` `mfc42.dll` `0DF6`  

1 `00011200` `mfc42.dll` `047A`  

1 `00011204` `mfc42.dll` `0237`  

1 `00011208` `mfc42.dll` `08FB`  

1 `0001120C` `mfc42.dll` `08FE`  

1 `00011210` `mfc42.dll` `06F7`  

1 `00011214` `mfc42.dll` `1040`  

1 `00011218` `mfc42.dll` `0B2F`  

1 `0001121C` `mfc42.dll` `094B`  

1 `00011220` `mfc42.dll` `0DF3`  

1 `00011224` `mfc42.dll` `0E2A`  

1 `00011228` `mfc42.dll` `096E`  

1 `0001122C` `mfc42.dll` `02F3`  

1 `00011230` `mfc42.dll` `01D6`  

1 `00011234` `mfc42.dll` `0280`  

1 `00011238` `mfc42.dll` `1699`  

1 `0001123C` `mfc42.dll` `0668`  

1 `00011240` `mfc42.dll` `0143`  

1 `00011244` `mfc42.dll` `0669`  

1 `00011248` `mfc42.dll` `0B2B`  

1 `0001124C` `mfc42.dll` `0A5C`  

1 `00011250` `mfc42.dll` `1631`  

1 `00011254` `mfc42.dll` `0685`  

1 `00011258` `mfc42.dll` `0CF6`  

1 `0001125C` `mfc42.dll` `10B5`  

1 `00011260` `mfc42.dll` `168D`  

1 `00011264` `mfc42.dll` `039E`  

1 `00011268` `mfc42.dll` `021C`  

1 `0001126C` `mfc42.dll` `0E4F`  

1 `00011270` `mfc42.dll` `19AB`  

1 `00011274` `mfc42.dll` `074F`  

1 `00011278` `mfc42.dll` `0337`  

1 `0001127C` `mfc42.dll` `0339`  

1 `00011280` `mfc42.dll` `0320`  

1 `00011284` `mfc42.dll` `0ED6`  

1 `00011288` `mfc42.dll` `14A0`  

1 `0001128C` `mfc42.dll` `1101`  

1 `00011290` `mfc42.dll` `18E6`  

1 `00011294` `mfc42.dll` `142B`  

1 `00011298` `mfc42.dll` `0951`  

1 `0001129C` `mfc42.dll` `1479`  

1 `000112A0` `mfc42.dll` `1137`  

1 `000112A4` `mfc42.dll` `06EF`  

1 `000112A8` `mfc42.dll` `0EF1`  

1 `000112AC` `mfc42.dll` `17A4`  

1 `000112B0` `mfc42.dll` `09D2`  

1 `000112B4` `mfc42.dll` `1266`  

1 `000112B8` `mfc42.dll` `0A18`  

1 `000112BC` `mfc42.dll` `12F5`  

1 `000112C0` `mfc42.dll` `1118`  

1 `000112C4` `mfc42.dll` `1479`  

1 `000112C8` `mfc42.dll` `184F`  

1 `000112CC` `mfc42.dll` `10B6`  

1 `000112D0` `mfc42.dll` `164E`  

1 `000112D4` `mfc42.dll` `035A`  

1 `000112D8` `mfc42.dll` `039A`  

1 `000112DC` `mfc42.dll` `0217`  

1 `000112E0` `mfc42.dll` `106C`  

1 `000112E4` `mfc42.dll` `09FA`  

1 `000112E8` `mfc42.dll` `09D0`  

1 `000112EC` `mfc42.dll` `0B67`  

1 `000112F0` `mfc42.dll` `1241`  

1 `000112F4` `mfc42.dll` `0261`  

1 `000112F8` `mfc42.dll` `0628`  

`FThunk:` `00011300` `NbFunc:` `0000001E`

1 `00011300` `msvcrt.dll` `00EE` `\`_except\_handler3\`
1 `00011304` `msvcrt.dll` `009A` `\`__set\_app\_type\`
1 `00011308` `msvcrt.dll` `0087` `\`__p\_fmode\`
1 `0001130C` `msvcrt.dll` `0082` `\`__p\_commode\`
1 `00011310` `msvcrt.dll` `00B7` `\`_adjust\_fdiv\`
1 `00011314` `msvcrt.dll` `009C` `\`__setusermatherr\`
1 `00011318` `msvcrt.dll` `013B` `\`_initterm\`
1 `0001131C` `msvcrt.dll` `006F` `\`__getmainargs\`
1 `00011320` `msvcrt.dll` `00A9` `\`_acmdln\`
1 `00011324` `msvcrt.dll` `0290` `exit`
1 `00011328` `msvcrt.dll` `0050` `\`_XcptFilter\`
1 `0001132C` `msvcrt.dll` `00F7` `\`_exit\`
1 `00011330` `msvcrt.dll` `01B4` `\`_onexit\`
1 `00011334` `msvcrt.dll` `006C` `\`__dllonexit\`
1 `00011338` `msvcrt.dll` `0284` `atoi`
1 `0001133C` `msvcrt.dll` `018C` `\`_mbsicmp\`
1 `00011340` `msvcrt.dll` `02FB` `srand`
1 `00011344` `msvcrt.dll` `02ED` `rand`
1 `00011348` `msvcrt.dll` `02D8` `malloc`
1 `0001134C` `msvcrt.dll` `02A5` `free`
1 `00011350` `msvcrt.dll` `00D7` `\`_controlfp\`
1 `00011354` `msvcrt.dll` `0317` `time`
1 `00011358` `msvcrt.dll` `0307` `strncmp`
1 `0001135C` `msvcrt.dll` `0160` `\`_itoa\`
1 `00011360` `msvcrt.dll` `0054` `\`__CxxFrameHandler\`
1 `00011364` `msvcrt.dll` `0186` `\`_mbscmp\`
1 `00011368` `msvcrt.dll` `02F9` `sprintf`
1 `0001136C` `msvcrt.dll` `030A` `strrchr`
1 `00011370` `msvcrt.dll` `0308` `strncpy`
1 `00011374` `msvcrt.dll` `01DE` `\`_setmbcp\`

`FThunk:` `0001137C` `NbFunc:` `00000001`

1 `0001137C` `netapi32.dll` `0100` `Netbios`

`FThunk:` `00011384` `NbFunc:` `0000000C`

1 `00011384` `user32.dll` `010D` `GetDC`
1 `00011388` `user32.dll` `01B6` `LoadBitmapA`
1 `0001138C` `user32.dll` `01A7` `IsIconic`
1 `00011390` `user32.dll` `0009` `AppendMenuA`
1 `00011394` `user32.dll` `0100` `GetClientRect`
1 `00011398` `user32.dll` `00B7` `DrawIcon`
1 `0001139C` `user32.dll` `015D` `GetSystemMenu`
1 `000113A0` `user32.dll` `015E` `GetSystemMetrics`
1 `000113A4` `user32.dll` `0194` `InvalidateRect`
1 `000113A8` `user32.dll` `01BC` `LoadIconA`
1 `000113AC` `user32.dll` `00C5` `EnableWindow`
1 `000113B0` `user32.dll` `023C` `SendMessageA`

`4dd466d0e1e0cc123afbe504da2462ca`
