
### 0042ECDA: 调用 `_Readmemory`

```
0042ECDA  |.  83E9 02    sub     ecx, 2               ; |  
0042ECDD  |.  51         push    ecx                  ; |Arg2  
0042ECDE  |.  8D45 BB    lea     eax, dword ptr [ebp-45]    ; |  
0042ECE1  |.  50         push    eax                  ; |Arg1  
0042ECE2  |.  E8 2526030>call    _Readmemory          ; \\_Readmemory  
0042ECE7  |.  83C4 10    add     esp, 10  
0042ECEA  |.  83F8 01    cmp     eax, 1  
0042ECED  |.  75 0D      jnz     short 0042ECFC  
0042ECEF  |.  33D2       xor     edx, edx  
0042ECF1  |.  8A55 BB    mov     dl, byte ptr [ebp-45]  
0042ECF4  |.  81FA CD000>cmp     edx, 0CD  
0042ECFA  |.  74 07      je      short 0042ED03  
0042ECFC  |>  33C9       xor     ecx, ecx  
0042ECFE  |.  894D DC    mov     dword ptr [ebp-24], ecx  
0042ED01  |.  EB 07      jmp     short 0042ED0A  
0042ED03  |>  C745 DC 02>mov     dword ptr [ebp-24], 2  
```

### 检查参数中的值是 eip 还是 exceptionaddr

若是 `eip` 则跳过对几个全局变量的赋值：

```
0042ED0A  |>  8B45 DC    mov     eax, dword ptr [ebp-24]  
0042ED0D  |.  2945 D8    sub     dword ptr [ebp-28], eax  
0042ED10  |.  8B15 08574>mov     edx, dword ptr [4D5708]    ; edx = reg_eip  
0042ED16  |.  3B55 D8    cmp     edx, dword ptr [ebp-28]  
0042ED19  |.  75 08      jnz     short 0042ED23      ; 参数为 exceptionaddr 时跳转  
0042ED1B  |.  3B1D 0C574>cmp     ebx, dword ptr [4D570C]  ; 检查 ecx = reg_ecx?  
0042ED21  |.  74 16      je      short 0042ED39    ; 参数为 eip 时跳转  
```

### 对全局变量赋值

```
0042ED23  |>  33C9       xor     ecx, ecx  
0042ED25  |.  8B45 D8    mov     eax, dword ptr [ebp-28]  
0042ED28  |.  890D 10574>mov     dword ptr [4D5710], ecx  
0042ED2E  |.  A3 08574D0>mov     dword ptr [4D5708], eax  ; reg_eip  
0042ED33  |.  891D 0C574>mov     dword ptr [4D570C], ebx  ; reg_ecx  
```

### 检查异常类型并跳转

检查到底是什么异常，并做相应的跳转。我们主要处理的是内存断点，其他略过，即是这里的访问异常 `EXCEPTION_ACCESS_VIOLATION`：

```
0042ED39  |>  8B55 A4    mov     edx, dword ptr [ebp-5C]  
0042ED3C  |.  8B0A       mov     ecx, dword ptr [edx]  
0042ED3E  |.  81F9 8F000>cmp     ecx, C000008F           ; EXCEPTION_FLT_INEXACT_RESULT  
0042ED44  |.  7F 69      jg      short 0042EDAF  
0042ED46  |.  0F84 3F1A0>je      0043078B  
0042ED4C  |.  81F9 1D000>cmp     ecx, C000001D    ; EXCEPTION_ILLEGAL_INSTRUCTION  
0042ED52  |.  7F 33      jg      short 0042ED87  
0042ED54  |.  0F84 D01D0>je      00430B2A  
0042ED5A  |.  81E9 01000>sub     ecx, 80000001  ; EXCEPTION_GUARD_PAGE  
0042ED60  |.  0F84 1A180>je      00430580  
0042ED66  |.  83E9 02    sub     ecx, 2      ; EXCEPTION_BREAKPOINT  
0042ED69  |.  0F84 95000>je      0042EE04  
0042ED6F  |.  49         dec     ecx         ; EXCEPTION_SINGLE_STEP  
0042ED70  |.  0F84 F70E0>je      0042FC6D  
0042ED76  |.  81E9 01000>sub     ecx, 40000001    ; EXCEPTION_ACCESS_VIOLATION  
0042ED7C  |.  0F84 12120>je      0042FF94  
0042ED82  |.  E9 CD1E000>jmp     00430C54  
```

