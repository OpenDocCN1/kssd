
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
