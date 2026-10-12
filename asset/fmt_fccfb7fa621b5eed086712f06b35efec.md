
```
0040C36B   .  83C4 08       add     esp, 8  
0040C36E   .  83FB 03       cmp     ebx, 3  
0040C371   .  7D 51        jge     short 0040C3C4  

跟踪 call    0040C090  

0040C090  /$  83EC 08       sub     esp, 8  
0040C093  |.  53           push    ebx  
0040C094  |.  55           push    ebp  
0040C095  |.  8B6C24 14    mov     ebp, dword ptr [esp+14]  
0040C099  |.  57           push    edi  
0040C09A  |.  6A 00        push    0                      ; /pFileSizeHigh = NULL  
0040C09C  |.  56           push    esi                    ; |hFile  
0040C09D  |.  FF15 08014300 call    dword ptr [<&KERNEL32.GetFileSiz>; \GetFileSize  
0040C0A3  |.  8BF8         mov     edi, eax  
0040C0A5  |.  8D5F FC      lea     ebx, dword ptr [edi-4]  
0040C0A8  |.  53           push    ebx  
0040C0A9  |.  C74424 1C 000>mov     dword ptr [esp+1C], 0  
0040C0B1  |.  E8 B75D0100  call    00421E6D               ;   new  
0040C0B6  |.  83C4 04      add     esp, 4  
0040C0B9  |.  6A 00        push    0                      ; /pOverlapped = NULL  
0040C0BB  |.  8D4C24 10    lea     ecx, dword ptr [esp+10]          ; |  
0040C0BF  |.  51           push    ecx                    ; |pBytesRead  
0040C0C0  |.  53           push    ebx                    ; |BytesToRead  
0040C0C1  |.  8B1D 10014300 mov     ebx, dword ptr [<&KERNEL32.ReadF>; |kernel32.ReadFile  
0040C0C7  |.  50           push    eax                    ; |Buffer 文件内容  
0040C0C8  |.  56           push    esi                    ; |hFile  
0040C0C9  |.  8945 00      mov     dword ptr [ebp], eax     ; |  
0040C0CC  |.  FFD3         call    ebx                    ; \ReadFile  
0040C0CE  |.  6A 00        push    0                      ; /pOverlapped = NULL  
0040C0D0  |.  8D5424 10    lea     edx, dword ptr [esp+10]          ; |  
0040C0D4  |.  52           push    edx                    ; |pBytesRead  
0040C0D5  |.  6A 04        push    4                      ; |BytesToRead = 4  
0040C0D7  |.  8D4424 1C    lea     eax, dword ptr [esp+1C]          ; |  
0040C0DB  |.  50           push    eax                    ; |BUFFER 解密号  
0040C0DC  |.  56           push    esi                    ; |hFile  
0040C0DD  |.  FFD3         call    ebx                    ; \ReadFile  
0040C0DF  |.  8B4D 00      mov     ecx, dword ptr [ebp]     ;  new address  
0040C0E2  |.  51           push    ecx  
0040C0E3  |.  8D57 FC      lea     edx, dword ptr [edi-4]  
0040C0E6  |.  8D4C24 1C    lea     ecx, dword ptr [esp+1C]  
0040C0EA  |.  E8 21FFFFFF  call    0040C010               ;  通过文件内容算出加密号  
0040C0EF  |.  8B4424 14    mov     eax, dword ptr [esp+14]  
0040C0F3  |.  8B4C24 1C    mov     ecx, dword ptr [esp+1C]  
0040C0F7  |.  83C4 04      add     esp, 4  
0040C0FA  |.  3BC1         cmp     eax, ecx               ;  比较两个加密号  
0040C0FC  |.  5F           pop     edi  
0040C0FD  |.  5D           pop     ebp  
0040C0FE  |.  5B           pop     ebx  
0040C0FF  |.  74 07        je      short 0040C108          
0040C101  |.  83C8 FF      or      eax, FFFFFFFFF           ；出错  
0040C104  |.  83C4 08      add     esp, 8  
0040C107  |.  C3           retn  
0040C108  |>  8BC2         mov     eax, edx               ;加密号位置  
0040C10A  |.  83C4 08      add     esp, 8  
0040C10D  \\.  C3           retn  
```

分析后得出：

取出文件内容直到倒数第 4 个字节。最后 4 个字节再独立取出来。

我们认为这 4 个字节用于文件校验。称为加密号

接下来应该对 `NME` 文件进行处理，我们猜想可能是查 `mlog.bmp` 对应的数据