下面有对上面提到的几个异常的处理，这里先略过。。。：

……

……

### 查找异常所在的模块，并得到其模块信息

```
0042FF94  |>  8B45 D8    mov     eax, dword ptr [ebp-28]  
0042FF97  |.  50         push    eax                  ; /Arg1  
0042FF98  |.  E8 7BDE020>call    _Findmodule          ; \\_Findmodule  
0042FF9D  |.  59         pop     ecx  
0042FF9E  |.  8945 AC    mov     dword ptr [ebp-54], eax     ; 模块首地址赋值给一个局部变量  
```

### 通过比较模块中的各种信息判断内存断点是否正常

若不正常跳转到错误处理处：

```
0042FFA1  |.  8B45 A4    mov     eax, dword ptr [ebp-5C]  
0042FFA4  |.  8B55 A4    mov     edx, dword ptr [ebp-5C]  
0042FFA7  |.  8378 10 02  cmp     dword ptr [eax+10], 2    ; ExceptionFlags > 2 ?  
0042FFAB  |.  8B7A 18    mov     edi, dword ptr [edx+18]  
0042FFAE  |.  0F82 65040>jb      00430419  
0042FFB4  |.  833D 40814>cmp   dword ptr [4D8140], 0  ; 内存断点的长度是否为 0  
0042FFBB  |.  0F84 58040>je      00430419  
0042FFC1  |.  833D 00574>cmp     dword ptr [4D5700], 0  
0042FFC8  |.  0F84 4B040>je      00430419  
0042FFCE  |.  3B3D 44814>cmp     edi, dword ptr [4D8144]  ;异常地址 < 断点内存页首地址?  
0042FFD4  |.  0F82 3F040>jb      00430419  
0042FFDA  |.  3B3D 48814>cmp     edi, dword ptr [4D8148]  ;异常地址 > 断点内存页尾地址?  
0042FFE0  |.  0F83 33040>jnb     00430419  
```

### 读取异常部分的机器码 10h 字节

由于 `OpCode` 不会超过 16 个字节，所以一定够用了：

```
0042FFE6  |.  830D 74574>or      dword ptr [4D5774], 20  
0042FFED  |.  8D95 98FDF>lea     edx, dword ptr [ebp-268]  
0042FFF3  |.  52         push    edx                  ; /Arg2  
0042FFF4  |.  830D 10574>or      dword ptr [4D5710], 2         ; |  
0042FFFB  |.  8B4D D8    mov     ecx, dword ptr [ebp-28]       ; |  
0042FFFE  |.  51         push    ecx                  ; |Arg1  
0042FFFF  |.  E8 8016030>call    _Readcommand         ; \\_Readcommand  
00430004  |.  83C4 08    add     esp, 8  
```

### 检查读取的机器码长度

若为小于等于 0 则跳转到错误处理：

```
00430007  |.  8945 C8    mov     dword ptr [ebp-38], eax  
0043000A  |.  837D C8 00  cmp     dword ptr [ebp-38], 0  
0043000E  |.  76 28      jbe     short 00430038  
```

### 将内存异常的机器码反汇编

并将汇编代码的机器码长度给一个局部变量：

