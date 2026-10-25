
# 某著名绘图软件 XXXX Graphics Suite X5 算法分析 + 注册机

> 原文：[`www.pediy.com/kssd/pediy12/142857.html`](https://www.pediy.com/kssd/pediy12/142857.html)

**【文章标题】**: `XXXX Graphics Suite X5` 算法分析（一） `Serial code`

**【文章作者】**: `Daniel`

**【作者邮箱】**: `vvcracker@gmail.com`

**【操作系统】**: `Windows 7`

**【生产日期】**: `2010`

**【软件名称】**: `Graphics XXXXX XX SP3.v15.2.0.686`

**【软件介绍】**: 一款著名的绘图软件

**【加壳方式】**: 无

**【保护方式】**: 注册码

**【编写语言】**: `Microsoft Visual C++ 10.0`

**【使用工具】**: `OD` + `PEID` + `IDA Pro`

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请各位大牛小牛原谅!

**【讨论说明】**

本文只讨论核心验证算法,由于涉及到函数处理较多，我只会将最重要的处理过程作说明，其他不作说明。

列出 `md5`，`blowfish` 算法只列出调用地方，算法的运作也不作说明。

### 验证入口

```
0248344F  .|  51           push    ecx
02483450  .|  8D0D 62344802 lea     ecx, dword ptr ds:[2483462]
02483456  .|  83C1 01      add     ecx, 1
02483459  .|  51           push    ecx     [ecx+4]  Serial 码
0248345A  .|  E8 D1600100  call    02499530
```

进入 `02499530` 来到主验证过程，列出主要处理，

```
024834A4   .  FF75 0C      push    dword ptr ss:[ebp+C]                   ; /maxlen
024834A7   .  8B45 90      mov     eax, dword ptr ss:[ebp-70]             ; |
024834AA   .  FF70 04      push    dword ptr ds:[eax+4]                  ; |s2
024834AD   .  FF75 08      push    dword ptr ss:[ebp+8]                  ; |s1         验证是否有 DR15
024834B0   .  FF15 58F56502 call    dword ptr ds:[<&MSVCR90._strnicmp>]    ; \_strnicmp
024834F1   .  50           push    eax
024834F2   .  8B4D 90      mov     ecx, dword ptr ss:[ebp-70]
024834F5   .  E8 74FEFFFF  call    0248336E            验证输入 Serial code 前 7 个字符是否符合条件
02483516   .  50           push    eax
02483517   .  8B4D 90      mov     ecx, dword ptr ss:[ebp-70]
0248351A   .  E8 FAFFFF    call    02483319            验证输入 Serial code 后 21 个字符是否符合条件
0248354F   .  FF75 EC      push    dword ptr ss:[ebp-14]        长度
02483552   .  8B45 90      mov     eax, dword ptr ss:[ebp-70]
02483555   .  FF70 04      push    dword ptr ds:[eax+4]         [eax+4] 字符串
02483558   .  8B4D A4      mov     ecx, dword ptr ss:[ebp-5C]    转换结果[ ecx + 4 ]
0248355B   .  E8 74F40000  call    024929D4             将 Serial code 后 21 个字符转换为 15 个
```

### 进入 024929D4

```
02492A16  .|  8B5D 08      mov     ebx, dword ptr ss:[ebp+8]
02492A19  .|  6A 07        push    7
02492A1B  .|  53           push    ebx
02492A1C  .|  57           push    edi
02492A1D  .|  FF76 04      push    dword ptr ds:[esi+4]       先复制前 7 个字节
02492A20  .|  FF15 14F76502 call    dword ptr ds:[<&MSVCR90.memcpy_s>]    ;  MSVCR90.memcpy_s
02492A26  .|  83C4 20      add     esp, 20
02492A29  .|  8D47 FA      lea     eax, dword ptr ds:[edi-6]
02492A2C  .|  8945 FC      mov     dword ptr ss:[ebp-4], eax
02492A2F  .|  8D45 FC      lea     eax, dword ptr ss:[ebp-4]
02492A32  .|  50           push    eax
02492A33  .|  8B46 04      mov     eax, dword ptr ds:[esi+4]
02492A36  .|  83C0 07      add     eax, 7
02492A39  .|  50           push    eax                          eax 转换后的数组地址 B
02492A3A  .|  8B45 0C      mov     eax, dword ptr ss:[ebp+C]
02492A3D  .|  83C0 F9      add     eax, -7
02492A40  .|  50           push    eax
02492A41  .|  83C3 07      add     ebx, 7           长度
02492A44  .|  53           push    ebx              ebx 要转换的数组地址 A
02492A45  .|  E8 9F100000  call    02493AE9          将 21 个字符转换为 15 个字符
02492A4A  .|  33C0         xor     eax, eax
02492A4C  .|  3846 08      cmp     byte ptr ds:[esi+8], al
02492A4F  .|  5B           pop     ebx
02492A50  .|  0F95C0       setne   al
02492A53  .|  8D0485 140000> lea     eax, dword ptr ds:[eax*4+14]
02492A5A  .|  8946 0C      mov     dword ptr ds:[esi+C], eax
02492A5D  .|  3BF8         cmp     edi, eax
02492A5F  .|  76 1A        jbe     short 02492A7B
02492A61  .|  8D4F FF      lea     ecx, dword ptr ds:[edi-1]
02492A64  .|  3BC8         cmp     ecx, eax
02492A66  .|  72 13        jb      short 02492A7B
02492A68  .|  8B56 04      mov     edx, dword ptr ds:[esi+4]      转换后的结果要求第 15 个字符为 0，算法第一处验证
02492A6B  |>  803C0A 00    /cmp     byte ptr ds:[edx+ecx], 0
02492A6F  .|  75 07        |jnz     short 02492A78
02492A71  .|  49           |dec     ecx
02492A72  .|  3BC8         |cmp     ecx, eax
02492A74  .|^ 73 F5        \jnb     short 02492A6B
02492A76  .|  EB 03        jmp     short 02492A7B
02492A78  .|> 897E 0C      mov     dword ptr ds:[esi+C], edi       此值要求为 0
```

### 看 02493AE9 的处理过程

主要是移位操作，从要转换的数组 `A` 中，获取的每个值先转换为 5 位数值，然后填充转换后的数组 `B` 中，

先填充每个字节的高位，接下来的值填充低位，每个字节填充完毕后，才增加索引，如此循环，直到用完要转换的数组 `A` 的 21 个字符。

```
02493AE9  /$  55           push    ebp
02493AEA  .|  8BEC         mov     ebp, esp
02493AEC  .|  51           push    ecx
02493AED  .|  8B45 0C      mov     eax, dword ptr ss:[ebp+C]
02493AF0  .|  53           push    ebx
02493AF1  .|  6BC0 05      imul    eax, eax, 5          与 5 相乘
02493AF4  .|  56           push    esi
02493AF5  .|  57           push    edi
02493AF6  .|  8B7D 10      mov     edi, dword ptr ss:[ebp+10]
02493AF9  .|  6A 07        push    7
02493AFB  .|  5B           pop     ebx
02493AFC  .|  03C3         add     eax, ebx
02493AFE  .|  33F6         xor     esi, esi
02493B00  .|  C1E8 03      shr     eax, 3               除以 8，eax 为转换数组的长度 = 0xE
02493B03  .|  C645 FF 01   mov     byte ptr ss:[ebp-1], 1
02493B07  .|  3BFE         cmp     edi, esi             edi 为要转换数组的地址，简称 B
02493B09  .|  0F84 D9000000 je      02493BE8
02493B0F  .|  8B4D 14      mov     ecx, dword ptr ss:[ebp+14]
02493B12  .|  8B09         mov     ecx, dword ptr ds:[ecx]
02493B14  .|  3BC1         cmp     eax, ecx
02493B16  .|  76 09        jbe     short 02493B21
02493B18  .|  C645 FF 00   mov     byte ptr ss:[ebp-1], 0
02493B1C  .|  E9 C7000000  jmp     02493BE8
02493B21  |>  51           push    ecx                  ; /n
02493B22  .|  56           push    esi                  ; |c
02493B23  .|  57           push    edi                  ; |s
02493B24  .|  E8 853C0000  call    <jmp.&MSVCR90.memset>            ; \memset
02493B29  .|  33D2         xor     edx, edx
02493B2B  .|  83C4 0C      add     esp, 0C
02493B2E  .|  8975 10      mov     dword ptr ss:[ebp+10], esi
02493B31  .|  3955 0C      cmp     dword ptr ss:[ebp+C], edx
02493B34  .|  0F86 A2000000 jbe     02493BDC
02493B3A  |>  8B45 08      /mov     eax, dword ptr ss:[ebp+8]     eax 转换数组开始地址称为 A
02493B3D  .|  8B4D 10      |mov     ecx, dword ptr ss:[ebp+10]    ecx 索引
02493B40  .|  8A0C01       |mov     cl, byte ptr ds:[ecx+eax]       cl 获取数组的值
02493B43  .|  8AC1         |mov     al, cl
02493B45  .|  2C 32        |sub     al, 32               以下是判断该值属于哪个范围，然后减去相应的值
02493B47  .|  3AC3         |cmp     al, bl               经过观察，相减后，范围为 0-0x1f
02493B49  .|  77 05        |ja      short 02493B50
02493B4B  .|  80E9 1A      |sub     cl, 1A
02493B4E  .|  EB 42        |jmp     short 02493B92
02493B50  .|> 8AC1         |mov     al, cl
02493B52  .|  2C 41        |sub     al, 41
02493B54  .|  3AC3         |cmp     al, bl
02493B56  .|  76 3C        |jbe     short 02493B94
02493B58  .|  8AC1         |mov     al, cl
02493B5A  .|  2C 4A        |sub     al, 4A                ;  分支 (案例 4A..A4)
02493B5C  .|  3C 04        |cmp     al, 4
02493B5E  .|  77 05        |ja      short 02493B65
```



### 02493B60 分支处理

```
02493B60  |.  80E9 42       |sub     cl, 42                              ;  案例 4A,4B,4C,4D,4E --> 分支 02493B5A  
02493B63  |.  EB 2D         |jmp     short 02493B92  
02493B65  |>  8AC1          |mov     al, cl  
02493B67  |.  2C 50         |sub     al, 50  
02493B69  |.  3C 0A         |cmp     al, 0A  
02493B6B  |.  77 05         |ja      short 02493B72  
02493B6D  |.  80E9 43       |sub     cl, 43                              ;  案例 9A,9B,9C,9D,9E,9F,A0,A1,A2,A3,A4 --> 分支 02493B5A  
02493B70  |.  EB 20         |jmp     short 02493B92  
02493B72  |>  8AC1          |mov     al, cl                              ;  分支 02493B5A 默认案例  
02493B74  |.  2C 61         |sub     al, 61  
02493B76  |.  3AC3          |cmp     al, bl  
02493B78  |.  76 1A         |jbe     short 02493B94  
02493B7A  |.  8AC1          |mov     al, cl  
02493B7C  |.  2C 6A         |sub     al, 6A                              ;  分支 (案例 6A..E4)  
02493B7E  |.  3C 04         |cmp     al, 4  
02493B80  |.  77 05         |ja      short 02493B87  
02493B82  |.  80E9 62       |sub     cl, 62                              ;  案例 6A,6B,6C,6D,6E --> 分支 02493B7C  
02493B85  |.  EB 0B         |jmp     short 02493B92  
02493B87  |>  8AC1          |mov     al, cl  
02493B89  |.  2C 70         |sub     al, 70  
02493B8B  |.  3C 0A         |cmp     al, 0A  
02493B8D  |.  77 49         |ja      short 02493BD8  
02493B8F  |.  80E9 63       |sub     cl, 63                              ;  案例 DA,DB,DC,DD,DE,DF,E0,E1,E2,E3,E4 --> 分支 02493B7C  
02493B92  |>  8AC1          |mov     al, cl  
```

### 02493B94 状态比较与位运算

```
02493B94  |>  83FA 03       |cmp     edx, 3                  edx 比较重要，代表当前已使用位数的数值,判断是否大于 3  
02493B97  |.  77 12         |ja      short 02493BAB  
02493B99  |.  83C2 FD       |add     edx, -3                 
02493B9C  |.  23D3          |and     edx, ebx                计算当前值剩余位数  
02493B9E  |.  75 06         |jnz     short 02493BA6  
02493BA0  |.  08043E        |or      byte ptr ds:[esi+edi], al    直接赋值  
02493BA3  |.  46            |inc     esi                       增加 B 数组索引  
02493BA4  |.  EB 21         |jmp     short 02493BC7  
```

### 02493BA6 数据移位与赋值

```
02493BA6  |>  6A 08         |push    8  
02493BA8  |.  59            |pop     ecx  
02493BA9  |.  EB 15         |jmp     short 02493BC0  
02493BAB  |>  83C2 FD       |add     edx, -3  
02493BAE  |.  23D3          |and     edx, ebx                计算当前值剩余位数  
02493BB0  |.  8AD8          |mov     bl, al  
02493BB2  |.  8ACA          |mov     cl, dl  
02493BB4  |.  D2EB          |shr     bl, cl                  当前值向右移位  
02493BB6  |.  6A 08         |push    8  
02493BB8  |.  59            |pop     ecx                     ecx 为字节的位数 8  
02493BB9  |.  6A 07         |push    7  
02493BBB  |.  081C3E        |or      byte ptr ds:[esi+edi], bl   赋值 B 数组  
02493BBE  |.  46            |inc     esi                     B 数组索引加 1  
02493BBF  |.  5B            |pop     ebx  
02493BC0  |>  2BCA          |sub     ecx, edx                要移位的数值 = 每个字节的位数 - 当前已使用的位数  
02493BC2  |.  D2E0          |shl     al, cl                  当前值移位  
02493BC4  |.  08043E        |or      byte ptr ds:[esi+edi], al    赋值 B 数组  
```

### 02493BC7 循环控制与结束条件

```
02493BC7  |>  FF45 10       |inc     dword ptr ss:[ebp+10]    增加数组 A 的索引  
02493BCA  |.  8B45 10       |mov     eax, dword ptr ss:[ebp+10]  
02493BCD  |.  3B45 0C       |cmp     eax, dword ptr ss:[ebp+C]  
02493BD0  |.^ 0F82 64FFFFFF |jb      02493B3A  
02493BD6  |.  EB 04         |jmp     short 02493BDC  
02493BD8  |>  C645 FF 00    |mov     byte ptr ss:[ebp-1], 0          
02493BDC  |>  845D 0C       |test    byte ptr ss:[ebp+C], bl  
02493BDF  |.  74 01         |je      short 02493BE2  
```

这处比较难理解，要细心调试，我只是点明关键部分，关键还是要自己调试。

### 回到主验证过程

```
02483583   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
02483586   .  FF70 28       |push    dword ptr ds:[eax+28]  
02483589   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
0248358C   .  FF70 24       |push    dword ptr ds:[eax+24]  
0248358F   .  8B4D E0       |mov     ecx, dword ptr ss:[ebp-20]  
02483592   .  E8 2A050100   |call    02493AC1  
02483597   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
0248359A   .  FF70 1C       |push    dword ptr ds:[eax+1C]  
0248359D   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
024835A0   .  FF70 14       |push    dword ptr ds:[eax+14]  
024835A3   .  8B4D E0       |mov     ecx, dword ptr ss:[ebp-20]  
024835A6   .  E8 D8F50000   |call    02492B83  
024835AB   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
024835AE   .  FF70 20       |push    dword ptr ds:[eax+20]  
024835B1   .  8B45 90       |mov     eax, dword ptr ss:[ebp-70]  
024835B4   .  FF70 18       |push    dword ptr ds:[eax+18]  
024835B7   .  8B4D E0       |mov     ecx, dword ptr ss:[ebp-20]  
024835BA   .  E8 04FBFFFF   |call    024830C3                      
024835BF   .  FF75 10       |push    dword ptr ss:[ebp+10]  
024835C2   .  8B4D E0       |mov     ecx, dword ptr ss:[ebp-20]  
024835C5   .  E8 27040100   |call    024939F1                    对 024929D4 产生的结果进行 blowfish 解密  
```

`02493AC1`，`02492B83`，`024830C3` 获取固定字符，经过以上处理后分别在 `ecx+0x10`,`ecx+0x14`,`ecx+0x24` 保存结果。

### 进入 024939F1，看主要处理过程

```
02493A12   .  8D4D EC       |lea     ecx, dword ptr ss:[ebp-14]  
02493A15   .  51            |push    ecx                       ecx 为 key  
02493A16   .  8BCE          |mov     ecx, esi  
02493A18   .  FF50 08       |call    dword ptr ds:[eax+8]      产生 blowfish key  
02493A1B   .  6A 10         |push    10  
02493A1D   .  8D45 EC       |lea     eax, dword ptr ss:[ebp-14]  
02493A20   .  50            |push    eax                       用 key 初始化  
02493A21   .  E8 35C2FCFF   |call    0245FC5B                  blowfish init  
02493A26   .  33C0          |xor     eax, eax  
02493A28   .  3846 08       |cmp     byte ptr ds:[esi+8], al  
02493A2B   .  0F95C0        |setne   al  
02493A2E   .  8D0485 140000>|lea     eax, dword ptr ds:[eax*4+14]  
02493A35   .  83E8 07       |sub     eax, 7  
02493A38   .  50            |push    eax  
02493A39   .  8B46 04       |mov     eax, dword ptr ds:[esi+4]  
02493A3C   .  83C0 07       |add     eax, 7  
02493A3F   .  50            |push    eax                       从第 7 个字节开始解密，一共解 8 个字节  
02493A40   .  E8 D1C2FCFF   |call    0245FD16                  blowfish decrypt  
```

### 进入 02493A18 的 md5 hash 过程，列出主要过程

```
02493508   .  6A 07         |push    7  
0249350A   .  FF76 04       |push    dword ptr ds:[esi+4]  
0249350D   .  57            |push    edi  
0249350E   .  8B3D 14F76502 |mov     edi, dword ptr ds:[<&MSVCR90.memcpy_s>]    
02493514   .  50            |push    eax                       复制输入 Serial Code 前 7 个字符，eax 为 md5 字符数组起始地址  
02493515   .  FFD7          |call    edi                        
02493517   .  FF76 28       |push    dword ptr ds:[esi+28]  
0249351A   .  8B45 20       |mov     eax, dword ptr ss:[ebp+20]  
0249351D   .  FF76 24       |push    dword ptr ds:[esi+24]  
02493520   .  83C0 F9       |add     eax, -7  
02493523   .  50            |push    eax  
02493524   .  8945 2C       |mov     dword ptr ss:[ebp+2C], eax  
02493527   .  8B45 18       |mov     eax, dword ptr ss:[ebp+18]  
0249352A   .  83C0 07       |add     eax, 7  
0249352D   .  50            |push    eax  
0249352E   .  FFD7          |call    edi                       复制系列号的值，此处固定值  
02493530   .  8B46 28       |mov     eax, dword ptr ds:[esi+28]  
02493533   .  2945 2C       |sub     dword ptr ss:[ebp+2C], eax  
02493536   .  8B4D 18       |mov     ecx, dword ptr ss:[ebp+18]  
02493539   .  6A 02         |push    2  
0249353B   .  83C0 07       |add     eax, 7  
```



```
0249353E   .  68 40397202   push    02723940                              

02493543   .  FF75 2C       push    dword ptr ss:[ebp+2C]  

02493546   .  8945 24      mov     dword ptr ss:[ebp+24], eax  

02493549   .  03C1         add     eax, ecx  

0249354B   .  50          push    eax  

0249354C   .  FFD7        call    edi                                     复制字符串“v2”  

0249356D   .  50          push    eax  

0249356E   .  FF76 10     push    dword ptr ds:[esi+10]        复制 esi+10 的字符串，跟 024835BA 处想同  

02493571   .  50          push    eax  

02493572   .  FF75 28     push    dword ptr ss:[ebp+28]  

02493575   .  FFD7        call    edi                                    

02493577   .  83C4 10     add     esp, 10  

0249357A   .  6A 04       push    4                     ; /Arg3 = 00000004  

0249357C   .  FF76 18     push    dword ptr ds:[esi+18]        ; |Arg2  

0249357F   .  FF75 28     push    dword ptr ss:[ebp+28]        ; |Arg1  

02493582   .  E8 72C6FCFF call    0245FBF9              此处经过一些变换，固定变换  

02493587   .  FF76 18     push    dword ptr ds:[esi+18]  

0249358A   .  8B4D 18     mov     ecx, dword ptr ss:[ebp+18]  

0249358D   .  FF75 28     push    dword ptr ss:[ebp+28]  

02493590   .  8B45 24     mov     eax, dword ptr ss:[ebp+24]  

02493593   .  FF75 2C     push    dword ptr ss:[ebp+2C]  

02493596   .  03C1         add     eax, ecx  

02493598   .  50          push    eax  

02493599   .  FFD7        call    edi                 复制到 md5 字符数组  

024935CD   .  8B46 1C     mov     eax, dword ptr ds:[esi+1C]     复制 esi+1C 的字符串，跟 024835BA 处想同  

024935D0   .  50          push    eax  

024935D1   .  FF76 14     push    dword ptr ds:[esi+14]        原始值处  

024935D4   .  50          push    eax  

024935D5   .  FF75 28     push    dword ptr ss:[ebp+28]  

024935D8   .  FFD7        call    edi                     ;  MSVCR90.memcpy_s  

024935DA   .  83C4 10     add     esp, 10  

024935DD   .  6A 05       push    5                     ; /Arg3 = 00000005  

024935DF   .  FF76 1C     push    dword ptr ds:[esi+1C]        ; |Arg2  

024935E2   .  FF75 28     push    dword ptr ss:[ebp+28]        ; |Arg1  

024935E5   .  E8 0FC6FCFF call    0245FBF9              此处经过一些变换，固定变换  

024935EA   .  FF76 1C     push    dword ptr ds:[esi+1C]  

024935ED   .  8B4D 18     mov     ecx, dword ptr ss:[ebp+18]  

024935F0   .  FF75 28     push    dword ptr ss:[ebp+28]  

024935F3   .  8B45 24     mov     eax, dword ptr ss:[ebp+24]  

024935F6   .  FF75 2C     push    dword ptr ss:[ebp+2C]  

024935F9   .  03C1         add     eax, ecx  

024935FB   .  50          push    eax  

024935FC   .  FFD7        call    edi                 复制到 md5 字符数组  

02493615   .  FF75 04     push    dword ptr ss:[ebp+4]        hash 保存地址  

02493618   .  FF75 20     push    dword ptr ss:[ebp+20]       长度  

0249361B   .  FF75 18     push    dword ptr ss:[ebp+18]       要 hash 数组  

0249361E   .  E8 131EFEFF call    02475436              md5  
```

这个过程主要`md5` `hash` 出来一个`key`给`blowfish` `init` 用，回到上层。

然后`Blowfish` `Init`，`Blowfish` `decrypt` 还原 8 个字符，这里不作详细说明。

回到主验证过程，

```
024835CA   .  FF75 10     push    dword ptr ss:[ebp+10]  

024835CD   .  8B4D E0     mov     ecx, dword ptr ss:[ebp-20]  

024835D0   .  E8 83030100 call    02493958              验证  
```

进入`02493958`

```
02493989  |.  53          push    ebx  

0249398A  |.  8B1D 14F76502 mov    ebx, dword ptr ds:[<&MSVCR90.memcpy_s>]   ;  MSVCR90.memcpy_s  

02493990  |.  56          push    esi  

02493991  |.  FF77 04     push    dword ptr ds:[edi+4]  

02493994  |.  56          push    esi  

02493995  |.  55          push    ebp          复制解密后的数据  

02493996  |.  FFD3        call    ebx          

02493998  |.  83C4 10     add     esp, 10  

0249399B  |.  FF7424 18   push    dword ptr ss:[esp+18]  

0249399F  |.  8BCF        mov     ecx, edi  

024939A1  |.  E8 EEFCFFFF call    02493694          

024939A6  |.  6A 07       push    7  

024939A8  |.  58          pop     eax  

024939A9  |.  3BF0        cmp     esi, eax  

024939AB  |.  76 17       jbe     short 024939C4  

024939AD  |.  8B4F 04     mov     ecx, dword ptr ds:[edi+4]         以下对比产生的值是否一致，从第 8 个字符开始，算法第二处验证  

024939B0  |>  8A1401      /mov    dl, byte ptr ds:[ecx+eax]       经 02493694 处理后的数据   

024939B3  |.  3A1428      |cmp    dl, byte ptr ds:[eax+ebp]        Blowfish 解密后的数据  

024939B6  |.  75 07       |jnz    short 024939BF  

024939B8  |.  40          |inc    eax  

024939B9  |.  3BC6        |cmp    eax, esi  

024939BB  |.^ 72 F3       \jb     short 024939B0  

024939BD  |.  EB 05       jmp     short 024939C4  

024939BF  |>  C64424 13 00 mov    byte ptr ss:[esp+13], 0  
```

看`02493694` 的主要过程有`02475481`，

```
024936BF   .  8D4D 50     lea     ecx, dword ptr ss:[ebp+50]  

024936C2   .  51          push    ecx  

024936C3   .  8BCE        mov     ecx, esi          

024936C5   .  FF50 08     call    dword ptr ds:[eax+8]          计算 key 值，与 02493A18 处相同  

024936C8   .  6A 10       push    10          

024936DA   .  6A 08       push    8           

024936DC   .  0F95C0      setne   al           

024936DF   .  6A 00       push    0           

024936E1   .  8D0485 140000>lea    eax, dword ptr ds:[eax*4+14]       

024936E8   .  8BF8        mov     edi, eax           

024936EA   .  8B46 04     mov     eax, dword ptr ds:[esi+4]         

024936ED   .  83C0 0C     add     eax, 0C           

024936F0   .  50          push    eax         清空解密后的数据后 8 个字节  

024936F1   .  E8 B8400000 call    <jmp.&MSVCR90.memset>             ; \memset  

024936F6   .  83C4 18     add     esp, 18  

024936F9   .  8D45 60     lea     eax, dword ptr ss:[ebp+60]  

024936FC   .  50          push    eax         eax 保存 02475481 处理后的结果  

024936FD   .  8D45 50     lea     eax, dword ptr ss:[ebp+50]        

02493700   .  50          push    eax         

02493701   .  57          push    edi         edi 数据长度 0x14  

02493702   .  FF76 04     push    dword ptr ds:[esi+4]       esi 数据起始地址  

02493705   .  E8 771DFEFF call    02475481          ; \CorelD_1.02475481  

024938E2   .  50          push    eax  

024938E3   .  53          push    ebx         ebx 保存结果  

024938E4   .  8D45 60     lea     eax, dword ptr ss:[ebp+60]  

024938E7   .  50          push    eax         eax 是 02475481 返回的 hash 数据  

024938E8   .  8BCE        mov     ecx, esi  

024938EA   .  E8 71FDFFFF call    02493660         

02493902   .  53          push    ebx  

02493903   .  83C0 0C     add     eax, 0C  

02493906   .  57          push    edi  

02493907   .  50          push    eax  

02493908   .  FF15 BCF46502 call   dword ptr ds:[<&MSVCR90.memmove_s>]       copy 经过 02493660 处理的结果 ，然后返回上层对比结果         

```

先进入`02475481`，看细致处理过程

```
024754B7  |.  6A 10       push    10          ; /n = 10 (16.)  

024754B9  |.  6A 36       push    36          ; |c = 36  ('6')  
```



```
024754BB  .|.  56        push    esi
024754BC  .|.  E8 ED220200   call    <jmp.&MSVCR90.memset>
024754C1  .|.  8D47 01       lea     eax, dword ptr ds:[edi+1]
024754C4  .|.  50        push    eax
024754C5  .|.  8D5E 10       lea     ebx, dword ptr ds:[esi+10]
024754C8  .|.  6A 00         push    0
024754CA  .|.  53        push    ebx
024754CB  .|.  E8 DE220200   call    <jmp.&MSVCR90.memset>
024754D0  .|.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]
024754D3  .|.  83C4 18       add     esp, 18
024754D6  .|.  8BC6          mov     eax, esi
024754D8  .|.  2BCE          sub     ecx, esi
024754DA  .|.  C745 E4 10000>mov     dword ptr ss:[ebp-1C], 10
024754E1  |>   8A1401        /mov    dl, byte ptr ds:[ecx+eax]
024754E4  .|.  3010          |xor    byte ptr ds:[eax], dl
024754E6  .|.  40        |inc    eax
024754E7  .|.  FF4D E4       |dec    dword ptr ss:[ebp-1C]
024754EA  .|^  75 F5         \jnz    short 024754E1
024754EC  .|.  57        push    edi
024754ED  .|.  FF75 E0       push    dword ptr ss:[ebp-20]
024754F0  .|.  57        push    edi
024754F1  .|.  53        push    ebx
024754F2  .|.  FF15 14F76502 call    dword ptr ds:[<&MSVCR90.memcpy_s>]
024754F8  .|.  83C4 10       add     esp, 10
024754FB  .|.  8D45 EC       lea     eax, dword ptr ss:[ebp-14]
024754FE  .|.  50        push    eax
024754FF  .|.  83C7 10       add     edi, 10
02475502  .|.  57        push    edi
02475503  .|.  56        push    esi
02475504  .|.  E8 2DFFFFFF   call    02475436
0247551E  .|.  6A 10         push    10
02475520  .|.  5F        pop     edi
02475521  .|.  57        push    edi
02475522  .|.  6A 5C         push    5C
02475524  .|.  56        push    esi
02475525  .|.  E8 84220200   call    <jmp.&MSVCR90.memset>
0247552A  .|.  8B4D 10       mov     ecx, dword ptr ss:[ebp+10]
0247552D  .|.  83C4 0C       add     esp, 0C
02475530  .|.  8BC6          mov     eax, esi
02475532  .|.  2BCE          sub     ecx, esi
02475534  .|.  8BD7          mov     edx, edi
02475536  |>   8A1C01        /mov    bl, byte ptr ds:[ecx+eax]
02475539  .|.  3018          |xor    byte ptr ds:[eax], bl
0247553B  .|.  40        |inc    eax
0247553C  .|.  4A        |dec    edx
0247553D  .|^  75 F7         \jnz    short 02475536
0247553F  .|.  57        push    edi
02475540  .|.  8D45 EC       lea     eax, dword ptr ss:[ebp-14]
02475543  .|.  50        push    eax
02475544  .|.  8D46 10       lea     eax, dword ptr ds:[esi+10]
02475547  .|.  57        push    edi
02475548  .|.  50        push    eax
02475549  .|.  FF15 BCF46502 call    dword ptr ds:[<&MSVCR90.memmove_s>]
0247554F  .|.  83C4 10       add     esp, 10
02475552  .|.  FF75 DC       push    dword ptr ss:[ebp-24]
02475555  .|.  6A 20         push    20
02475557  .|.  56        push    esi
02475558  .|.  E8 D9FEFFFF   call    02475436
```

再进入 `02493660`，这里主要将 hash 后的结果，前 8 位，与偶数的位异或，得到一个 8 字节的数组

```
02493678  |>   /8A1C4F       /mov    bl, byte ptr ds:[edi+ecx*2]
0249367B  .|. |321C06        |xor    bl, byte ptr ds:[esi+eax]
0249367E  .|. |41        |inc    ecx
0249367F  .|. |8818         |mov    byte ptr ds:[eax], bl
02493681  .|. |40        |inc    eax
02493682  .|. |3BCA         |cmp    ecx, edx
02493684  .|^  \72 F2         \jb    short 02493678
```

最后返回到 `024939AD` 对比结果。

### 【加密思路】

1. 对输入的 Serial code 后 21 个字符数组转换为 14 个字节数组，14 个字节数组简称 A，且最后一位要求为 0，具体算法看 `02493AE9` 的分析。
2. 输入的 Serial code 前 7 个字符调用 `02492CF8`，产生 BlowFish 的 Key，再调用 BlowFish Init；
3. Serial code 前 7 个字符与 A 组成新的数组 B，20 个字节，对 B 调用 BlowFish Decrypt，还原第 8 个字节起的 8 个字节，产生 C；
4. 数组 D 复制 C 数组，然后最后 8 个字节清零，然后调用 `02475481`，产生一个 hash，然后再调用 `02493660`，将 hash 变为一个 8 字节的数组，赋值到数组 D 的最后 8 个字节。
5. 数组 D 与数组 C 比较，要求值一致。

### 【解密思路】

1. 用字符串 `"DR15"` + 随机生成 8 位字节组成 12 位的数组 A，再加上 8 个值为零的字节组成 20 个字节数组 B;
2. 用 B 模拟调用 `02475481`，产生一个 hash，然后再模拟调用 `02493660`，将 hash 变为一个 8 字节的数组，然后将此数组与 A 合并组成 C；
3. 取 C 的前 7 个字节模拟调用 `02492CF8`，产生 BlowFish 的 Key，再调用 BlowFish Init；
4. 对 C 第 8 个字节起的 8 个字节，调用 BlowFish Encrypt，加密 8 个字节；
5. 对 C 第 8 个字节起的 13 个字节，加上一个值为零的字节，组成 14 个字节，用到 105 位的数据，要还原 21 个字符，每一次取 5 位的数据，按当前值的范围还原出原来的字符（还原规则看 `02493AE9`），最后组成 Serial code。

### 【结语】

这篇文章，只是列出关键的处理过程，只是向大家说明一个加密思路，细致的跟踪留给感兴趣的朋友们。

### 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢！

2011 年 11 月 16 日 by Daniel

【文章标题】: XXXX Graphics Suite X5 算法分析（二） activation code + 注册机

【文章作者】: Daniel

【作者邮箱】: vvcracker@gmail.com

【操作系统】: Windows 7

【生产日期】: 2010

【软件名称】: Graphics XXXXX XX SP3.v15.2.0.686

【软件介绍】: 一款著名的绘图软件

【加壳方式】: 无

【保护方式】: 注册码

【编写语言】: Microsoft Visual C++ 10.0

【使用工具】: OD + PEID + IDA Pro

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请各位大牛小牛原谅!

### 【讨论说明】

本文只讨论核心验证算法,由于涉及到函数处理较多，我只会将最重要的处理过程作说明，其他不作说明。

列出 sha，blowfish 算法只列出调用地方，算法的运作也不作说明。

#### 验证入口

```
100240ED    50            push    eax
100240EE    51            push    ecx
100240EF    E8 CCED0000   call    10032EC0
```

#### 过程 I: 硬件指纹

```
10051E94    8B7C24 48     mov     edi, dword ptr ss:[esp+48]
10051E98    8D5424 10     lea     edx, dword ptr ss:[esp+10]
10051E9C    52            push    edx
10051E9D    8BCE          mov     ecx, esi
10051E9F    C646 1A 01    mov     byte ptr ds:[esi+1A], 1
10051EA3    897E 0C       mov     dword ptr ds:[esi+C], edi
10051EA6    E8 45220000   call    100540F0
```

进入 `100540F0`

```
1005410D    56            push    esi
1005410E    53            push    ebx
1005410F    8BF1          mov     esi, ecx
10054111    E8 9AFEFFFF   call    10053FB0
1005413E    50            push    eax
1005413F    8D4C24 20     lea     ecx, dword ptr ss:[esp+20]
10054143    E8 C5320000   call    1005740D
10054148    BE A43D1010   mov     esi, 10103DA4
1005414D    33C0          xor     eax, eax
1005414F    897424 1C     mov     dword ptr ss:[esp+1C], esi
10054153    884424 34     mov     byte ptr ss:[esp+34], al
10054157    8D4C24 1C     lea     ecx, dword ptr ss:[esp+1C]
1005415B    51            push    ecx
1005415C    8D4C24 10     lea     ecx, dword ptr ss:[esp+10]
10054160    894424 44     mov     dword ptr ss:[esp+44], eax
10054164    E8 B5390000   call    10057B1E
10054169    6A 04         push    4
1005416B    8D5424 4C     lea     edx, dword ptr ss:[esp+4C]
```



## 硬件指纹计算流程

### 1. 初始调用链

进入 `10053FB0`，`10053FB0` -> `100BA7C0`，`->` 表示前者函数调用后者函数。

```
1005416F    52              push    edx                     edx 产品系列号的值，此值为固定值
10054170    8D4424 14       lea     eax, dword ptr ss:[esp+14]
10054174    50              push    eax                     eax 保存结果到 eax + C 中
10054175    8D4C24 28       lea     ecx, dword ptr ss:[esp+28]
10054179    C64424 4C 01    mov     byte ptr ss:[esp+4C], 1
1005417E    E8 292C0000     call    10056DAC                sha Hash
10054183    8B7C24 18       mov     edi, dword ptr ss:[esp+18]
10054187    8A0F            mov     cl, byte ptr ds:[edi]   ecx Hash 后的数据
10054189    880B            mov     byte ptr ds:[ebx], cl   ebx 组成最后的硬件指纹数组
1005418B    8B4C24 14       mov     ecx, dword ptr ss:[esp+14]
```

### 2. 读取硬件信息与 SHA Hash

`100BA998` 开始执行代码段：

```
100BA998    52              push    edx
100BA999    8859 04         mov     byte ptr ds:[ecx+4], bl
100BA99C    E8 9F76F4FF     call    10002040
100BA9A1    E8 CAFAFFFF     call    100BA470                读取 4 次硬件信息，分别进行 SHA Hash
100BAA73    50              push    eax
100BAA74    8859 04         mov     byte ptr ds:[ecx+4], bl
100BAA77    E8 C475F4FF     call    10002040
100BAA7C    E8 4FF3FFFF     call    100B9DD0                对 100BA470 产生 4 次 hash 信息，串联起来再进行一次 Hash
```

调用链 `100BA470` -> `100B96A0` -> `100BC8A0`，以下 4 个函数调用分别是读取本机硬件信息，分别作 SHA Hash：

```
100BC8D4    8BC8            mov     ecx, eax
100BC8D6    E8 05470000     call    100C0FE0
100BC908    8BC8            mov     ecx, eax
100BC90A    E8 E13C0000     call    100C05F0
100BC937    8BC8            mov     ecx, eax
100BC939    E8 92380000     call    100C01D0
100BC962    8BC8            mov     ecx, eax
100BC964    E8 C7240000     call    100BEE30
```

### 3. 第一处硬件信息 Hash（网卡）

先看 `100C0FE0`，接着这函数会调用 `100C0D20`。

```
100C0DB6    8D5424 10       lea     edx, dword ptr ss:[esp+10]
100C0DBA    52              push    edx
100C0DBB    57              push    edi                     edi 保存读取的值
100C0DBC    FFD6            call    esi                     GetAdaptersInfo 获取网卡的信息 A
100C0E02    8D8E 94010000   lea     ecx, dword ptr ds:[esi+194]
100C0E08    51              push    ecx                     ecx hash 网卡 0x194 处的信息，8 字节
100C0E09    8BC8            mov     ecx, eax
100C0E0B    E8 10250000     call    100C3320                准备进行 sha hash
```

`100C3320` -> `100C3140`，再看看 `100C3140` 具体如何进行 sha hash：

```
100C31F7    8B45 00         mov     eax, dword ptr ss:[ebp]
100C31FA    8BCD            mov     ecx, ebp
100C31FC    FF50 0C         call    dword ptr ds:[eax+C]    先读取该类存储的值
100C3282    50              push    eax
100C3283    6A 01           push    1
100C3285    52              push    edx
100C3286    8D4C24 20       lea     ecx, dword ptr ss:[esp+20]
100C328A    E8 A126F5FF     call    10015930                与网卡 0x194 处的信息组装最后的字符串
100C32A5    8B45 04         mov     eax, dword ptr ss:[ebp+4]
100C32A8    8D4D 04         lea     ecx, dword ptr ss:[ebp+4]
100C32AB    FF50 04         call    dword ptr ds:[eax+4]    最终进行 sha hash，hash 结果保存在 [ESI+C]+C 处
```

至此第一处的硬件信息 Hash 结束。

### 4. 第二处硬件信息 Hash（磁盘）

再看第二处的硬件信息 Hash，`100C05F0` -> `100C0360`。

`100C0360` 会获取系统的盘符并作为参数传给 `100C2450` 调用，`100C2450` -> `100C2090`，看 `100C2090` 的细致处理：

```
100C21CF    FF15 54321010   call    dword ptr ds:[<&KERNEL32.GetVolu]  kernel32.GetVolumeInformationA
100C21F8    52              push    edx
100C21F9    8D4C24 24       lea     ecx, dword ptr ss:[esp+24]
100C21FD    51              push    ecx
100C21FE    8D5424 1C       lea     edx, dword ptr ss:[esp+1C]
100C2202    52              push    edx
100C2203    8D4C24 1C       lea     ecx, dword ptr ss:[esp+1C]
100C2207    51              push    ecx
100C2208    50              push    eax
100C2209    FF15 A8321010   call    dword ptr ds:[<&KERNEL32.GetDisk]   kernel32.GetDiskFreeSpaceA
100C223C    8D8424 F4000000 lea     eax, dword ptr ss:[esp+F4]
100C2243    68 B0A71110     push    1011A7B0                ; ASCII "%ld%ld%ld%ld"
100C2248    50              push    eax
100C2249    E8 7161FEFF     call    100A83BF                将数字转化为字符
```

`100A83BF` 该处是将 `GetVolumeInformationA`，`GetDiskFreeSpaceA` 获取的 `SectorsPerCluster`，`BytesPerSector`，`VolumeSerialNumber` 的值转化为字符串并保存到 `eax`。

```
100C2260    50              push    eax
100C2261    E8 9A8CFFFF     call    100BAF00                100BAF00 读取该类存储的值 A
100C22D4    6A FF           push    -1
100C22D6    53              push    ebx
100C22D7    8D4424 54       lea     eax, dword ptr ss:[esp+54]
100C22DB    8969 18         mov     dword ptr ds:[ecx+18], ebp
100C22DE    8959 14         mov     dword ptr ds:[ecx+14], ebx
100C22E1    50              push    eax
100C22E2    8859 04         mov     byte ptr ds:[ecx+4], bl
100C22E5    E8 56FDF3FF     call    10002040                A 与上述字符串组装最后的字符串
100C22EA    8B56 04         mov     edx, dword ptr ds:[esi+4]
100C22ED    8D4E 04         lea     ecx, dword ptr ds:[esi+4]
100C22F0    FF52 08         call    dword ptr ds:[edx+8]    最终进行 sha hash，hash 结果保存在 [ESI+C]+C 处
```

### 5. 第三处硬件信息 Hash（硬盘）

再看第 3 处的硬件信息 Hash，`100C01D0` -> `100BFD70` -> `100BF590`，列出 `100BF590` 主要处理过程：

```
100BF609    51              push    ecx
100BF60A    FF15 98311010   call    dword ptr ds:[<&KERNEL32.CreateF]  创建硬盘句柄
100BF69E    FFD7            call    edi                 获取硬盘 SMART_GET_VERSION 的信息
100BF724    52              push    edx
100BF725    55              push    ebp
100BF726    E8 F5F7FFFF     call    100BEF20            获取硬盘 SMART_RCV_DRIVE_DATA 的信息，获取其硬盘 ID 号
100BF7EA    52              push    edx                 [edx + c] 为硬盘 ID 号的字符串
100BF7EB    50              push    eax
100BF7EC    E8 0FFDFFFF     call    100BF500            sha hash，细节处理跟 100C3140 相同
```

### 6. 第四处硬件信息 Hash

第 4 处硬件信息 Hash，`100BEE30` 返回的结果为一固定字符串，没有读取硬件信息，细节处理跟上述的 sha hash 差不多。

### 7. 最终混合 Hash

现在我们再看 `100B9DD0` 的处理，看调用 `100B9DD0` -> `100BCB10` -> `100BCD20`：

```
100BCE14    52              push    edx                 edx 为长度
100BCE15    50              push    eax                 eax 为刚才产生的 4 份硬件指纹数据
100BCE16    8D4C24 64       lea     ecx, dword ptr ss:[esp+64]
100BCE1A    E8 2F9FF9FF     call    10056D4E            sha update
100BCE1F    8D4424 5C       lea     eax, dword ptr ss:[esp+5C]
100BCE23    50              push    eax
100BCE24    8D4C24 30       lea     ecx, dword ptr ss:[esp+30]
100BCE28    E8 F1ACF9FF     call    10057B1E
100BCE2D    8D4C24 2C       lea     ecx, dword ptr ss:[esp+2C]
100BCE31    51              push    ecx
100BCE32    8D4C24 60       lea     ecx, dword ptr ss:[esp+60]
100BCE36    C68424 A0000000 mov     byte ptr ss:[esp+A0], 5
100BCE3E    E8 829EF9FF     call    10056CC5            sha finale，得到的结果为 [ecx+8]+C
```

`100BCE1A` 处要注意，除了刚才 4 份硬件指纹数据外，还有几个固定字符组合在一起，取 `10056CC5` 处 sha hash 后的数据，取从第 10 位开始的 5 个字节，返回到 `10054169`。

### 8. 返回与后续处理

```
10054169    6A 04           push    4                     产品系列号的值的长度
1005416B    8D5424 4C       lea     edx, dword ptr ss:[esp+4C]
1005416F    52              push    edx                     edx 产品系列号的值，此值为固定值
10054170    8D4424 14       lea     eax, dword ptr ss:[esp+14]
10054174    50              push    eax                     保存结果到 eax + C 中
10054175    8D4C24 28       lea     ecx, dword ptr ss:[esp+28]
10054179    C64424 4C 01    mov     byte ptr ss:[esp+4C], 1
1005417E    E8 292C0000     call    10056DAC              sha Hash
10054183    8B7C24 18       mov     edi, dword ptr ss:[esp+18]
10054187    8A0F            mov     cl, byte ptr ds:[edi]   取产品系列号 Sha Hash 后的第一位
10054189    880B            mov     byte ptr ds:[ebx], cl   组成最后的硬件指纹数组 5 字节
```



# 硬件指纹验证流程分析

## 过程 I：硬件指纹生成

```
1005418B    8B4C24 14       mov     ecx, dword ptr ss:[esp+14]
```

至此，整个硬件指纹产生结束，硬件指纹值为 `5` 字节。

## 过程 II：Activation Code 的验证

```
10051EF0    E8 9BFAFFFF     call    10051990                # 验证算法，赋值到 esi+8, esi+C，我称为验证 I, II
10051EF5    385E 08         cmp     byte ptr ds:[esi+8], bl # esi+8 的值要为 1
10051EF8    74 0A           je      short 10051F04
10051EFA    397E 0C         cmp     dword ptr ds:[esi+C], edi # esi+C 值要等于 产品系列号的值
10051EFD    0F94C0          sete    al
10051F00    3AC3            cmp     al, bl
10051F02    75 4C           jnz     short 10051F50
```

`10051F50 -> 10051460`，分析 `10051460` 的主验证过程

### 1. 主验证过程 (`10051460`)

```
10051569    8A56 04         mov     dl, byte ptr ds:[esi+4]
1005156C    8A46 03         mov     al, byte ptr ds:[esi+3]
1005156F    8A4E 02         mov     cl, byte ptr ds:[esi+2]
10051572    889424 B4000000 mov     byte ptr ss:[esp+B4], dl # 逆转硬件指纹数据,并储存到此地址，称 Rstr
10051579    8A56 01         mov     dl, byte ptr ds:[esi+1]
1005157C    888424 B5000000 mov     byte ptr ss:[esp+B5], al
10051583    8A06            mov     al, byte ptr ds:[esi]
100515D7    6A 03           push    3
100515D9    6A 05           push    5
100515DB    8D8C24 BC000000 lea     ecx, dword ptr ss:[esp+BC]
100515E2    51              push    ecx
100515E3    8D5424 6C       lea     edx, dword ptr ss:[esp+6C] # 压入 Rstr
100515E7    52              push    edx
100515E8    8D8C24 98000000 lea     ecx, dword ptr ss:[esp+98]
100515EF    C68424 1C010000 mov     byte ptr ss:[esp+11C], 5
100515F7    E8 80630000     call    1005797C                # 产生 init key, IV str for blowfish
```

在 `1005797C` 处，会产生一组 `8` 字节字符，和一组 `16` 字节字符，主要的产生函数是 `1005759B`。

### 2. Key 和 IV 生成函数 (`1005797C`)

```
100577CF    6A 01           push    1
100577D1    6A 10           push    10                      # 长度
100577D3    53              push    ebx                       # 固定字符串
100577D4    FF75 0C         push    dword ptr ss:[ebp+C]
100577D7    8D45 D4         lea     eax, dword ptr ss:[ebp-2C] # 长度
100577DA    FF75 08         push    dword ptr ss:[ebp+8]    # Rstr
100577DD    50              push    eax
100577DE    E8 B8FDFFFF     call    1005759B                # ShaEX call
```

列出 `1005759B` 的关键处理过程：

```
10057604    8B4D C4         mov     ecx, dword ptr ss:[ebp-3C] # 此处控制循环次数 2 次
10057607    8D0419          lea     eax, dword ptr ds:[ecx+ebx]
1005760A    3945 08         cmp     dword ptr ss:[ebp+8], eax
1005760D    72 0C           jb      short 1005761B
1005760F    8B45 E4         mov     eax, dword ptr ss:[ebp-1C]
10057612    3945 EC         cmp     dword ptr ss:[ebp-14], eax
10057628    FF75 DC         push    dword ptr ss:[ebp-24]    # 字符串长度
1005762B    8BCE            mov     ecx, esi
1005762D    FF75 E0         push    dword ptr ss:[ebp-20]    # 压入第一次 sha 的 Hash 值
10057630    E8 19F7FFFF     call    10056D4E                # sha update
10057635    FF75 10         push    dword ptr ss:[ebp+10]    # 字符串长度
10057638    8BCE            mov     ecx, esi
1005763A    FF75 0C         push    dword ptr ss:[ebp+C]     # 上层传入的字符串
1005763D    C645 F3 01      mov     byte ptr ss:[ebp-D], 1
10057641    E8 08F7FFFF     call    10056D4E                # sha update
10057646    837D 14 00      cmp     dword ptr ss:[ebp+14], 0
1005764A    74 0D           je      short 10057659
1005764C    FF75 18         push    dword ptr ss:[ebp+18]
1005764F    8BCE            mov     ecx, esi
10057651    FF75 14         push    dword ptr ss:[ebp+14]
10057654    E8 F5F6FFFF     call    10056D4E                # sha update
10057659    8BCE            mov     ecx, esi
1005765B    E8 E4F5FFFF     call    10056C44                # sha finalize
1005770B    51              push    ecx       # 压入 16 字节
1005770C    53              push    ebx       # ebx 为 hash 的前 16 字节
1005770D    8BCF            mov     ecx, edi      # 10057E75 将第一次 hash 结果保存到 ecx+14 处
1005770F    E8 61070000     call    10057E75
10057714    FF75 D0         push    dword ptr ss:[ebp-30]  # 压入 8 字节
10057717    8BCF            mov     ecx, edi
10057719    FF75 D4         push    dword ptr ss:[ebp-2C]  # 改处为第一次 sha 的最后 4 位+第二次 sha 的前 4 位
1005771C    E8 B7070000     call    10057ED8    # 10057ED8 将第二次 hash 结果保存到 ecx+20 处
```

至此，`BlowFish` 所需要的 `key`，和 `IV` 数组便产生，回到主验证过程 `10051605`。

### 3. Blowfish 初始化与解密

```
10051605    51              push    ecx               # ecx + c 为 IV 数组 8 字节
10051606    8D9424 8C000000 lea     edx, dword ptr ss:[esp+8C]
1005160D    898424 B8000000 mov     dword ptr ss:[esp+B8], eax
10051614    52              push    edx               # edx + c 为要进行 Blowfish Init 的 key
10051615    8D4C24 68       lea     ecx, dword ptr ss:[esp+68]
10051619    C68424 14010000 mov     byte ptr ss:[esp+114], 6
10051621    888424 C0000000 mov     byte ptr ss:[esp+C0], al
10051628    E8 2E5D0000     call    1005735B          # blowfish Init
```

这里是 `blowfish Init` 处理：

```
1005166A    50              push    eax               # eax+8 为输入的验证码
1005166B    8D4C24 64       lea     ecx, dword ptr ss:[esp+64]
1005166F    C68424 10010000 mov     byte ptr ss:[esp+110], 7 # ecx + 0x20 blowfish 结构体
10051677    E8 D55A0000     call    10057151          # blowfish decrypt
```

`blowfish decrypt` 处理，这里不是调用 `ECB`，`CFB` 模式，具体流程会走到 `1005E280`，列出主要过程：

```
1005E3F7    50              push    eax
1005E3F8    51              push    ecx
1005E3F9    895424 1C       mov     dword ptr ss:[esp+1C], edx
1005E3FD    E8 7EF4FFFF     call    1005D880          # blowfish 的具体 decrypt 函数
1005E447    8A03            mov     al, byte ptr ds:[ebx]       # 注册码
1005E449    8A0C3E          mov     cl, byte ptr ds:[esi+edi]   # IV 数组
1005E44C    88043E          mov     byte ptr ds:[esi+edi], al   # 用注册码值来作为下次加密
1005E44F    32C1            xor     al, cl                # 两个数据 xor
1005E451    8B4C24 20       mov     ecx, dword ptr ss:[esp+20]
1005E455    43              inc     ebx
1005E456    8801            mov     byte ptr ds:[ecx], al   # 结果保存到 ecx 的地址
1005E458    8B4424 24       mov     eax, dword ptr ss:[esp+24]
1005E45C    41              inc     ecx
1005E45D    46              inc     esi
1005E45E    83E6 07         and     esi, 7
1005E461    48              dec     eax                     # 递减加密字符串的长度
```

回到主验证过程 `1005167C`：

```
1005167C    8B4C24 1C       mov     ecx, dword ptr ss:[esp+1C]
10051680    8B01            mov     eax, dword ptr ds:[ecx]
10051682    898424 BC000000 mov     dword ptr ss:[esp+BC], eax
10051689    8B51 04         mov     edx, dword ptr ds:[ecx+4]
1005168C    899424 C0000000 mov     dword ptr ss:[esp+C0], edx
10051693    66:8B49 08      mov     cx, word ptr ds:[ecx+8]
10051697    66:898C24 C4000 mov     word ptr ss:[esp+C4], cx
1005169F    8B4D 0C         mov     ecx, dword ptr ss:[ebp+C]
100516A2    8BBC24 C1000000 mov     edi, dword ptr ss:[esp+C1] # 将 Blowfish decrypt 后的数据，第 5 位起的 4 字节赋值到 edi
100516A9    6A 05           push    5
100516AB    8D55 3C         lea     edx, dword ptr ss:[ebp+3C]
100516AE    8902            mov     dword ptr ds:[edx], eax
100516B0    8A8424 C4000000 mov     al, byte ptr ss:[esp+C4]
100516B7    56              push    esi               # 硬件指纹数组
100516B8    51              push    ecx
100516B9    8BCD            mov     ecx, ebp
100516BB    8842 04         mov     byte ptr ds:[edx+4], al
100516BE    E8 4DD6FFFF     call    1004ED10          # sha(硬件指纹数组+固定值)
```

看 `1004ED10` 的重要处理过程。



```
1004ED6B    8B5424 64       mov     edx, dword ptr ss:[esp+64]  

1004ED6F    51              push    ecx                             长度  

1004ED70    52              push    edx                             edx 为硬件指纹数组  

1004ED71    8D4C24 14       lea     ecx, dword ptr ss:[esp+14]  

1004ED75    C64424 60 01    mov     byte ptr ss:[esp+60], 1  

1004ED7A    E8 CF7F0000     call    10056D4E                        sha update  

1004ED7F    8A4C24 63       mov     cl, byte ptr ss:[esp+63]  

1004ED83    B0 53           mov     al, 53  

1004ED85    884424 44       mov     byte ptr ss:[esp+44], al  

1004ED89    884424 45       mov     byte ptr ss:[esp+45], al  

1004ED8D    8B4424 60       mov     eax, dword ptr ss:[esp+60]  

1004ED91    884424 48       mov     byte ptr ss:[esp+48], al  

1004ED95    8A4424 62       mov     al, byte ptr ss:[esp+62]  

1004ED99    6A 14           push    14                              长度  

1004ED9B    8D5424 3C       lea     edx, dword ptr ss:[esp+3C]  

1004ED9F    886424 4D       mov     byte ptr ss:[esp+4D], ah  

1004EDA3    884424 4E       mov     byte ptr ss:[esp+4E], al  

1004EDA7    52              push    edx                             sha update，这数据此值为固定值  

1004EDA8    8D4424 30       lea     eax, dword ptr ss:[esp+30]  

1004EDAC    884C24 53       mov     byte ptr ss:[esp+53], cl  

1004EDB0    50              push    eax                             返回 sha 后的结果,在 eax+C 处  

1004EDB1    8D4C24 18       lea     ecx, dword ptr ss:[esp+18]  

1004EDF6    C64424 53 02    mov     byte ptr ss:[esp+53], 2  

1004EDFB    E8 AC7F0000     call    10056DAC                        sha update，并且 sha finalize  

返回到 100516C3，此处要求值一致  

100516C3    3BC7            cmp     eax, edi        返回结果到 eax，取头 4 字节  

100516C5    74 03           je      short 100516CA          与 100516A2 处的 edi 对比，即与 Blowfish 加密后的数据对比  

100516C7   |897D 0C         mov     dword ptr ss:[ebp+C], edi      赋值到验证处 2  

到了最后一步验证，将 blowfish crypto 后的 9 个字节数据进行 sha hash，然后与取其第一个值与第 10 个字节对比  

10051707    6A 09           push    9                               长度为 9 字节  

10051709    8D8C24 C0000000 lea     ecx, dword ptr ss:[esp+C0]  

10051710    51              push    ecx                             blowfish decrypto 后的 str 地址  

10051711    8D5424 3C       lea     edx, dword ptr ss:[esp+3C]  

10051715    52              push    edx                             edx+C 储存 sha hash 后的结果  

10051716    8D4C24 50       lea     ecx, dword ptr ss:[esp+50]  

1005171A    C68424 18010000>mov     byte ptr ss:[esp+118], 9  

10051722    E8 85560000     call    10056DAC                        对 ecx 进行 sha hash  

10051727    8B7C24 40       mov     edi, dword ptr ss:[esp+40]  

1005172B    8A0F            mov     cl, byte ptr ds:[edi]           获取 ecx 第 10 的值  

1005172D    8A8424 C5000000 mov     al, byte ptr ss:[esp+C5]       获取 1004ED10 hash 的第一个值  

10051734    3AC1            cmp     al, cl                          对比两个值  

10051736    0F94C1          sete    cl  

10051739    884D 08         mov     byte ptr ss:[ebp+8], cl        赋值到验证处 1  
```

至此，整个 `activation code` 验证完毕。

### 加密思路

1. 读取硬件信息，产生 4 个 `sha` hash 数组;
2. 4 个 `sha` hash 数组加固定字符组合在一起，然后再进行 `sha` hash，取从第 10 个字节开始的 5 个字节;
3. 对产品系列号的值进行 `sha` hash，取第一个字节，与第二步骤产生的 5 个字节合并，产生硬件指纹;
4. 逆转第 3 步产生的字符串，然后调用 `1005797C`，产生 `blowfish` 的 `IV`，`Key` 数组;
5. 先调用 `BlowFish Init`，然后对输入的 `activation code` 进行 `Blowfish` 解密处理，产生数组简称 `Dstr`;
6. 对第 3 步产生的硬件指纹，调用 `1004ED10` 进行 `Sha Hash`，取头 4 字节，取 `Dstr` 第 5 个字节起的 4 个字节，两者进行比较，要求值一致;
7. 取 `Dstr` 数组的头 9 个字节，调用 `sha` hash，hash 后得到的数组取第 1 个字节与 `Dstr` 数组的第 10 个字节，进行比较，要求值一致.

### 解密思路

1. 头 4 个步骤与加密思路相同，先得到硬件指纹，然后模拟 `1005797C` 函数得到 `blowfish` 的 `IV`，`Key` 数组;
2. 模拟解密后的数组，先随机产生 5 个字节数据;
3. 与加密思路步骤 6 相同，模拟 `1004ED10`，取头 4 字节，与步骤 4 产生的数据合并，组成 9 个字节的数组 `A`;
4. 对 `A` 数组，调用 `sha` hash，取 hash 数组第 1 个字节，作为数组 `A` 的第 10 个字节
5. 对 `A` 进行 `Blowfish` 加密处理，还原出 `activation code`.

### 结语

这篇文章，只是列出关键的处理过程，只是向大家说明一个加密思路，细致的跟踪留给感兴趣的朋友们，还有我放出的注册机

只是在 `xp`，`08` 系统测试过，不保证所有系统平台都可以。

### 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢！

`2011` 年 `11` 月 `18` 日 by `Daniel`

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | `CorelDRAW Graphics Suite X5 SP3.v15.2.0.686.rar` |
| :--- | :--- |
