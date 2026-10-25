
# qq 木马分析(伪造 dll)

> 原文：[`www.pediy.com/kssd/pediy12/134287.html`](https://www.pediy.com/kssd/pediy12/134287.html)

附：老手飘过，新手心情好的可以看看.....

原理：伪造技术，很老的技术，复习一下就当，此木马样本会利用伪造 `msimg32.dll` 技术,对目录下的 `register.db` 获取 qq 聊天记录，和相关的 qq 信息（我没有看到密码...呵呵）

提供者：`x-man`(很不错的家伙，够意思，每次向他要都可以拿到好东西，表扬一下，以后不要和我纠结密码学的东东，我表示我很菜滴)

木马：`web24` `upx` 修改加壳

分析部分：主体分析（紧贴重点）

1. 拿到此马，查壳，`upx` 改造加壳，手脱之 `dumped\_.exe`

********************************************************************************************************************************

2.

用户调用：

```
            loc_402376:                    ; CODE XREF: CODE:00402367j

CODE:00402376 83 F8 01          cmp     eax, 1
CODE:00402379 75 0D             jnz     short loc_402388
CODE:0040237B E8 B0 04 00 00    call    sub_402830            ; 只看此处的用户调用过程
```

3 个关键的 `call` ,主体行为的全部：

```
DE:00402830

CODE:00402830                    sub_402830 proc near          ; CODE XREF: CODE:0040237Bp
CODE:00402830 E8 0B FD FF FF    call    sub_402540            ; 互斥量设置
CODE:00402835 E8 36 FE FF FF    call    sub_402670            ; 遍历进程找 qq
CODE:0040283A E8 71 FD FF FF    call    sub_4025B0
CODE:0040283F A1 48 36 40 00    mov     eax, hWnd
CODE:00402844 50                push    eax                   ; hWnd
CODE:00402845 FF 15 C4 30 41 00 call    ds:CloseWindow
CODE:0040284B E8 90 FE FF FF    call    sub_4026E0            ; 继续启动
CODE:00402850 6A 00             push    0                     ; uExitCode
CODE:00402852 FF 15 10 30 41 00 call    ds:ExitProcess
```

第一个 `call` 就不看了，很简单，易还原

第二个 `call`:`sub_402670`  如果 `qq.exe` 处于运行状态，直接遍历找到后，找第一执行模块相应的执行目录，对目录下 `db`,和 目标 `dll` 进行感染，如果没运行，则找 qq 各种版本的注册表信息 获取路径，执行感染

遍历 qq  `pid`

```
CODE:00401000 81 EC 50 01 00 00 sub     esp, 150h
CODE:00401006 53                push    ebx
CODE:00401007 55                push    ebp
CODE:00401008 56                push    esi
CODE:00401009 57                push    edi
CODE:0040100A 33 C0             xor     eax, eax
CODE:0040100C B9 49 00 00 00    mov     ecx, 49h
CODE:00401011 8D 7C 24 3C       lea     edi, [esp+160h+pe.cntUsage]
CODE:00401015 50                push    eax                   ; th32ProcessID
CODE:00401016 F3 AB             rep stosd
CODE:00401018 6A 02             push    2                     ; dwFlags
CODE:0040101A C7 44 24 40 28 01+mov     [esp+168h+pe.dwSize], 128h
CODE:00401022 E8 61 18 00 00    call    CreateToolhelp32Snapshot
CODE:00401027 8B E8             mov     ebp, eax
CODE:00401029 83 FD FF          cmp     ebp, 0FFFFFFFFh
CODE:0040102C 75 0D             jnz     short loc_40103B
CODE:0040102E 5F                pop     edi
CODE:0040102F 5E                pop     esi
CODE:00401030 5D                pop     ebp
CODE:00401031 33 C0             xor     eax, eax
CODE:00401033 5B                pop     ebx
CODE:00401034 81 C4 50 01 00 00 add     esp, 150h
CODE:0040103A C3                retn
CODE:0040103B                    ; ---------------------------------------------------------------------------

CODE:0040103B

CODE:0040103B                    loc_40103B:                   ; CODE XREF: sub_401000+2Cj
CODE:0040103B 8D 44 24 38       lea     eax, [esp+160h+pe]
CODE:0040103F 50                push    eax                   ; lppe
CODE:00401040 55                push    ebp                   ; hSnapshot
CODE:00401041 E8 3C 18 00 00    call    Process32First
CODE:00401046 85 C0             test    eax, eax
CODE:00401048 75 0B             jnz     short loc_401055
CODE:0040104A 5F                pop     edi
CODE:0040104B 5E                pop     esi
CODE:0040104C 5D                pop     ebp
CODE:0040104D 5B                pop     ebx
CODE:0040104E 81 C4 50 01 00 00 add     esp, 150h
CODE:00401054 C3                retn
```

获取 `pid` 后：找主模块目录

```
       sub     esp, 8                  ; 找主模块目录即 qq.exe 所在目录，利用遍历 qq 模块

CODE:00401203 53                push    ebx
CODE:00401204 55                push    ebp
CODE:00401205 56                push    esi
CODE:00401206 57                push    edi
CODE:00401207 6A 00             push    0                     ; th32ProcessID
CODE:00401209 6A 02             push    2                     ; dwFlags
CODE:0040120B E8 78 16 00 00    call    CreateToolhelp32Snapshot
CODE:00401210 68 28 01 00 00    push    128h                  ; unsigned int
CODE:00401215 89 44 24 14       mov     [esp+1Ch+hObject], eax
CODE:00401219 E8 58 16 00 00    call    ??2@YAPAXI@Z          ; operator new(uint)
CODE:0040121E 8B F0             mov     esi, eax
CODE:00401220 68 24 02 00 00    push    224h                  ; unsigned int
CODE:00401225 89 74 24 1C       mov     [esp+20h+var_4], esi
CODE:00401229 E8 48 16 00 00    call    ??2@YAPAXI@Z          ; operator new(uint)
CODE:0040122E 83 C4 08          add     esp, 8
CODE:00401231 8B D8             mov     ebx, eax
CODE:00401233 8B 44 24 1C       mov     eax, [esp+18h+th32ProcessID]
CODE:00401237 C7 06 28 01 00 00 mov     dword ptr [esi], 128h
CODE:0040123D 50                push    eax                   ; th32ProcessID
CODE:0040123E 6A 08             push    8                     ; dwFlags
CODE:00401240 C7 03 24 02 00 00 mov     dword ptr [ebx], 224h
CODE:00401246 E8 3D 16 00 00    call    CreateToolhelp32Snapshot
CODE:0040124B 53                push    ebx                   ; lpme
CODE:0040124C 50                push    eax                   ; hSnapshot
CODE:0040124D 89 44 24 24       mov     [esp+20h+th32ProcessID], eax
CODE:00401251 E8 38 16 00 00    call    Module32First
CODE:00401256 68 00 01 00 00    push    100h                  ; Size
CODE:0040125B FF 15 A0 30 41 00 call    ds:malloc
CODE:00401261 8B E8             mov     ebp, eax
CODE:00401263 B9 40 00 00 00    mov     ecx, 40h
CODE:00401268 33 C0             xor     eax, eax
CODE:0040126A 8B FD             mov     edi, ebp
CODE:0040126C 8D B3 20 01 00 00 lea     esi, [ebx+120h]
CODE:00401272 6A 5C             push    5Ch                   ; Ch
```

找 qq 各种版本的注册表目录：

```
CODE:004015F0 56                push    esi
CODE:004015F1 8B 74 24 08       mov     esi, [esp+4+arg_0]
CODE:004015F5 56                push    esi                   ; int
CODE:004015F6 68 44 21 40 00    push    offset Str2           ; "QQ"
CODE:004015FB E8 E0 FD FF FF    call    sub_4013E0
CODE:00401600 85 C0             test    eax, eax
CODE:00401602 74 09             jz      short loc_40160D
CODE:00401604 B8 01 00 00 00    mov     eax, 1
CODE:00401609 5E                pop     esi
CODE:0040160A C2 04 00          retn    4
CODE:0040160D                    ; ---------------------------------------------------------------------------

CODE:0040160D

CODE:0040160D                    loc_40160D:                   ; CODE XREF: sub_4015F0+12j
CODE:0040160D 56                push    esi                   ; int
CODE:0040160E 68 3C 21 40 00    push    offset aQq2009        ; "QQ2009"
CODE:00401613 E8 C8 FD FF FF    call    sub_4013E0
CODE:00401618 85 C0             test    eax, eax
CODE:0040161A 74 09             jz      short loc_401625
CODE:0040161C B8 01 00 00 00    mov     eax, 1
CODE:00401621 5E                pop     esi
CODE:00401622 C2 04 00          retn    4
CODE:00401625                    ; ---------------------------------------------------------------------------

CODE:00401625

CODE:00401625                    loc_401625:                   ; CODE XREF: sub_4015F0+2Aj
CODE:00401625 56                push    esi                   ; int
CODE:00401626 68 34 21 40 00    push    offset aQq2010        ; "QQ2010"
CODE:0040162B E8 B0 FD FF FF    call    sub_4013E0
CODE:00401630 85 C0             test    eax, eax
CODE:00401632 74 09             jz      short loc_40163D
CODE:00401634 B8 01 00 00 00    mov     eax, 1
```



### CODE:00401639

```
CODE:00401639 5E                     pop     esi  

CODE:0040163A C2 04 00               retn    4  

CODE:0040163D                         ; ---------------------------------------------------------------------------  

CODE:0040163D  

CODE:0040163D                         loc_40163D:                     ; CODE XREF: sub_4015F0+42j  

CODE:0040163D 56                     push    esi                     ; int  

CODE:0040163E 68 2C 21 40 00       push    offset aQq2011          ; "QQ2011"  

CODE:00401643 E8 98 FD FF FF       call    sub_4013E0  

CODE:00401648 85 C0                  test    eax, eax  

CODE:0040164A 74 09                  jz      short loc_401655  

CODE:0040164C B8 01 00 00 00       mov     eax, 1  

CODE:00401651 5E                     pop     esi  

CODE:00401652 C2 04 00               retn    4  

CODE:00401655                         ; ---------------------------------------------------------------------------  

CODE:00401655  

CODE:00401655                         loc_401655:                     ; CODE XREF: sub_4015F0+5Aj  

CODE:00401655 56                     push    esi                     ; int  

CODE:00401656 68 24 21 40 00       push    offset aTm2009          ; "TM2009"  

CODE:0040165B E8 80 FD FF FF       call    sub_4013E0  

CODE:00401660 F7 D8                  neg     eax  

CODE:00401662 1B C0                  sbb     eax, eax  

CODE:00401664 5E                     pop     esi  

CODE:00401665 F7 D8                  neg     eax  

CODE:00401667 C2 04 00               retn    4  

CODE:00401667                         sub_4015F0 endp  
```

这两种方式找到路径后：判断 `dll` 是否存在

### CODE:004019BE

```
CODE:004019BE 8D 8C 24 0C 01 00+lea     ecx, [esp+20Ch+FileName]  

CODE:004019C5 51                     push    ecx                     ; lpFileName  

CODE:004019C6 E8 D5 F9 FF FF       call    sub_4013A0              ; 打开目录下的 msimg32.dll,达到试探文件是否存在为 0 存在  

CODE:004019CB 85 C0                  test    eax, eax  

CODE:004019CD 74 2E                  jz      short loc_4019FD  

CODE:004019CF 55                     push    ebp  

CODE:004019D0 E8 DB 0B 00 00       call    sub_4025B0  

CODE:004019D5 8B 2D 1C 30 41 00    mov     ebp, ds:DeleteFileA  

CODE:004019DB 8B 3D 20 30 41 00    mov     edi, ds:Sleep  
```

存在就进行感染：

### CODE:00401B15

```
CODE:00401B15 57                     push    edi                     ; nNumberOfBytesToWrite  

CODE:00401B16 8D 94 24 10 01 00+lea     edx, [esp+210h+FileName]  

CODE:00401B1D 53                     push    ebx                     ; lpBuffer  

CODE:00401B1E 52                     push    edx                     ; NumberOfBytesWritten  

CODE:00401B1F E8 6C FD FF FF       call    sub_401890              ; 将加密数据，写入 msimsg.dll 中  

CODE:00401B24 5F                     pop     edi  
```

下面这段代码貌似在主体中作用没啥用，但代码在感染后的 `dll` 中就是通过 `ole32.dll` 函数获取聊天记录

### CODE:00401AA2

```
CODE:00401AA2 50                     push    eax                     ; lpFileName  

CODE:00401AA3 F3 A4                  rep     movsb  

CODE:00401AA5 E8 F6 F8 FF FF       call    sub_4013A0              ; 同样找目录下 register.db 文件判断是否存在  

CODE:00401AAA 85 C0                  test    eax, eax  

CODE:00401AAC 74 0A                  jz      short loc_401AB8  

CODE:00401AAE 8D 4C 24 0C            lea     ecx, [esp+20Ch+MultiByteStr] ; db 路径  

CODE:00401AB2 51                     push    ecx                     ; lpMultiByteStr  

CODE:00401AB3 E8 B8 FB FF FF       call    sub_401670  

ODE:00401670 81 EC 58 03 00 00    sub     esp, 358h  

CODE:00401676 68 58 21 40 00       push    offset LibFileName      ; "OLE32.dll"  

CODE:0040167B C7 44 24 08 00 00+   mov     [esp+35Ch+var_354], 0  

CODE:00401683 FF 15 84 30 41 00    call    ds:LoadLibraryA  

CODE:00401689 85 C0                  test    eax, eax  

CODE:0040168B 0F 84 36 01 00 00    jz      loc_4017C7  

CODE:00401691 68 48 21 40 00       push    offset ProcName        ; "StgOpenStorage"  

CODE:00401696 50                     push    eax                     ; hModule  

CODE:00401697 FF 15 80 30 41 00    call    ds:GetProcAddress  

CODE:0040169D A3 60 36 40 00       mov     StgOpenStorage, eax  
```

感染和路径搜索完成后，这里用了一种很戳的方法加载.

1. 关 `qq` 进程 用户如果重启 `qq`,就加载

2. 遍历模块线程，如果所属的就是 `qq` 进程的线程 发送结束消息，破坏，重启..

### CODE:004025E2

```
04025E2  74 6B                     jz      short loc_40264F  

CODE:004025E4 50                     push    eax                     ; th32ProcessID  

CODE:004025E5 E8 16 EC FF FF       call    sub_401200  

CODE:004025EA A1 58 36 40 00       mov     eax, th32ProcessID  

CODE:004025EF 50                     push    eax                     ; dwProcessId  

CODE:004025F0 E8 8B FF FF FF       call    sub_402580              ; 结束 qq 进程 准备重启，加载 感染后的 dll  

CODE:004025F5 85 C0                  test    eax, eax  

CODE:004025F7 75 49                  jnz     short loc_402642  

CODE:004025F9 8B 0D 58 36 40 00    mov     ecx, th32ProcessID  

CODE:004025FF 51                     push    ecx                     ; th32ProcessID  

CODE:00402600 E8 DB EC FF FF       call    sub_4012E0              ; 找 tid 特定的  

CODE:00402605 8B F8                  mov     edi, eax  

CODE:00402607 85 FF                  test    edi, edi  

CODE:00402609 74 37                  jz      short loc_402642  

CODE:0040260B 6A 00                   push    0                       ; lParam  

CODE:0040260D 6A 00                   push    0                       ; wParam  

CODE:0040260F 6A 12                   push    12h                     ; Msg :quit  

CODE:00402611 57                     push    edi                     ; idThread  

CODE:00402612 FF 15 C8 30 41 00    call    ds:PostThreadMessageA  

CODE:00402618 8B 1D 20 30 41 00    mov     ebx, ds:Sleep  

CODE:0040261E 68 F4 01 00 00       push    1F4h                     ; dwMilliseconds  

CODE:00402623 FF D3                  call    ebx ; Sleep  

CODE:00402625 BE 0A 00 00 00       mov     esi, 0Ah  

 ......  

CODE:0040262A  

CODE:0040262A                         loc_40262A:                     ; CODE XREF: sub_4025B0+90j  

CODE:0040262A 83 FE 19               cmp     esi, 19h  

CODE:0040262D 7D 19                  jge     short loc_402648  

CODE:0040262F 6A 00                   push    0                       ; lParam  

CODE:00402631 6A 00                   push    0                       ; wParam  

CODE:00402633 56                     push    esi                     ; Msg:对特定的 qq 线程一直发送 quit ,,消息一直 由 ah 到 18h  

CODE:00402633                                 ; 加 sleep 获取时间长度...目的让想要的线程重启击中我们模块的.  

CODE:00402634 57                     push    edi                     ; idThread  

CODE:00402635 FF 15 C8 30 41 00    call    ds:PostThreadMessageA  

CODE:0040263B 6A 0A                   push    0Ah                     ; dwMilliseconds  

CODE:0040263D FF D3                  call    ebx ; Sleep  

CODE:0040263F 46                       inc     esi  

CODE:00402640 EB E8                  jmp     short loc_40262A  

CODE:00402642                         ; -------------------------------  
```

`call sub_4026E0` :继续启动本程序 手法：创建进程

### CODE:004027DD

```
CODE:004027DD 56                     push    esi                     ; lpThreadAttributes  

CODE:004027DE 56                     push    esi                     ; lpProcessAttributes  

CODE:004027DF 50                     push    eax                     ; lpCommandLine  

CODE:004027E0 56                     push    esi                     ; lpApplicationName  

CODE:004027E1 FF 15 44 30 41 00    call    ds:CreateProcessA  

CODE:004027E7 85 C0                  test    eax, eax  

CODE:004027E9 74 32                  jz      short loc_40281D  

CODE:004027EB 8D 4C 24 60            lea     ecx, [esp+360h+szLongPath]  

CODE:004027EF 68 80 00 00 00       push    80h                       ; dwFileAttributes  

CODE:004027F4 51                     push    ecx                     ; lpFileName  

CODE:004027F5 FF 15 40 30 41 00    call    ds:SetFileAttributesA  

CODE:004027FB 8B 35 3C 30 41 00    mov     esi, ds:SetPriorityClass  

CODE:00402801 6A 40                   push    40h                     ; dwPriorityClass  

CODE:00402803 8B 54 24 10            mov     edx, [esp+364h+ProcessInformation.hProcess]  

CODE:00402807 52                     push    edx                     ; hProcess  

CODE:00402808 FF D6                  call    esi ; SetPriorityClass  

CODE:0040280A 68 80 00 00 00       push    80h                       ; dwPriorityClass  

CODE:0040280F 53                     push    ebx                     ; hProcess  

CODE:00402810 FF D6                  call    esi ; SetPriorityClass  
```



# 技术分析

```
CODE:00402812 8B 44 24 10       mov     eax, [esp+360h+ProcessInformation.hThread]  
CODE:00402816 50                push    eax                ; hThread  
```

感染后的 `dll`，基本就是先加载伪造 `dll`，然后在伪造 `dll` 中，加载系统目录下 未感染的 `msimg32.dll` 达到不影响 `qq` 运行...这技术 `海风大大` 有专门讲过，大家可以看看：

## `dll` 中

```
5C    mov     [esp+124h+var_110], 5Ch  
```

```
.text:730014E4 88 4C 24 15       mov     [esp+124h+var_10F], cl  
```

```
.text:730014E8 C6 44 24 19 67    mov     [esp+124h+var_10B], 67h  
```

```
.text:730014ED C6 44 24 1A 33    mov     [esp+124h+var_10A], 33h  
```

```
.text:730014F2 C6 44 24 1B 32    mov     [esp+124h+var_109], 32h  
```

```
.text:730014F7 C6 44 24 1C 2E    mov     [esp+124h+var_108], 2Eh  
```

```
.text:730014FC C6 44 24 1D 64    mov     [esp+124h+var_107], 64h  
```

```
.text:73001501 C6 44 24 16 73    mov     [esp+124h+var_10E], 73h  
```

```
.text:73001506 C6 44 24 17 69    mov     [esp+124h+var_10D], 69h  
```

```
.text:7300150B 88 4C 24 18       mov     [esp+124h+var_10C], cl  
```

```
.text:7300150F C6 44 24 20 00    mov     [esp+124h+var_104], 0  
```

```
.text:73001514 FF 15 08 10 00 73 call    ds:GetSystemDirectoryA  
```

```
.text:7300151A 8D 7C 24 0C       lea     edi, [esp+11Ch+var_110]  
```

```
.text:7300151E 83 C9 FF          or      ecx, 0FFFFFFFFh  
```

```
.text:73001521 33 C0             xor     eax, eax  
```

```
.text:73001523 8D 54 24 1C       lea     edx, [esp+11Ch+LibFileName]  
```

```
.text:73001527 F2 AE             repne scasb  
```

```
.text:73001529 F7 D1             not     ecx  
```

```
.text:7300152B 2B F9             sub     edi, ecx  
```

```
.text:7300152D 8B F7             mov     esi, edi  
```

```
.text:7300152F 8B D9             mov     ebx, ecx  
```

```
.text:73001531 8B FA             mov     edi, edx  
```

```
.text:73001533 83 C9 FF          or      ecx, 0FFFFFFFFh  
```

```
.text:73001536 F2 AE             repne scasb  
```

```
.text:73001538 8B CB             mov     ecx, ebx  
```

```
.text:7300153A 4F                dec     edi  
```

```
.text:7300153B C1 E9 02          shr     ecx, 2  
```

```
.text:7300153E F3 A5             rep movsd  
```

```
.text:73001540 8B CB             mov     ecx, ebx  
```

```
.text:73001542 8D 44 24 1C       lea     eax, [esp+11Ch+LibFileName]  
```

```
.text:73001546 83 E1 03          and     ecx, 3  
```

```
.text:73001549 50                push    eax                ; lpLibFileName  "system32\\msimg32.dll"  
```

```
.text:7300154A F3 A4             rep movsb  
```

```
.text:7300154C FF 15 04 10 00 73 call    ds:LoadLibraryA      ; 继续加载此 msgig.dll  
```

```
.text:73001552 8B 3D 00 10 00 73 mov     edi, ds:GetProcAddress  
```

```
.text:73001558 8B F0             mov     esi, eax  
```

## 清除木马

### 法一

1. 清除 `qq` 目录下的 `msimg32.dll` 将 系统目录下的 拷贝进来，也可以根据 加载优先的关系 直接 `delete` 掉感染的 `dll`（貌似没有必要去还原感染 `dll`，因为已经面目全非了）
2. 杀掉 `web24.exe` 进程 ，并清除掉

### 法二

你懂的.....

此马到此结束 ，适合新手观看...