```
00430010  |.  56         push    esi                  ; /Arg7 threadid  
00430011  |.  6A 05      push    5                    ; |Arg6 = 00000005 DISASM_ALL  
00430013  |.  8D85 80E5F>lea     eax, dword ptr [ebp-1A80]      ; |  
00430019  |.  50         push    eax                  ; |Arg5 pt_Disasm 结构体  
0043001A  |.  6A 00      push    0                    ; |Arg4 = 00000000  
0043001C  |.  8B55 D8    mov     edx, dword ptr [ebp-28]       ; |  
0043001F  |.  52         push    edx                  ; |Arg3 addr of command  
00430020  |.  8B4D C8    mov     ecx, dword ptr [ebp-38]       ; |  
00430023  |.  51         push    ecx                  ; |Arg2  
00430024  |.  8D85 98FDF>lea     eax, dword ptr [ebp-268]       ; |  
0043002A  |.  50         push    eax                  ; |Arg1  
0043002B  |.  E8 185FFEF>call    _Disasm              ; \\_Disasm  
00430030  |.  83C4 1C    add     esp, 1C  
00430033  |.  8945 F4    mov     dword ptr [ebp-C], eax  ; 将长度给局部变量  
00430036  |.  EB 05      jmp     short 0043003D  ;跳过错误处理  
```

### 读取机器码失败的错误处理

设置读取长度为 0：

```
00430038  |>  33D2       xor     edx, edx  
0043003A  |.  8955 F4    mov     dword ptr [ebp-C], edx  
```

### 检查读取长度

若小于等于 0,则跳转到错误处理：

```
0043003D  |>  837D F4 00  cmp     dword ptr [ebp-C], 0  
00430041  |.  0F8E 9A010>jle     004301E1  
```

### 比较地址边界

将异常地址加上其汇编代码长度，比较其地址是否低于等于异常地址，若是则跳到异常处理：

```
00430047  |.  8B4D D8    mov     ecx, dword ptr [ebp-28]  
0043004A  |.  034D F4    add     ecx, dword ptr [ebp-C]  
0043004D  |.  3B0D 3C814>cmp     ecx, dword ptr [4D813C]    ;  Ollydbg.0040100C  
00430053  |.  0F86 88010>jbe     004301E1  
```

将内存断点地址加断点范围，比较其地址是否小于等于异常地址，若是则跳到异常处理：

```
00430059  |.  A1 3C814D0>mov     eax, dword ptr [4D813C]  
0043005E  |.  0305 40814>add     eax, dword ptr [4D8140]  
00430064  |.  3B45 D8    cmp     eax, dword ptr [ebp-28]  
00430067  |.  0F86 74010>jbe     004301E1  
```

### 继续检查标志并跳转

并跳转到相应的处理函数处：

```
0043006D  |.  833D 38814>cmp     dword ptr [4D8138], 0  
00430074  |.  0F84 43010>je      004301BD  
0043007A  |.  833D 688D4>cmp     dword ptr [4D8D68], 0  
00430081  |.  74 78      je      short 004300FB  
```

### 检查模块信息

检查模块信息是否存在，并检查模块信息中的值是否正确，若入口在代码段的外面则要求：

```
00430083  |.  837D AC 00  cmp     dword ptr [ebp-54], 0    ; 检查模块信息是否存在  
00430087  |.  74 72      je      short 004300FB  
00430089  |.  8B55 AC    mov     edx, dword ptr [ebp-54]  ; [ebp-54]为模块信息结构体首地址  
0043008C  |.  8B45 AC    mov     eax, dword ptr [ebp-54]  
0043008F  |.  8B8A E1030>mov     ecx, dword ptr [edx+3E1]  
00430095  |.  3B48 0C    cmp     ecx, dword ptr [eax+C]  ; 入口点是否在代码段前面  
00430098  |.  72 61      jb      short 004300FB  
0043009A  |.  8B55 AC    mov     edx, dword ptr [ebp-54]  
0043009D  |.  8B45 AC    mov     eax, dword ptr [ebp-54]  
004300A0  |.  8B4A 0C    mov     ecx, dword ptr [edx+C]  
004300A3  |.  8B55 AC    mov     edx, dword ptr [ebp-54]  
004300A6  |.  0388 E9030>add     ecx, dword ptr [eax+3E9]  
```