```
0040C3F0   >  8B53 F8      mov     edx, dword ptr [ebx-8]  
0040C3F3   . |035424 14    add     edx, dword ptr [esp+14]          ;  取下一个 OBJ  
0040C3F7   . |68 A8CE4300  push    0043CEA8               ;  ASCII "mlogo.bmp"  
0040C3FC   . |52           push    edx                    ;  NME 中的文件名  
0040C3FD   . |E8 EF5F0100  call    004223F1               ;  Find Obj  
0040C402   . |83C4 08      add     esp, 8  
0040C405   . |85C0         test    eax, eax               ;  0:found  
0040C407   . |0F85 B9000000 jnz     0040C4C6  
0040C40D   . |68 B8D04300  push    0043D0B8               ;  ASCII "parts"  
0040C412   . |68 3C984300  push    0043983C               ;  ASCII "%s.det"  
0040C417   . |68 B0CF4300  push    0043CFB0               ;  ASCII "parts.nme"  
0040C41C   . |FFD7         call    edi                    ； wspintf  
0040C41E   . |83C4 0C      add     esp, 0C  
0040C421   > |6A 00        push    0  
0040C423   . |68 80000000  push    80  
0040C428   . |6A 03        push    3  
0040C42A   . |6A 00        push    0  
0040C42C   . |6A 00        push    0  
0040C42E   . |68 00000080  push    80000000  
0040C433   . |68 B0CF4300  push    0043CFB0               ;  ASCII "parts.nme"  
0040C438   . |FFD5         call    ebp                    ; open this file  
0040C43A   . |6A 00        push    0                      ; /Origin = FILE\_BEGIN  
0040C43C   . |8BF8         mov     edi, eax               ; |  
0040C43E   . |8B43 FC      mov     eax, dword ptr [ebx-4]     ; |  
0040C441   . |6A 00        push    0                      ; |pOffsetHi = NULL  
0040C443   . |50           push    eax                    ; |OffsetLo  
0040C444   . |57           push    edi                    ; |hFile  
0040C445   . |FF15 0C014300 call    dword ptr [<&KERNEL32.SetFilePoi>; \SetFilePointer  
0040C44B   . |8B33         mov     esi, dword ptr [ebx]  
0040C44D   . |56           push    esi  
0040C44E   . |E8 1A5A0100  call    00421E6D               ; new  
0040C453   . |83C4 04      add     esp, 4  
0040C456   . |6A 00        push    0                      ; /pOverlapped = NULL  
0040C458   . |8D4C24 2C    lea     ecx, dword ptr [esp+2C]          ; |  
0040C45C   . |51           push    ecx                    ; |pBytesRead  
0040C45D   . |56           push    esi                    ; |BytesToRead  
0040C45E   . |8BE8         mov     ebp, eax               ; |  
0040C460   . |55           push    ebp                    ; |Buffer  
0040C461   . |57           push    edi                    ; |hFile  
0040C462   . |FF15 10014300 call    dword ptr [<&KERNEL32.ReadFile>] ; \ReadFile  
0040C468   . |57           push    edi                    ; /hObject  
0040C469   . |FF15 D0004300 call    dword ptr [<&KERNEL32.CloseHandl>; \CloseHandle  
```

查到 `mlog.bmp` 时，除了返回 `TRUE`，还需要一些相关信息。这些信息可能记录在内存中。或一个对象中

要么是 `OBJ` 对象。要么是全局变量。所以需要跟踪 `call    004223F1`

发现内部复杂，找不到有用的东西。暂时不管它。

我们又发现在不断的查找过程当中有个值在累加。是 `EBX`

分析后面的代码发现：

```
0040C4C6   > \8B4424 18    mov     eax, dword ptr [esp+18]  
0040C4CA   .  8B4C24 20    mov     ecx, dword ptr [esp+20]  
0040C4CE   .  40           inc     eax  
0040C4CF   .  83C3 14      add     ebx, 14  
0040C4D2   .  3BC1         cmp     eax, ecx  
0040C4D4   .  894424 18    mov     dword ptr [esp+18], eax  
0040C4D8   .^ 0F8C 12FFFFFF jl      0040C3F0  
```

这说明信息并不是通过 `call 4223f1` 来得到的，而只是简单的一一对应方式。

`mblog.bmp` 是 `parts.nme` 中的第 6 个，按 0 算，就是 `5*14h=64h`; `14h` 一定是 `BMP` 的信息结构。

`64h+8h` 是在 `parts.atm` 中的位置。也就是说，`ATM` 文件中存放的是 `DEC` 文件中资源的定位信息。

`parts.atm` 中头 8 个字节(值为 0)可能有另用。

我们猜想接下来就是查看 `parts.det`，并得到 `mlog.bmp` 对应的资源数据。

```
0040C40D   .  68 B8D04300   push    0043D0B8               ;  ASCII "parts"  
0040C412   .  68 3C984300   push    0043983C               ;  ASCII "%s.det"  
0040C417   .  68 B0CF4300   push    0043CFB0               ;  ASCII "parts.det"  
0040C41C   .  FFD7          call    edi  
0040C41E   .  83C4 0C       add     esp, 0C  
```
