
# OllyDBG 分析报告系列(2)---内存断点

> 原文：[`www.pediy.com/kssd/pediy10/65221.html`](https://www.pediy.com/kssd/pediy10/65221.html)

这是分析报告的第二部分，请多多指教。`![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")`

`Ollydbg`（以下均简称为 `OD`）中的内存断点的主要功能是：修改要下内存断点的内存页属性，如访问断点就设置禁止访问、执行断点就设置禁止执行。这样当程序要用到这个内存页时就会报异常，由 `OD` 捕获，`OD` 比较一下是否是下了断点的内存区域，如果不是就继续执行，否则就断下等待调试人员操作。

内存断点的添加、设置以及判断部分是由不同的函数处理的，先说一下这三个函数的关系，然后写出其流程，最后分析代码：

内存断点添加函数一般只是设置内存断点信息表，并不修改内存访问属性，若是在运行状态时设置内存断点，则会修改内存页属性；内存断点设置函数是在按下 `F9` 时才会被调用的，并且每走一步调用一次，其主要作用就是根据内存断点信息表设置内存页属性；内存断点处理函数则是在一个大的异常处理函数中的一部分，`OD` 通过异常事件结构体处理对应的各种异常，下面只提取有关的部分进行分析。

`OD` 中用一个结构体来存储内存断点的信息，且只能下一个内存断点，下第二个断点会将前面的记录覆盖：

```text
第一个 4 字节 00000000  未知
第二个 4 字节 00400654  标识内存断点首地址
第三个 4 字节 00000004  内存断点字节的长度
第四个 4 字节 00400000  内存断点首地址所在内存页的地址
第五个 4 字节 00401000  内存断点尾地址所在内存页的地址
第六个 4 字节 00000001  01－标识内存访问断点 20－标识内存写入断点
第七个 4 字节 00000000  未知

004D8138  00 00 00 00 33 05 40 00 02 00 00 00 00 00 40 00  ....3 @. .....@.
004D8148  00 10 40 00 01 00 00 00 00 00 00 00              . @. .......
```

此外还有一个存放内存信息的结构体：

```c
typedef struct t_memory {      // Memory block descriptor
  ulong      base;             // Base address of memory block
  ulong      size;             // Size of block
  ulong      type;             // Service information, TY_xxx
  ulong      owner;            // Address of owner of the memory
  ulong      initaccess;       // Initial read/write access
  ulong      access;           // Actual status and read/write access
  ulong      threadid;         // Block belongs to this thread or 0
  char       sect[SHORTLEN];   // Name of module section
  char       *copy;            // Copy used in CPU window or NULL
  ulong      reserved[8];      // Reserved for plugin compatibility
} t_memory;
```

首先是添加部分的函数：

1. 内存断点的添加是通过消息来处理的，而且反汇编窗口和数据窗口的消息还不一样，这样就有了两组对应的消息，数据窗口中：访问断点－`7Eh`、写入断点－`7Fh`、清除断点－`80h`；反汇编窗口中：访问断点－`23h`、写入断点－`24h`、清除断点－`25h`；
2. 将 `Setmembreakpoint` 函数参数传入，让函数做相应的处理。该函数有三个参数，分别为标识断点属性、首地址以及长度，这里先概括的描述一下这个函数的流程：
    - 将参数赋值给寄存器，并得到内存断点所在的内存页；
    - 检查是否在系统内存处、资源内存处、堆栈内存处下断点，若是则弹出警告，并返回 `-1` 退出函数；
    - 将内存断点的信息写入一个结构体中，便于查询；
    - 写完结构体后检查是否运行状态，如果是则调用函数修改内存页属性，值得一提的这里修改内存页属性的 API 并不是直接调用，而是用一个指针指向这两个 API：`VirtualQuery`、`VirtualProtectEx`，在使用前还会检查两个 API 是否存在；
    - 最后返回 `0` 并退出

这部分为数据窗口中设置内存断点部分的代码：

判断是否为访问断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
00451C29   > \83FF 7>cmp     edi, 7E
00451C2C   .  75 2E  jnz     short 00451C5C
00451C2E   .  8B45 A>mov     eax, dword ptr [ebp-54]   ;  Case 7E of switch 0045115A
00451C31   .  2B45 B>sub     eax, dword ptr [ebp-50]
00451C34   .  50     push    eax                   ; /Arg3
00451C35   .  8B55 B>mov     edx, dword ptr [ebp-50]     ; |
00451C38   .  52     push    edx                   ; |Arg2
00451C39   .  6A 03  push    3                     ; |Arg1 = 00000003
00451C3B   .  E8 987>call    _Setmembreakpoint     ; \_Setmembreakpoint
00451C40   .  83C4 0>add     esp, 0C
00451C43   .  F683 5>test    byte ptr [ebx+25D], 10
00451C4A   .  0F84 2>je      00452C7C
00451C50   .  53     push    ebx                   ; /Arg1
00451C51   .  E8 F6C>call    0044E54C              ; \Ollydbg.0044E54C
00451C56   .  59     pop     ecx
00451C57   .  E9 201>jmp     00452C7C
```

判断是否为写入断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
00451C5C   >  83FF 7>cmp     edi, 7F
00451C5F   .  75 2E  jnz     short 00451C8F
00451C61   .  8B4D A>mov     ecx, dword ptr [ebp-54]   ;  Case 7F of switch 0045115A
00451C64   .  2B4D B>sub     ecx, dword ptr [ebp-50]
00451C67   .  51     push    ecx                   ; /Arg3
00451C68   .  8B45 B>mov     eax, dword ptr [ebp-50]   ; |
00451C6B   .  50     push    eax                   ; |Arg2
00451C6C   .  6A 02  push    2                     ; |Arg1 = 00000002
00451C6E   .  E8 657>call    _Setmembreakpoint     ; \_Setmembreakpoint
00451C73   .  83C4 0>add     esp, 0C
00451C76   .  F683 5>test    byte ptr [ebx+25D], 10
00451C7D   .  0F84 F>je      00452C7C
00451C83   .  53     push    ebx                   ; /Arg1
00451C84   .  E8 C3C>call    0044E54C              ; \Ollydbg.0044E54C
00451C89   .  59     pop     ecx
00451C8A   .  E9 ED0>jmp     00452C7C
```

判断是否为清除断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
00451C8F   >  81FF 8>cmp     edi, 80
00451C95   .  75 13  jnz     short 00451CAA
00451C97   .  6A 00  push    0                     ; /Arg3 = 00000000;
00451C99   .  6A 00  push    0                     ; |Arg2 = 00000000
00451C9B   .  6A 00  push    0                     ; |Arg1 = 00000000
00451C9D   .  E8 367>call    _Setmembreakpoint     ; \_Setmembreakpoint
00451CA2   .  83C4 0>add     esp, 0C
00451CA5   .  E9 D20>jmp     00452C7C
```

这部分为反汇编窗口中设置内存断点部分的代码：

判断是否为访问断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
004238D3   > \83FB 23    cmp     ebx, 23
004238D6   .  75 22     jnz     short 004238FA
004238D8   .  8B0D 31DA4>mov     ecx, dword ptr [4CDA31]   ;  Case 23 of switch 00422DD4
004238DE   .  A1 2DDA4C0>mov     eax, dword ptr [4CDA2D]
004238E3   .  2B0D 2DDA4>sub     ecx, dword ptr [4CDA2D]
004238E9   .  51         push    ecx                   ; /Arg3
004238EA   .  50         push    eax                   ; |Arg2 => 00000000
004238EB   .  6A 03      push    3                     ; |Arg1 = 00000003
004238ED   .  E8 E659FFF>call    _Setmembreakpoint     ; \_Setmembreakpoint
004238F2   .  83C4 0C    add     esp, 0C
004238F5   .  E9 E224000>jmp     00425DDC
```

判断是否为写入断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
004238FA   >  83FB 24    cmp     ebx, 24
004238FD   .  75 23     jnz     short 00423922
004238FF   .  8B15 31DA4>mov     edx, dword ptr [4CDA31]   ;  Case 24 of switch 00422DD4
00423905   .  8B0D 2DDA4>mov     ecx, dword ptr [4CDA2D]
0042390B   .  2B15 2DDA4>sub     edx, dword ptr [4CDA2D]
00423911   .  52         push    edx                   ; /Arg3
00423912   .  51         push    ecx                   ; |Arg2 => 00000000
00423913   .  6A 02      push    2                     ; |Arg1 = 00000002
00423915   .  E8 BE59FFF>call    _Setmembreakpoint     ; \_Setmembreakpoint
0042391A   .  83C4 0C    add     esp, 0C
0042391D   .  E9 BA24000>jmp     00425DDC
```

判断是否为清除断点，并填入相应的参数让 `Setmembreakpoint` 函数处理：

```asm
00423922   >  83FB 25    cmp     ebx, 25
00423925   .  75 13     jnz     short 0042393A
00423927   .  6A 00      push    0                     ; /Arg3 = 00000000; Case 25 of switch 00422DD4
00423929   .  6A 00      push    0                     ; |Arg2 = 00000000
0042392B   .  6A 00      push    0                     ; |Arg1 = 00000000
0042392D   .  E8 A659FFF>call    _Setmembreakpoint     ; \_Setmembreakpoint
00423932   .  83C4 0C    add     esp, 0C
00423935   .  E9 A224000>jmp     00425DDC
```

下面就来详细的分析 `Setmembreakpoint` 这个函数：

首先将参数赋值给寄存器，并得到内存断点所在的内存页：

```asm
004192D8 >/$  55         push    ebp
004192D9  |.   8BEC       mov     ebp, esp
004192DB  |.   53         push    ebx
004192DC  |.   56         push    esi
004192DD  |.   57         push    edi
004192DE  |.   833D 38814D00>cmp    dword ptr [4D8138], 0
004192E5  |.   8B7D 10    mov     edi, dword ptr [ebp+10]    内存断点长度
004192E8  |.   8B75 0C    mov     esi, dword ptr [ebp+C]    内存断点首地址赋值
004192EB  |.   8B5D 08    mov     ebx, dword ptr [ebp+8]    内存断点标识赋值
004192EE  |.   0F85 99000000 jnz    0041938D
004192F4  |.   833D D8364D00>cmp    dword ptr [4D36D8], 2
004192FB  |.   0F84 8C000000 je     0041938D
00419301  |.   56         push    esi                   ; /Arg1
00419302  |.   E8 41870400  call    _Findmemory         ; \_Findmemory
00419307  |.   59         pop     ecx
```

检查是否在系统内存处下断点，并弹出警告：



### 汇编代码：检查资源内存断点

```
00419308  .|  81FE 00000080  cmp     esi, 80000000     
0041930E  .|  72 24           jb      short 00419334  
00419310  .|  6A 14          push    14  
00419312  .|  68 CA224B00   push    004B22CA  
00419317  .|  68 2E224B00   push    004B222E  
0041931C  .|  A1 7C3B4D00   mov     eax, dword ptr [4D3B7C]     ; |
00419321  .|  50            push    eax  
00419322  .|  E8 EF610900   call    <jmp.&USER32.MessageBoxA>    ; \\MessageBoxA  
00419327  .|  83F8 06       cmp     eax, 6  
0041932A  .|  74 61         je      short 0041938D  
0041932C  .|  83C8 FF       or      eax, FFFFFFFF  
0041932F  .|  E9 EA000000   jmp     0041941E  
```

检查是否在资源内存处下断点，并弹出警告：

```
00419334  |>  85C0          test    eax, eax  
00419336  .|  74 2B        je      short 00419363  
00419338  .|  F640 0B 01    test    byte ptr [eax+B], 1  
0041933C  .|  74 25        je      short 00419363  
0041933E  .|  6A 14        push    14  
00419340  .|  68 A1234B00   push    004B23A1  
00419345  .|  68 EB224B00   push    004B22EB  
0041934A  .|  8B15 7C3B4D00 mov     edx, dword ptr [4D3B7C]     ; |
00419350  .|  52          push    edx  
00419351  .|  E8 C0610900   call    <jmp.&USER32.MessageBoxA>    ; \\MessageBoxA  
00419356  .|  83F8 06       cmp     eax, 6  
00419359  .|  74 32        je      short 0041938D  
0041935B  .|  83C8 FF       or      eax, FFFFFFFF  
0041935E  .|  E9 BB000000   jmp     0041941E  
```

检查是否在堆栈内存处下断点，并弹出警告：

```
00419363  |>  85C0          test    eax, eax  
00419365  .|  74 26        je      short 0041938D  
00419367  .|  F640 0B 04    test    byte ptr [eax+B], 4  
0041936B  .|  74 20        je      short 0041938D  
0041936D  .|  6A 10        push    10  
0041936F  .|  68 24244B00   push    004B2424  
00419374  .|  68 BF234B00   push    004B23BF  
00419379  .|  8B15 7C3B4D00 mov     edx, dword ptr [4D3B7C]     ; |
0041937F  .|  52          push    edx  
00419380  .|  E8 91610900   call    <jmp.&USER32.MessageBoxA>    ; \\MessageBoxA  
00419385  .|  83C8 FF       or      eax, FFFFFFFF  
00419388  .|  E9 91000000   jmp     0041941E  
```

函数中该部分主要设置内存属性：

```
0041938D  |>  \\E8 92FAFFFF   call    00418E24    该函数主要用于设置内存属性  
00419392  .|  85C0          test    eax, eax  
00419394  .|  74 08         je      short 0041939E  设置成功就去设置结构体，否则退出  
00419396  .|  83C8 FF       or      eax, FFFFFFFF  
00419399  .|  E9 80000000   jmp     0041941E  
```

函数中该部分主要用于设置断点结构体：

```
0041939E  |>  \\8BC6          mov     eax, esi  
004193A0  .|  8BD7          mov     edx, edi  
004193A2  .|  A3 3C814D00   mov     dword ptr [4D813C], eax  ;填写结构体第二个字节  
004193A7  .|  8BC8          mov     ecx, eax  
004193A9  .|  03C2          add     eax, edx  
004193AB  .|  81E1 00F0FFFF and     ecx, FFFFF000  
004193B1  .|  05 FF0F0000   add     eax, 0FFF  
004193B6  .|  8915 40814D00 mov     dword ptr [4D8140], edx    ;填写结构体第三个字节  
004193BC  .|  25 00F0FFFF   and     eax, FFFFF000  
004193C1  .|  890D 44814D00 mov     dword ptr [4D8144], ecx    ;填写结构体第四个字节  
004193C7  .|  F6C7 10       test    bh, 10  
004193CA  .|  A3 48814D00   mov     dword ptr [4D8148], eax  ;填写结构体第五个字节  
004193CF  .|  0F95C0        setne   al  
004193D2  .|  83E0 01       and     eax, 1  
004193D5  .|  83E3 03       and     ebx, 3  
004193D8  .|  A3 38814D00   mov     dword ptr [4D8138], eax  
004193DD  .|  C705 5C8D4D00>mov     dword ptr [4D8D5C], 1  
004193E7  .|  85DB          test    ebx, ebx  
004193E9  .|  74 04         je      short 004193EF  
004193EB  .|  85FF          test    edi, edi  
004193ED  .|  75 04         jnz     short 004193F3  
004193EF  |>  33C0          xor     eax, eax  
004193F1  .|  EB 2B         jmp     short 0041941E  
004193F3  |>  83FB 02       cmp     ebx, 2  
004193F6  .|  75 0C         jnz     short 00419404  
004193F8  .|  C705 4C814D00>mov     dword ptr [4D814C], 20    ;填写结构体第六个字节  
00419402  .|  EB 0A         jmp     short 0041940E  
00419404  |>  C705 4C814D00>mov     dword ptr [4D814C], 1  
```

设置完内存断点记录后，检查是否是运行状态，如果是则调用函数设置内存页属性：

```
0041940E  |>  833D 5C5A4D00>cmp     dword ptr [4D5A5C], 3  ;查看是否是运行状态  
00419415  .|  75 05         jnz     short 0041941C  ;不是则结束  
00419417  .|  E8 18FCFFFF   call    00419034    ;如果是则调用函数修改内存页属性  
```

恢复现场，返回 0：

```
0041941C  |>  33C0          xor     eax, eax  ;返回 0  
0041941E  |>  5F          pop     edi  
0041941F  .|  5E          pop     esi  
00419420  .|  5B          pop     ebx  
00419421  .|  5D          pop     ebp  
00419422  .\\  C3          retn  
```

### 设置部分的函数逻辑

其次是设置部分的函数：

1.  一些参数的检查，错误则返回 `-1` 并结束；
2.  检查 `VirtualQuery`、`VirtualProtectEx` 这两个函数是否加载成功，若失败则返回 `-1` 并结束；
3.  这里是一个循环体，用于设置内存断点范围内的内存页属性，主要流程如下：
    a)  初始化循环体，得到内存断点所在的第一个内存页地址，并检查其合法性；
    b)  加上内存页最小单位（`1000h`），检查是否大于等于内存断点的尾地址，若小于则得到其内存断点的范围减去 `1000h`，用于下次判断；
    c)  一些标志、长度的判断以及设置；
    d)  按内存断点的范围设置内存页属性，将内存页原来的属性保存到两个数组中去，并检查是否设置成功，若设置失败则跳出循环体；
    e)  判断判断是否设置完内存断点范围内的所有内存页，并检查是否超出最大内存断点范围（`1000h*100h`），若超过或者设置完则退出循环，否则跳转到 b 步骤；
4.  检查设置内存页属性后其地址是否等于内存断点范围首地址，若等于说明一开始就设置失败，删除内存断点表，并给出相应的提示以及弹出窗口；
5.  检查设置内存页属性后其地址是否大于等于内存断点范围尾地址，若小于说明有一部分设置失败，缩小内存断点的范围，并给出相应的提示以及弹出窗口；

该函数我命名为 `SetMembpoint`，主要用于 `F9` 运行后内地断点的设置，下面来详细分析：

首先是一些参数的检查，错误则跳转到错误处理处：

```
00419034  /$  53            push    ebx  
00419035  .|  56            push    esi  
00419036  .|  57            push    edi  
00419037  .|  55            push    ebp  
00419038  .|  81C4 E4FDFFFF   add     esp, -21C  
0041903E  .|  BD 588D4D00     mov     ebp, 004D8D58  
00419043  .|  833D 685A4D00 0>cmp     dword ptr [4D5A68], 0    ; 检查线程句柄  
0041904A  .|  74 12         je      short 0041905E  
0041904C  .|  833D 40814D00 0>cmp     dword ptr [4D8140], 0    ; 检查断点长度  
00419053  .|  74 09         je      short 0041905E  
00419055  .|  833D 34814D00 0>cmp     dword ptr [4D8134], 0    ; 检查是否已经检查的标识  
0041905C  .|  74 07         je      short 00419065    ; 若正确则跳过下面的错误处理  
```

错误处理，返回 `-1` 并跳转到结束处：

```
0041905E  |>  33C0          xor     eax, eax  
00419060  .|  E9 66020000     jmp     004192CB  ; 跳转到结束处  
```

下面检查 `VirtualQuery`、`VirtualProtectEx` 这两个函数是否加载成功：

```
00419065  |>  833D 145A4D00 0>cmp     dword ptr [4D5A14], 0  ; 检查 VirtualQuery  
0041906C  .|  74 09         je      short 00419077  
0041906E  .|  833D 185A4D00 0>cmp     dword ptr [4D5A18], 0    ; 检查 VirtualProtectEx  
00419075  .|  75 08         jnz     short 0041907F    ; 若正确则跳过下面的错误处理  
```

错误处理，返回 `-1` 并跳转到结束处：

```
00419077  |>  83C8 FF       or      eax, FFFFFFFF  
0041907A  .|  E9 4C020000     jmp     004192CB  
```

通过内存断点所在页的首地址得到其内存相关信息结构体 `t_memory`，检查是否取得成功；并检查该内存断点所在页是否是 `TY_GUARDED`（被保护状态），若是则跳转到最后检查是否比较完处，初始化循环体：

```
0041907F  |>  33D2          xor     edx, edx  
00419081  .|  8955 00       mov     dword ptr [ebp], edx  
00419084  .|  A1 44814D00     mov     eax, dword ptr [4D8144]  
00419089  .|  8BD8          mov     ebx, eax  
0041908B  .|  8125 4C814D00 F>and     dword ptr [4D814C], FFFFFEFF>  
00419095  .|  50          push    eax                 ; /Arg1  
00419096  .|  E8 AD890400     call    _Findmemory         ; \\_Findmemory  
0041909B  .|  59          pop     ecx  
0041909C  .|  85C0          test    eax, eax    ;检查是否取得成功  
0041909E  .|  0F84 25010000   je      004191C9  
004190A4  .|  F640 0B 20      test    byte ptr [eax+B], 20  ; 检查是否为 TY_GUARDED  
004190A8  .|  0F84 1B010000   je      004191C9  
004190AE  .|  810D 4C814D00 0>or      dword ptr [4D814C], 100  ; 将写入标志与 0x100 或  
004190B8  .|  E9 0C010000     jmp     004191C9  
```

首先取得内存断点所在的首内存页的内存属性：

```
004190BD  |>  6A 1C         /push    1C  
004190BF  .|  8D8424 04020000 |lea     eax, dword ptr [esp+204]  
004190C6  .|  50          |push    eax  
004190C7  .|  53          |push    ebx  
004190C8  .|  8B15 685A4D00   |mov     edx, dword ptr [4D5A68]  
004190CE  .|  52          |push    edx  
004190CF  .|  FF15 145A4D00   |call    dword ptr [4D5A14]        ;  kernel32.VirtualQueryEx  
```



加上内存页最小单位（`1000h`），检查是否已经包含了内存断点的范围：

```
004190D5  |.  8B9424 0C020000 |mov     edx, dword ptr [esp+20C]  

004190DC  |.  8BCA            |mov     ecx, edx  

004190DE  |.  03CB            |add     ecx, ebx    ; ecx ：BP_Begin_Page + 1000  

004190E0  |.  A1 48814D00     |mov     eax, dword ptr [4D8148]  ; eax ：BreakPoint_End_Page  

004190E5  |.  3BC8            |cmp     ecx, eax    ; 检查是否包含内存断点的范围  

004190E7  |.  73 04           |jnb     short 004190ED  ; 不包含则跳到下面处理代码  

004190E9  |.  8BF2            |mov     esi, edx    ; esi ：内存对齐最小单位  

004190EB  |.  EB 04           |jmp     short 004190F1 ; 跳过处理代码  

若没有完全包含，则得到其内存断点的范围：  

004190ED  |>  8BF0            |mov     esi, eax  

004190EF  |.  2BF3            |sub     esi, ebx  

下面是一些标志位的比较以及设置：  

004190F1  |>  833D 38814D00 0>|cmp     dword ptr [4D8138], 0  

004190F8  |.  74 1B           |je      short 00419115  

004190FA  |.  3B1D 50814D00   |cmp     ebx, dword ptr [4D8150]  

00419100  |.  73 13           |jnb     short 00419115  

00419102  |.  8D041E          |lea     eax, dword ptr [esi+ebx]  

00419105  |.  3B05 50814D00   |cmp     eax, dword ptr [4D8150]  

0041910B  |.  76 08           |jbe     short 00419115  

0041910D  |.  8B35 50814D00   |mov     esi, dword ptr [4D8150]  

00419113  |.  2BF3            |sub     esi, ebx  

00419115  |>  833D 38814D00 0>|cmp     dword ptr [4D8138], 0  

0041911C  |.  74 1B           |je      short 00419139  

0041911E  |.  3B1D 54814D00   |cmp     ebx, dword ptr [4D8154]  

00419124  |.  73 13           |jnb     short 00419139  

00419126  |.  8D041E          |lea     eax, dword ptr [esi+ebx]  

00419129  |.  3B05 54814D00   |cmp     eax, dword ptr [4D8154]  

0041912F  |.  76 08           |jbe     short 00419139  

00419131  |.  8B35 54814D00   |mov     esi, dword ptr [4D8154]  

00419137  |.  2BF3            |sub     esi, ebx  

00419139  |>  833D 38814D00 0>|cmp     dword ptr [4D8138], 0  

00419140  |.  74 2A           |je      short 0041916C  

00419142  |.  3B1D 50814D00   |cmp     ebx, dword ptr [4D8150]  

00419148  |.  75 22           |jnz     short 0041916C  

0041914A  |.  BE 00100000     |mov     esi, 1000  

0041914F  |.  8B55 00        |mov     edx, dword ptr [ebp]  

00419152  |.  8B8424 14020000 |mov     eax, dword ptr [esp+214]  

00419159  |.  890495 58814D00 |mov     dword ptr [edx*4+4D8158],>  

00419160  |.  8B4D 00        |mov     ecx, dword ptr [ebp]  

00419163  |.  89048D 58854D00 |mov     dword ptr [ecx*4+4D8558],>  

0041916A  |.  EB 4E           |jmp     short 004191BA  

0041916C  |>  833D 38814D00 0>|cmp     dword ptr [4D8138], 0  

00419173  |.  74 14           |je      short 00419189  

00419175  |.  3B1D 54814D00   |cmp     ebx, dword ptr [4D8154]  

0041917B  |.  75 0C           |jnz     short 00419189  

0041917D  |.  BE 00100000     |mov     esi, 1000  

00419182  |.  BF 02000000     |mov     edi, 2  

00419187  |.  EB 06           |jmp     short 0041918F  

根据内存断点表设置内存页属性，并检查是否设置成功：  

00419189  |>  8B3D 4C814D00   |mov     edi, dword ptr [4D814C]  

0041918F  |>  8B45 00        |mov     eax, dword ptr [ebp]  

00419192  |.  8B15 685A4D00   |mov     edx, dword ptr [4D5A68]  

00419198  |.  C1E0 02         |shl     eax, 2  

0041919B  |.  81C0 58814D00   |add     eax, 004D8158  

004191A1  |.  50             |push    eax  

004191A2  |.  57             |push    edi  

004191A3  |.  56             |push    esi  

004191A4  |.  53             |push    ebx  

004191A5  |.  52             |push    edx  

004191A6  |.  FF15 185A4D00  |call    dword ptr [4D5A18]        ;  kernel32.VirtualProtectEx  

004191AC  |.  85C0            |test    eax, eax    ;检查是否设置成功  

004191AE  |.  74 2E           |je      short 004191DE  ;设置失败跳到错误处理处  

将内存页原属性放到全局变量数组中去，并将计数器加一：  

004191B0  |.  8B4D 00        |mov     ecx, dword ptr [ebp]  

004191B3  |.  893C8D 58854D00 |mov     dword ptr [ecx*4+4D8558],>  

004191BA  |>  8B45 00        |mov     eax, dword ptr [ebp]  

004191BD  |.  893485 58894D00 |mov     dword ptr [eax*4+4D8958],>  

004191C4  |.  03DE            |add     ebx, esi  

004191C6  |.  FF45 00        |inc     dword ptr [ebp]  

循环体条件检查部分，设定了内存断点的最大范围是 `1000h*100h`，若超过了则不再设置内存断点；另外检查是否已经设置完内存断点范围内的所有内存页，若没有设置完继续设置：  

004191C9  |>  817D 00 0001000> cmp     dword ptr [ebp], 100  ;检查内存断点是否超过最大范围  

004191D0  |.  7D 0C           |jge     short 004191DE  

004191D2  |.  3B1D 48814D00   |cmp     ebx, dword ptr [4D8148] ; 检查是否设置完内存属性  

004191D8  |.^ 0F82 DFFEFFFF   \jb     004190BD    ; 若没有设置完则继续  

检查设置内存页属性结构体的循环体结束后，其地址是否等于内存断点范围的首内存页地址，若不等于则跳过下面的错误处理：  

004191DE  |>  3B1D 44814D00   cmp     ebx, dword ptr [4D8144]    ;   

004191E4  |.  75 64           jnz     short 0041924A  

这里为设置内存断点失败的处理处，得到内存无法设置的范围，并给出错误提示和弹出窗口：  

004191E6  |.  8D9424 00010000 lea     edx, dword ptr [esp+100]  

004191ED  |.  52             push    edx                     ; /Arg3  

004191EE  |.  8B0D 40814D00   mov     ecx, dword ptr [4D8140]    ; |  

004191F4  |.  51             push    ecx                     ; |Arg2 => 00000002  

004191F5  |.  A1 3C814D00     mov     eax, dword ptr [4D813C]    ; |  

004191FA  |.  50             push    eax                     ; |Arg1 => 00400533  

004191FB  |.  E8 E04D0400     call    _Decoderange              ; \\_Decoderange  

00419200  |.  83C4 0C         add     esp, 0C  

00419203  |.  8D9424 00010000 lea     edx, dword ptr [esp+100]  

0041920A  |.  52             push    edx                     ; /Arg3  

0041920B  |.  68 FA204B00     push    004B20FA         ; |ollydbg 在地址范围 %s 中无法激活内存断点。断点已经被完全删除。  

00419210  |.  8D4C24 08       lea     ecx, dword ptr [esp+8]     ; |  

00419214  |.  51             push    ecx                     ; |Arg1  

00419215  |.  E8 12DA0800     call    004A6C2C                 ; \\_sprintf  

0041921A  |.  83C4 0C         add     esp, 0C  

0041921D  |.  6A 10           push    10                      ; /Style = MB_OK  

0041921F  |.  68 61214B00     push    004B2161             ; |无法激活内存中的断点  

00419224  |.  8D4424 08       lea     eax, dword ptr [esp+8]     ; |  

00419228  |.  50             push    eax                     ; |Text  

00419229  |.  8B15 7C3B4D00   mov     edx, dword ptr [4D3B7C]    ; |  

0041922F  |.  52             push    edx                     ; |hOwner  

00419230  |.  E8 E1620900     call    <jmp.&USER32.MessageBoxA>  ; \\MessageBoxA  

将内存页属性还原回来，将内存断点长度设置为 0，返回 -1 并退出：  

00419235  |.  E8 EAFBFFFF     call    00418E24  

0041923A  |.  33C9            xor     ecx, ecx  

0041923C  |.  83C8 FF        or      eax, FFFFFFFF  

0041923F  |.  890D 40814D00   mov     dword ptr [4D8140], ecx  

00419245  |.  E9 81000000     jmp     004192CB  

检查设置内存页属性结构体的循环体结束后，其地址是否大于等于内存断点范围的尾内存页地址，若大于则跳过下面的错误处理：  

0041924A  |>  3B1D 48814D00   cmp     ebx, dword ptr [4D8148]    ;  Ollydbg.<模块入口点>  

00419250  |.  73 6D           jnb     short 004192BF  

这里为设置内存断点范围尾部设置失败的错误处理处，设置相应的错误提示和弹出窗口：  

00419252  |.  8D9424 00010000 lea     edx, dword ptr [esp+100]  

00419259  |.  52             push    edx                     ; /Arg3  

0041925A  |.  8B0D 40814D00   mov     ecx, dword ptr [4D8140]    ; |  

00419260  |.  51             push    ecx                     ; |Arg2 => 00000002  

00419261  |.  A1 3C814D00     mov     eax, dword ptr [4D813C]    ; |  

00419266  |.  50             push    eax                     ; |Arg1 => 00400533  

00419267  |.  E8 744D0400     call    _Decoderange              ; \\_Decoderange  

0041926C  |.  83C4 0C         add     esp, 0C  

0041926F  |.  8BD3            mov     edx, ebx  

00419271  |.  4A             dec     edx  

00419272  |.  52             push    edx                     ; /Arg5  

00419273  |.  8B0D 3C814D00   mov     ecx, dword ptr [4D813C]    ; |Ollydbg.00400533  
```



### 内存断点范围调整与消息提示

```
00419279  .|  51              push    ecx                         ; |Arg4 => 00400533
0041927A  .|  8D8424 08010000 lea     eax, dword ptr [esp+108]    ; |
00419281  .|  50              push    eax                         ; |Arg3
00419282  .|  68 86214B00    push    004B2186                    ; |ollydbg 在全部指定的地址范围（%s）中无法激活内存断点。断点已经减少范围为 %08lx..%08lx.
00419287  .|  8D5424 10      lea     edx, dword ptr [esp+10]      ; |
0041928B  .|  52              push    edx                         ; |Arg1
0041928C  .|  E8 9BD90800    call    004A6C2C                    ; \\_sprintf
00419291  .|  83C4 14        add     esp, 14
00419294  .|  6A 10          push    10                          ; /Style = MB_OK
00419296  .|  68 0E224B00    push    004B220E                    ; |内存断点的范围被减少
0041929B  .|  8D4C24 08      lea     ecx, dword ptr [esp+8]       ; |
0041929F  .|  51              push    ecx                         ; |Text
004192A0  .|  A1 7C3B4D00    mov     eax, dword ptr [4D3B7C]     ; |
004192A5  .|  50              push    eax                         ; |hOwner
004192A6  .|  E8 6B620900    call    <jmp.&USER32.MessageBoxA>  ; \\MessageBoxA
```

将内存断点的范围缩小为设置失败前已经成功设置的内存页范围：

```
004192AB  .|  8BD3            mov     edx, ebx
004192AD  .|  2B15 3C814D00   sub     edx, dword ptr [4D813C]     ; Ollydbg.00400533
004192B3  .|  8915 40814D00   mov     dword ptr [4D8140], edx
004192B9  .|  891D 48814D00   mov     dword ptr [4D8148], ebx
```

设置检查标识为 1，标识已经检查完毕，然后返回 0 并退出：

```
004192BF  |>  C705 34814D00 0>mov     dword ptr [4D8134], 1
004192C9  .|  33C0            xor     eax, eax
004192CB  |>  81C4 1C020000   add     esp, 21C
004192D1  .|  5D              pop     ebp
004192D2  .|  5F              pop     edi
004192D3  .|  5E              pop     esi
004192D4  .|  5B              pop     ebx
004192D5  \.  C3              retn
```

### 内存断点处理流程说明

最后是内存断点的处理，该部分用 `WaitForDebugEvent` 函数捕获了内存断点异常，然后对内存断点进行检查以及 OD 界面进行相关设置，由于之后没有用 `ContinueDebugEvent` 函数，被调试程序被断在了该异常处：

1.  得到线程信息结构体，将结构体指针赋值给局部变量；
2.  根据异常信息结构体中的异常事件标识，跳转到相应的异常的处理分支处；
3.  给异常地址变量赋值，若得到线程信息，则根据线程信息的 `eip` 赋值，否则根据异常地址赋值，并且将异常地址赋值给几个全局变量；
4.  得到异常所在的模块信息，提取异常所在处的汇编代码，并检查模块信息是否正确，汇编代码有没有获得，根据情况做相应的错误处理；
5.  若是模块为自解压（SFX）模式，则进行相应的检查以及错误处理；
6.  检查内存断点是否在 `kernel32.dll` 中，弹出提示窗口，并将断点去除；
7.  最后调整优先级并退出

这里是所有异常的处理函数，除了内存断点相关的代码，其他都会被省略：

#### 首先保存环境，拉伸栈空间

```
0042EBD0  /$  55             push    ebp
0042EBD1  .|  8BEC          mov     ebp, esp
0042EBD3  .|  81C4 04F0F>add     esp, -0FFC
0042EBD9  .|  50             push    eax
0042EBDA  .|  81C4 00F5F>add     esp, -0B00
```

得到调试线程的线程信息结构体，该函数在《OD 获得当前 cpu 工作状态的方法》中已详细分析：

```
0042EBE0  .|  53             push    ebx
0042EBE1  .|  56             push    esi
0042EBE2  .|  57             push    edi
0042EBE3  .|  8B35 1C574>mov     esi, dword ptr [4D571C]  ; DebugEvent.dwThreadId
0042EBE9  .|  56             push    esi
0042EBEA  .|  E8 5DF8FFF>call    0042E44C       ; GetThreadInfo
```

#### 将得到的寄存器结构体指针赋值给参数

```
0042EBEF  .|  8BF8          mov     edi, eax
0042EBF1  .|  8B45 08      mov     eax, dword ptr [ebp+8]
0042EBF4  .|  59             pop     ecx
0042EBF5  .|  8938          mov     dword ptr [eax], edi  ; [ebp+8] = pReg
```

#### 根据异常事件标识跳转到相应的处理函数

```
0042EBF7  .|  8B15 14574>mov     edx, dword ptr [4D5714] ; DebugEvent.dwDebugEventCode
0042EBFD  .|  83FA 09      cmp     edx, 9                ;  Switch (cases 1..9)
0042EC00  .|  0F87 EE270>ja       004313F4
0042EC06  .|  FF2495 0DE>jmp      dword ptr [edx*4+42EC0D]
```

#### 调试异常的跳表

```
0042EC0D  .|  F4134300     dd      Ollydbg.004313F4    ; Default
0042EC11  .|  35EC4200     dd      Ollydbg.0042EC35    ; EXCEPTION_DEBUG_EVENT
0042EC15  .|  FF0C4300     dd      Ollydbg.00430CFF    ; CREATE_THREAD_DEBUG_EVENT
0042EC19  .|  D70D4300     dd      Ollydbg.00430DD7  ; CREATE_PROCESS_DEBUG_EVENT
0042EC1D  .|  3F0F4300     dd      Ollydbg.00430F3F    ; EXIT_THREAD_DEBUG_EVENT
0042EC21  .|  37104300     dd      Ollydbg.00431037    ; EXIT_PROCESS_DEBUG_EVENT
0042EC25  .|  2D114300     dd      Ollydbg.0043112D   ; LOAD_DLL_DEBUG_EVENT
0042EC29  .|  B7114300     dd      Ollydbg.004311B7    ; UNLOAD_DLL_DEBUG_EVENT
0042EC2D  .|  76124300     dd      Ollydbg.00431276    ; OUTPUT_DEBUG_STRING_EVENT
0042EC31  .|  C7134300     dd      Ollydbg.004313C7    ; RIP_EVENT
```

#### 处理调试异常

这里为调试异常处理处，给异常标识赋值，并将异常结构体指针赋值给局部变量：

```
0042EC35  |>  8B0D 0C364>mov     ecx, dword ptr [4E360C] ;
0042EC3B  .|  33C0          xor     eax, eax
0042EC3D  .|  894D EC      mov     dword ptr [ebp-14], ecx
0042EC40  .|  A3 0C364E0>mov     dword ptr [4E360C], eax
0042EC45  .|  C745 A4 20>mov     dword ptr [ebp-5C], 004D5720  ; DebugEvent.u
```

检查异常信息结构体中寄存器结构体是否得到，若得到则跳过下面取寄存器信息：

```
0042EC4C  .|  85FF          test    edi, edi
0042EC4E  .|  75 0D        jnz     short 0042EC5D
```

若没有得到了寄存器信息，则将异常信息中异常地址赋值给参数：

```
0042EC50  .|  8B55 A4      mov     edx, dword ptr [ebp-5C]
0042EC53  .|  33DB          xor     ebx, ebx
0042EC55  .|  8B4A 0C      mov     ecx, dword ptr [edx+C]
0042EC58  .|  894D D8      mov     dword ptr [ebp-28], ecx  ; 参数赋值为 ExceptionAddress
0042EC5B  .|  EB 22        jmp     short 0042EC7F
```

若得到了寄存器信息，则将寄存器中的 `EIP` 地址赋值给参数，检查异常标识，若不为 0 则修改寄存器里面的标识，并设置寄存器已被修改：

```
0042EC5D  |>  8B47 2C      mov     eax, dword ptr [edi+2C]
0042EC60  .|  8945 D8      mov     dword ptr [ebp-28], eax  ; 参数赋值为 t_reg.r_eip
0042EC63  .|  837D EC 00  cmp     dword ptr [ebp-14], 0    ; 检查异常标识
0042EC67  .|  8B5F 10      mov     ebx, dword ptr [edi+10]    ; ebx : t_reg.r_ecx
0042EC6A  .|  74 13        je      short 0042EC7F      ; 若标识为 0 则跳过下面的操作
0042EC6C  .|  F647 31 01  test    byte ptr [edi+31], 1
0042EC70  .|  74 0D        je      short 0042EC7F
0042EC72  .|  8167 30 FF>and      dword ptr [edi+30], FFFFFE>  ; 修改标识
0042EC79  .|  C707 01000>mov      dword ptr [edi], 1    ; 设置寄存器修改标识
```

检查是否是单步异常，若是则跳过下面的代码，若不是则跳转到下面继续判断是什么异常：

```
0042EC7F  |>  8B45 A4      mov     eax, dword ptr [ebp-5C]
0042EC82  .|  8138 03000>cmp      dword ptr [eax], 80000003
0042EC88  .|  74 07        je      short 0042EC91
0042EC8A  .|  33D2          xor     edx, edx
0042EC8C  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042EC8F  .|  EB 79        jmp     short 0042ED0A
```

跳过对单步的分析，不是重点。。。：

```
0042EC91  |>  6A 02        push    2                       ; /Arg4 = 00000002
0042EC93  .|  6A 01        push    1                       ; |Arg3 = 00000001
0042EC95  .|  8B4D D8      mov     ecx, dword ptr [ebp-28]    ; |
0042EC98  .|  49             dec     ecx                      ; |
0042EC99  .|  51             push    ecx                      ; |Arg2
0042EC9A  .|  8D45 BB      lea     eax, dword ptr [ebp-45]    ; |
0042EC9D  .|  50             push    eax                      ; |Arg1
0042EC9E  .|  E8 6926030>call    _Readmemory               ; \\_Readmemory
0042ECA3  .|  83C4 10      add     esp, 10
0042ECA6  .|  83F8 01      cmp     eax, 1
0042ECA9  .|  74 07        je      short 0042ECB2
0042ECAB  .|  33D2          xor     edx, edx
0042ECAD  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042ECB0  .|  EB 58        jmp     short 0042ED0A
0042ECB2  |>  33C0          xor     eax, eax
0042ECB4  .|  8A45 BB      mov     al, byte ptr [ebp-45]
0042ECB7  .|  3D CC00000>cmp      eax, 0CC
0042ECBC  .|  75 09        jnz     short 0042ECC7
0042ECBE  .|  C745 DC 01>mov      dword ptr [ebp-24], 1
0042ECC5  .|  EB 43        jmp     short 0042ED0A
0042ECC7  |>  83F8 03      cmp     eax, 3
0042ECCA  .|  74 07        je      short 0042ECD3
0042ECCC  .|  33D2          xor     edx, edx
0042ECCE  .|  8955 DC      mov     dword ptr [ebp-24], edx
0042ECD1  .|  EB 37        jmp     short 0042ED0A
0042ECD3  |>  6A 02        push    2                       ; /Arg4 = 00000002
0042ECD5  .|  6A 01        push    1                       ; |Arg3 = 00000001
0042ECD7  .|  8B4D D8      mov     ecx, dword ptr [ebp-28]    ; |
```



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



`004300AC  |.  3B8A E1030>cmp     ecx, dword ptr [edx+3E1] ; 入口点是否在代码段后面`

`004300B2  |.  76 47      jbe     short 004300FB`

`004300B4  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]`

`004300B7  |.  8B81 E1030>mov     eax, dword ptr [ecx+3E1]`

`004300BD  |.  3B45 D8    cmp     eax, dword ptr [ebp-28]  ;入口点与内存断点地址是否相同`

`004300C0  |.  74 0A      je      short 004300CC`

`004300C2  |.  B8 0200000>mov     eax, 2      ; 返回 2`

`004300C7  |.  E9 5913000>jmp     00431425    ; 跳转到结束处`

设置 OD 的 CPU 显示信息，以及相关的提示信息，跳过错误处理：

```
004300CC  |>  68 0D80000>push    800D                  ; /Arg5 = 0000800D
004300D1  |.  6A 00      push    0                     ; |Arg4 = 00000000
004300D3  |.  6A 00      push    0                     ; |Arg3 = 00000000
004300D5  |.  8B55 D8    mov     edx, dword ptr [ebp-28]       ; |
004300D8  |.  52         push    edx                   ; |Arg2
004300D9  |.  6A 00      push    0                     ; |Arg1 = 00000000
004300DB  |.  E8 38D5FFF>call    _Setcpu               ; \\_Setcpu
004300E0  |.  83C4 14    add     esp, 14
004300E3  |.  E8 E4E4FEF>call    _Redrawdisassembler
004300E8  |.  68 F7614B0>push    004B61F7          ; /自解压（sfx）代码理论上的入口点
004300ED  |.  8B4D D8    mov     ecx, dword ptr [ebp-28]    ; |
004300F0  |.  51         push    ecx                   ; |Arg1
004300F1  |.  E8 0AE0FFF>call    0042E100             ; \\SetInfoMsg
004300F6  |.  83C4 08    add     esp, 8
004300F9  |.  EB 34      jmp     short 0043012F
```

错误处理，检查一些参数，返回 2 并退出：

```
004300FB  |>  8B9D 84EBF>mov     ebx, dword ptr [ebp-147C]
00430101  |.  81E3 F0000>and     ebx, 0F0
00430107  |.  81FB 80000>cmp     ebx, 80
0043010D  |.  74 05      je      short 00430114
0043010F  |.  83FB 50    cmp     ebx, 50
00430112  |.  75 0A      jnz     short 0043011E
00430114  |>  B8 0200000>mov     eax, 2    ; 返回 2
00430119  |.  E9 0713000>jmp     00431425  ; 跳转到结束处
```

设置提示信息，并根据清除内存断点表的信息，最后检查是否清除成功，否则跳转到错误处理：

```
0043011E  |>  68 17624B0>push    004B6217              ; /自解压（sfx）代码可能的入口点
00430123  |.  8B55 D8    mov     edx, dword ptr [ebp-28]    ; |
00430126  |.  52         push    edx                   ; |Arg1
00430127  |.  E8 D4DFFFF>call    0042E100             ; \\ SetInfoMsg
0043012C  |.  83C4 08    add     esp, 8
0043012F  |>  6A 00      push    0                     ; /Arg3 = 00000000
00430131  |.  6A 00      push    0                     ; |Arg2 = 00000000
00430133  |.  6A 00      push    0                     ; |Arg1 = 00000000
00430135  |.  E8 9E91FEF>call    _Setmembreakpoint     ; \\_Setmembreakpoint
0043013A  |.  83C4 0C    add     esp, 0C
0043013D  |.  837D AC 00 cmp     dword ptr [ebp-54], 0
00430141  |.  74 71      je      short 004301B4
```

写 UDD 文件，判断模块信息结构体中的 SFX 代码的入口点是否存在，若不存在则将内存断点的首地址赋值给他：

```
00430143  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]
00430146  |.  8B01       mov     eax, dword ptr [ecx]
00430148  |.  3B05 64734>cmp     eax, dword ptr [4D7364]
0043014E  |.  75 64      jnz     short 004301B4
00430150  |.  6A 00      push    0                     ; /Arg4 = 00000000
00430152  |.  6A 00      push    0                     ; |Arg3 = 00000000
00430154  |.  6A 00      push    0                     ; |Arg2 = 00000000
00430156  |.  8B55 AC    mov     edx, dword ptr [ebp-54]       ; |
00430159  |.  52         push    edx                   ; |Arg1
0043015A  |.  E8 91DF020>call    0045E0F0             ; \\ write_udd
0043015F  |.  83C4 10    add     esp, 10
00430162  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]
00430165  |.  83B9 E1030>cmp     dword ptr [ecx+3E1], 0
0043016C  |.  74 09      je      short 00430177
0043016E  |.  833D 688D4>cmp     dword ptr [4D8D68], 0
00430175  |.  75 0C      jnz     short 00430183
00430177  |>  8B45 AC    mov     eax, dword ptr [ebp-54]
0043017A  |.  8B55 D8    mov     edx, dword ptr [ebp-28]
0043017D  |.  8990 E1030>mov     dword ptr [eax+3E1], edx
```

检查模块的类型以及基地址，若错误则重新得到其模块信息，检查有没有得到模块信息，若没有得到则分析模块得到其信息：

```
00430183  |>  833D 0C924>cmp     dword ptr [4D920C], 0
0043018A  |.  74 28      je      short 004301B4
0043018C  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]
0043018F  |.  F641 08 04 test    byte ptr [ecx+8], 4
00430193  |.  74 1F      je      short 004301B4
00430195  |.  6A 00      push    0                     ; /Arg2 = 00000000
00430197  |.  8B45 AC    mov     eax, dword ptr [ebp-54]    ; |
0043019A  |.  8B50 0C    mov     edx, dword ptr [eax+C]     ; |
0043019D  |.  52         push    edx                   ; |Arg1
0043019E  |.  E8 5DDD020>call    _Finddecode           ; \\_Finddecode
004301A3  |.  83C4 08    add     esp, 8
004301A6  |.  85C0       test    eax, eax
004301A8  |.  75 0A      jnz     short 004301B4
004301AA  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]
004301AD  |.  51         push    ecx
004301AE  |.  E8 D1F0040>call    _Analysecode
004301B3  |.  59         pop     ecx
```

设置断点表的第一个变量为 0，并跳转到下面调整优先级：

```
004301B4  |>  33C0       xor     eax, eax
004301B6  |.  A3 38814D0>mov     dword ptr [4D8138], eax
004301BB  |.  EB 15      jmp     short 004301D2
```

设置断点相关的信息，并调整优先级，最后返回 0 退出：

```
004301BD  |>  8B55 D8    mov     edx, dword ptr [ebp-28]
004301C0  |.  52         push    edx                   ; /Arg3
004301C1  |.  68 38624B0>push    004B6238          ; |执行 [%08lx] 时设置内存断点
004301C6  |.  8B4D D8    mov     ecx, dword ptr [ebp-28]    ; |
004301C9  |.  51         push    ecx                   ; |Arg1
004301CA  |.  E8 31DFFFF>call    0042E100             ; \\Ollydbg.0042E100
004301CF  |.  83C4 0C    add     esp, 0C
004301D2  |>  6A 00      push    0                     ; /Arg1 = 00000000
004301D4  |.  E8 FF18000>call    _Animate            ; \\_Animate
004301D9  |.  59         pop     ecx
004301DA  |.  33C0       xor     eax, eax     ; 返回 0
004301DC  |.  E9 4412000>jmp     00431425    ; 跳转到结束处
```

这里为读取机器码失败之后的判断部分，为一个循环体，初始化 `ebx` 值为 0，跳转到下面的循环体判断处：

```
004301E1  |>  33DB       xor     ebx, ebx
004301E3  |.  8D85 CCEBF>lea     eax, dword ptr [ebp-1434]
004301E9  |.  E9 0C02000>jmp     004303FA
```

检查异常事件的信息是否正确：

```
004301EE  |>  8338 00    /cmp     dword ptr [eax], 0  eax : pDebugEvent
004301F1  |.  0F84 FF010>|je      004303F6  ; 跳转到循环体判断处
004301F7  |.  F640 F4 20 |test    byte ptr [eax-C], 20    ; 检查是否是内存断点异常
004301FB  |.  0F85 F5010>|jnz     004303F6  ; 跳转到循环体判断处
00430201  |.  8B50 18    |mov     edx, dword ptr [eax+18] ; 检查异常地址是否正确
00430204  |.  8B0D 3C814>|mov     ecx, dword ptr [4D813C]
0043020A  |.  030D 40814>|add     ecx, dword ptr [4D8140]
00430210  |.  3BD1       |cmp     edx, ecx
00430212  |.  0F83 DE010>|jnb     004303F6  ; 跳转到循环体判断处
00430218  |.  0310       |add     edx, dword ptr [eax]
0043021A  |.  3B15 3C814>|cmp     edx, dword ptr [4D813C]   ;  Ollydbg.0040100C
00430220  |.  0F86 D0010>|jbe     004303F6  ; 跳转到循环体判断处
00430226  |.  833D 38814>|cmp     dword ptr [4D8138], 0
0043022D  |.  0F84 85010>|je      004303B8
00430233  |.  833D 50814>|cmp     dword ptr [4D8150], 0   ; exception_addr_begin_page
0043023A  |.  74 15      |je      short 00430251
0043023C  |.  8B50 18    |mov     edx, dword ptr [eax+18] ; ExceptionAddress
0043023F  |.  81E2 00F0F>|and     edx, FFFFF000
00430245  |.  3B15 50814>|cmp     edx, dword ptr [4D8150] ; 检查异常地址首内存页
0043024B  |.  0F84 A5010>|je      004303F6
```

检查异常调试信息结构体以及异常地址是否正确：

```
00430251  |>  833D 54814>|cmp     dword ptr [4D8154], 0  ; exception_addr_begin_page
00430258  |.  74 1E      |je      short 00430278
0043025A  |.  8B4D A4    |mov     ecx, dword ptr [ebp-5C]  ; ExceptionInformation
```



```
0043025D  |.  8379 14 00 |cmp     dword ptr [ecx+14], 0  
00430261  |.  75 15      |jnz     short 00430278  
00430263  |.  8B50 18    |mov     edx, dword ptr [eax+18]  ; ExceptionAddress  
00430266  |.  81E2 00F0F>|and     edx, FFFFF000  
0043026C  |.  3B15 54814>|cmp     edx, dword ptr [4D8154]  ; exception_addr_begin_page  
00430272  |.  0F84 7E010>|je      004303F6  
00430278  |>   8B4D A4    |mov     ecx, dword ptr [ebp-5C]  
0043027B  |.  8379 14 00 |cmp     dword ptr [ecx+14], 0    ; ExceptionInformation  
0043027F  |.  75 14      |jnz     short 00430295  
```

将异常地址的内存页首地址赋值给两个全局变量：

```
00430281  |.  8B40 18    |mov     eax, dword ptr [eax+18]  ; ExceptionAddress  
00430284  |.  25 00F0FFF>|and     eax, FFFFF000  
00430289  |.  A3 403B4E0>|mov     dword ptr [4E3B40], eax  
0043028E  |.  A3 54814D0>|mov     dword ptr [4D8154], eax  
00430293  |.  EB 15      |jmp     short 004302AA  
00430295  |>   8B50 18    |mov     edx, dword ptr [eax+18]  
00430298  |.  81E2 00F0F>|and     edx, FFFFF000  
0043029E  |.  8915 443B4>|mov     dword ptr [4E3B44], edx  
004302A4  |.  8915 50814>|mov     dword ptr [4D8150], edx  
```

比较几个全局变量中代表的异常内存页首地址是否一样，若不一样跳转到下面比较：

```
004302AA  |>   A1 403B4E0>|mov     eax, dword ptr [4E3B40]  
004302AF  |.  3B05 483B4>|cmp     eax, dword ptr [4E3B48]  
004302B5  |.  75 12      |jnz     short 004302C9  
004302B7  |.  8B0D 443B4>|mov     ecx, dword ptr [4E3B44]  
004302BD  |.  3B0D 4C3B4>|cmp     ecx, dword ptr [4E3B4C]  
004302C3  |.  0F84 8A000>|je      00430353  
```

设置提示信息：

```
004302C9  |>   68 61624B0>|push    004B6261           ; /跟踪 sfx：  
004302CE  |.  8D85 98FDF>|lea     eax, dword ptr [ebp-268]  ; |  
004302D4  |.  50         |push    eax                 ; |Arg1  
004302D5  |.  E8 5269070>|call    004A6C2C           ; \sprintf  
004302DA  |.  83C4 08    |add     esp, 8  
004302DD  |.  8945 F4    |mov     dword ptr [ebp-C], eax  
004302E0  |.  833D 54814>|cmp     dword ptr [4D8154], 0  
004302E7  |.  74 21      |je      short 0043030A  
004302E9  |.  8B15 403B4>|mov     edx, dword ptr [4E3B40]  
004302EF  |.  8D8D 98FDF>|lea     ecx, dword ptr [ebp-268]  
004302F5  |.  52         |push    edx                 ; /Arg3 => 00000000  
004302F6  |.  68 6E624B0>|push    004B626E           ; | read=%08x  
004302FB  |.  034D F4    |add     ecx, dword ptr [ebp-C]    ; |  
004302FE  |.  51         |push    ecx                 ; |Arg1  
004302FF  |.  E8 2869070>|call    004A6C2C           ; \sprintf  
00430304  |.  83C4 0C    |add     esp, 0C  
00430307  |.  0145 F4    |add     dword ptr [ebp-C], eax  
0043030A  |>   833D 50814>|cmp     dword ptr [4D8150], 0  
00430311  |.  74 1D      |je      short 00430330  
00430313  |.  A1 443B4E0>|mov     eax, dword ptr [4E3B44]  
00430318  |.  8D95 98FDF>|lea     edx, dword ptr [ebp-268]  
0043031E  |.  50         |push    eax                 ; /Arg3 => 00000000  
0043031F  |.  68 79624B0>|push    004B6279           ; | write=%08x  
00430324  |.  0355 F4    |add     edx, dword ptr [ebp-C]    ; |  
00430327  |.  52         |push    edx                 ; |Arg1  
00430328  |.  E8 FF68070>|call    004A6C2C           ; \sprintf  
0043032D  |.  83C4 0C    |add     esp, 0C  
00430330  |>   8D8D 98FDF>|lea     ecx, dword ptr [ebp-268]  
00430336  |.  51         |push    ecx                 ; /Arg1  
00430337  |.  E8 2C14000>|call    _Infoline          ; \_Infoline  
0043033C  |.  59         |pop     ecx  
0043033D  |.  A1 403B4E0>|mov     eax, dword ptr [4E3B40]  
00430342  |.  A3 483B4E0>|mov     dword ptr [4E3B48], eax  
00430347  |.  8B15 443B4>|mov     edx, dword ptr [4E3B44]  
0043034D  |.  8915 4C3B4>|mov     dword ptr [4E3B4C], edx  
```

检查模块信息结构体是否正确：

```
00430353  |>   833D 648D4>|cmp     dword ptr [4D8D64], 2  
0043035A  |.  74 43      |je      short 0043039F  
0043035C  |.  833D 648D4>|cmp     dword ptr [4D8D64], 1  
00430363  |.  75 49      |jnz     short 004303AE  
00430365  |.  833D 688D4>|cmp     dword ptr [4D8D68], 0  
0043036C  |.  74 40      |je      short 004303AE  
0043036E  |.  837D AC 00 |cmp     dword ptr [ebp-54], 0  
00430372  |.  74 3A      |je      short 004303AE  
00430374  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]  
00430377  |.  8B55 AC    |mov     edx, dword ptr [ebp-54]  
0043037A  |.  8B81 E1030>|mov     eax, dword ptr [ecx+3E1]  
00430380  |.  3B42 0C    |cmp     eax, dword ptr [edx+C]  ; 检查入口点是否在模块首前面  
00430383  |.  72 29      |jb      short 004303AE  
00430385  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]  
00430388  |.  8B55 AC    |mov     edx, dword ptr [ebp-54]  
0043038B  |.  8B41 0C    |mov     eax, dword ptr [ecx+C]  
0043038E  |.  8B4D AC    |mov     ecx, dword ptr [ebp-54]  
00430391  |.  0382 E9030>|add     eax, dword ptr [edx+3E9]  
00430397  |.  3B81 E1030>|cmp     eax, dword ptr [ecx+3E1] ; 检查入口点是否在模块尾的后面  
0043039D  |.  76 0F      |jbe     short 004303AE  
```

将临时变量置空：

```
0043039F  |>   33C0       |xor     eax, eax  
004303A1  |.  33D2       |xor     edx, edx  
004303A3  |.  A3 54814D0>|mov     dword ptr [4D8154], eax  
004303A8  |.  8915 50814>|mov     dword ptr [4D8150], edx  
```

错误处理的部分：

```
004303AE  |>   B8 0200000>|mov     eax, 2       ; 返回 2  
004303B3  |.  E9 6D10000>|jmp     00431425     ; 跳转到结束处  
```

设置内存断点的提示信息：

```
004303B8  |>   8B55 A4    |mov     edx, dword ptr [ebp-5C]  
004303BB  |.  837A 14 00 |cmp     dword ptr [edx+14], 0  
004303BF  |.  75 14      |jnz     short 004303D5  
004303C1  |.  57         |push    edi                 ; /Arg3  
004303C2  |.  68 85624B0>|push    004B6285          ; |读取 [%08lx] 时设置内存断点  
004303C7  |.  8B4D D8    |mov     ecx, dword ptr [ebp-28]   ; |  
004303CA  |.  51         |push    ecx                 ; |Arg1  
004303CB  |.  E8 30DDFFF>|call    0042E100           ; \SetInfoMsg  
004303D0  |.  83C4 0C    |add     esp, 0C  
004303D3  |.  EB 12      |jmp     short 004303E7  
004303D5  |>   57         |push    edi                 ; /Arg3  
004303D6  |.  68 AC624B0>|push    004B62AC          ; |写到 [%08lx] 时设置内存断点  
004303DB  |.  8B45 D8    |mov     eax, dword ptr [ebp-28]   ; |  
004303DE  |.  50         |push    eax                 ; |Arg1  
004303DF  |.  E8 1CDDFFF>|call    0042E100           ; \ SetInfoMsg  
004303E4  |.  83C4 0C    |add     esp, 0C  
```

调整优先级：

```
004303E7  |>   6A 00      |push    0                   ; /Arg1 = 00000000  
004303E9  |.  E8 EA16000>|call    _Animate            ; \_Animate  
004303EE  |.  59         |pop     ecx  
004303EF  |.  33C0       |xor     eax, eax    ; 返回 0  
004303F1  |.  E9 2F10000>|jmp     00431425   ; 跳转到结束处  
```

循环体判断条件以及自加部分，小于三的时候继续比较：

```
004303F6  |>   43         |inc     ebx  
004303F7  |.  83C0 04    |add     eax, 4  
004303FA  |>   83FB 03     cmp     ebx, 3  
004303FD  |.  7D 0A      |jge     short 00430409  
```

检查读取机器码的长度，如果大于 0 则继续跳转到上面判断，否则就结束程序，返回 2：

```
004303FF  |.  837D F4 00 |cmp     dword ptr [ebp-C], 0  
00430403  |.^ 0F8F E5FDF>\\jg      004301EE  
```

设置错误返回值，并退出

```
00430409  |>   FF05 303B4>inc      dword ptr [4E3B30]  
0043040F  |.  B8 0200000>mov      eax, 2    ; 设置返回值为 2  
00430414  |.  E9 0C10000>jmp      00431425  ; 跳转到结束处  
```

设置错误信息到全局变量，返回 2 并退出：

```
00430419  |>   833D 5C8D4>cmp      dword ptr [4D8D5C], 0  
00430420  |.  74 19      je       short 0043043B  
00430422  |.  830D 74574>or       dword ptr [4D5774], 20  
00430429  |.  33D2       xor      edx, edx  
0043042B  |.  8915 5C8D4>mov      dword ptr [4D8D5C], edx  
00430431  |.  B8 0200000>mov      eax, 2      ; 返回 2  
00430436  |.  E9 EA0F000>jmp      00431425    ; 跳转到结束处  
```

检查内存断点是否在 `kernel32` 模块中，若是则弹出错误提示窗口：

```
0043043B  |>   810D 74574>or       dword ptr [4D5774], 204  
00430445  |.  833D A0574>cmp       dword ptr [4D57A0], 0  
```



### 1. 异常处理与模块查找

```
0043044C  |.  74 52      je      short 004304A0
0043044E  |.  8B55 D8    mov     edx, dword ptr [ebp-28]
00430451  |.  52         push    edx                   ; /Arg1
00430452  |.  E8 C1D9020>call    _Findmodule           ; \\_Findmodule
00430457  |.  59         pop     ecx
00430458  |.  8945 AC    mov     dword ptr [ebp-54], eax
0043045B  |.  837D AC 00 cmp     dword ptr [ebp-54], 0
0043045F  |.  74 3F      je      short 004304A0
00430461  |.  6A 08      push    8                     ; /Arg3 = 00000008
00430463  |.  68 D6624B0>push    004B62D6              ; |kernel32
00430468  |.  8B4D AC    mov     ecx, dword ptr [ebp-54]    ; |
0043046B  |.  83C1 48    add     ecx, 48             ; |
0043046E  |.  51         push    ecx                   ; |Arg1
0043046F  |.  E8 E034070>call    004A3954              ; \\Ollydbg.004A3954
00430474  |.  83C4 0C    add     esp, 0C
00430477  |.  85C0       test    eax, eax
00430479  |.  75 25      jnz     short 004304A0
0043047B  |.  68 DF624B0>push    004B62DF    ; /访问违反在 kernel32 中，根据请求已忽略
00430480  |.  8B45 D8    mov     eax, dword ptr [ebp-28]    ; |
00430483  |.  50         push    eax                   ; |Arg1
00430484  |.  E8 A711000>call    _Message              ; \\_Message
00430489  |.  83C4 08    add     esp, 8
0043048C  |.  C705 FC564>mov     dword ptr [4D56FC], 2
00430496  |.  B8 0100000>mov     eax, 1        ; 返回 1
0043049B  |.  E9 850F000>jmp     00431425    ; 跳到程序结束处
```

根据上面设置的变量，设置相关的错误信息以及弹出窗口：

### 2. 错误信息设置

```
004304A0  |>  8B55 A4    mov     edx, dword ptr [ebp-5C]
004304A3  |.  837A 10 02 cmp     dword ptr [edx+10], 2
004304A7  |.  72 3E      jb      short 004304E7
004304A9  |.  68 C6614B0>push   004B61C6 ; $使用“shift+f7/f8/f9”键跳过异常以继续执行程序
004304AE  |.  57         push    edi
004304AF  |.  8B4D A4    mov     ecx, dword ptr [ebp-5C]
004304B2  |.  8379 14 01 cmp     dword ptr [ecx+14], 1
004304B6  |.  75 07      jnz     short 004304BF
004304B8  |.  B8 32634B0>mov     eax, 004B6332           ; 写入到
004304BD  |.  EB 11      jmp     short 004304D0
004304BF  |>  3B7D D8    cmp     edi, dword ptr [ebp-28]
004304C2  |.  75 07      jnz     short 004304CB
004304C4  |.  B8 3D634B0>mov     eax, 004B633D           ; 正在执行
004304C9  |.  EB 05      jmp     short 004304D0
```

设置错误提示：

```
004304CB  |>  B8 47634B0>mov     eax, 004B6347           ; 读取
004304D0  |>  50         push    eax                   ; |Arg3
004304D1  |.  68 0F634B0>push    004B630F              ; |访问违反：%s [%08lx]%s
004304D6  |.  8D95 98FDF>lea     edx, dword ptr [ebp-268]       ; |
004304DC  |.  52         push    edx                   ; |Arg1
004304DD  |.  E8 4A67070>call    004A6C2C              ; \\_sprintf
004304E2  |.  83C4 14    add     esp, 14
004304E5  |.  EB 19      jmp     short 00430500
```

设置错误提示：

```
004304E7  |>  68 C6614B0>push    004B61C6              ; /Arg3 = 004B61C6
004304EC  |.  68 4F634B0>push    004B634F              ; |Arg2 = 004B634F
004304F1  |.  8D8D 98FDF>lea     ecx, dword ptr [ebp-268]       ; |
004304F7  |.  51         push    ecx                   ; |Arg1
004304F8  |.  E8 2F67070>call    004A6C2C              ; \\_sprintf
004304FD  |.  83C4 0C    add     esp, 0C
```

在 `OD` 的子窗口中设置各类错误信息：

### 3. 设置子窗口信息

```
00430500  |>  8D85 98FDF>lea     eax, dword ptr [ebp-268]
00430506  |.  50         push    eax                   ; /Arg2
00430507  |.  8B55 D8    mov     edx, dword ptr [ebp-28]    ; |
0043050A  |.  52         push    edx                   ; |Arg1
0043050B  |.  E8 F0DBFFF>call    0042E100              ; \\SetInfoMsg
00430510  |.  83C4 08    add     esp, 8
```

检查一些全局变量，并做相应的处理：

### 4. 全局变量检查与处理

```
00430513  |.  C705 84574>mov     dword ptr [4D5784], C0000005
0043051D  |.  833D AC574>cmp     dword ptr [4D57AC], 0
00430524  |.  75 16      jnz     short 0043053C
00430526  |.  833D 38814>cmp     dword ptr [4D8138], 0
0043052D  |.  74 09      je      short 00430538
0043052F  |.  833D 6C8D4>cmp     dword ptr [4D8D6C], 0
00430536  |.  75 04      jnz     short 0043053C
00430538  |>  33C9       xor     ecx, ecx
0043053A  |.  EB 05      jmp     short 00430541
0043053C  |>  B9 0100000>mov     ecx, 1
00430541  |>  51         push    ecx                   ; /Arg2
00430542  |.  68 050000C>push    C0000005              ; |Arg1 = C0000005
00430547  |.  E8 2CE6FFF>call    0042EB78              ; \\Ollydbg.0042EB78
0043054C  |.  83C4 08    add     esp, 8
0043054F  |.  85C0       test    eax, eax
00430551  |.  74 14      je      short 00430567
00430553  |.  C705 FC564>mov     dword ptr [4D56FC], 3
0043055D  |.  B8 0200000>mov     eax, 2        ; 返回 2
00430562  |.  E9 BE0E000>jmp     00431425    ; 跳转到结束处
```

调整优先级：

### 5. 调整优先级与恢复现场

```
00430567  |>  C705 FC564>mov     dword ptr [4D56FC], 2
00430571  |.  6A 00      push    0                     ; /Arg1 = 00000000
00430573  |.  E8 6015000>call    _Animate              ; \\_Animate
00430578  |.  59         pop     ecx
00430579  |.  33C0       xor     eax, eax    ; 返回 0
0043057B  |.  E9 A50E000>jmp     00431425    ; 跳转到结束处
```

下面是对其他异常的处理，这里忽略：

```
……
```

最后恢复现场并退出：

```
00431425  |>  5F         pop     edi
00431426  |.  5E         pop     esi
00431427  |.  5B         pop     ebx
00431428  |.  8BE5       mov     esp, ebp
0043142A  |.  5D         pop     ebp
0043142B  \.  C3         retn
```

武汉科锐学员: `angelqkm`

2008-5-21
