
# [原创]某著名绘图软件 XXXX Graphics Suite X5 算法分析 + 注册机

## 文章信息
【文章标题】: XXXX Graphics Suite X5 算法分析（一） Serial code
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

## 讨论说明

本文只讨论核心验证算法,由于涉及到函数处理较多，我只会将最重要的处理过程作说明，其他不作说明。

列出`md5`，`blowfish`算法只列出调用地方，算法的运作也不作说明。

### 验证入口

```
0248344F  |.  51            push    ecx
02483450  |.  8D0D 62344802 lea     ecx, dword ptr ds:[2483462]
02483456  |.  83C1 01       add     ecx, 1
02483459  |.  51            push    ecx      [ecx+4]  Serial码
0248345A  |.  E8 D1600100   call    02499530
```

进入`02499530`来到主验证过程，列出主要处理，

```
024834A4   .  FF75 0C       push    dword ptr ss:[ebp+C]                      ; /maxlen
024834A7   .  8B45 90       mov     eax, dword ptr ss:[ebp-70]                ; |
024834AA   .  FF70 04       push    dword ptr ds:[eax+4]                      ; |s2
024834AD   .  FF75 08       push    dword ptr ss:[ebp+8]                      ; |s1          验证是否有DR15
024834B0   .  FF15 58F56502 call    dword ptr ds:[<&MSVCR90.\_strnicmp>]       ; \\_strnicmp
024834F1   .  50            push    eax
024834F2   .  8B4D 90       mov     ecx, dword ptr ss:[ebp-70]
024834F5   .  E8 74FEFFFF   call    0248336E                         验证输入Serial code前7个字符是否符合条件
02483516   .  50            push    eax
02483517   .  8B4D 90       mov     ecx, dword ptr ss:[ebp-70]
0248351A   .  E8 FAFDFFFF   call    02483319                         验证输入Serial code后21个字符是否符合条件
0248354F   .  FF75 EC       push    dword ptr ss:[ebp-14]            长度
02483552   .  8B45 90       mov     eax, dword ptr ss:[ebp-70]
02483555   .  FF70 04       push    dword ptr ds:[eax+4]             [eax+4] 字符串
02483558   .  8B4D A4       mov     ecx, dword ptr ss:[ebp-5C]    转换结果[ ecx + 4 ]
0248355B   .  E8 74F40000   call    024929D4                            将Serial code后21个字符转换为15个
```

进入`024929D4`

```
02492A16  |.  8B5D 08       mov     ebx, dword ptr ss:[ebp+8]
02492A19  |.  6A 07         push    7
02492A1B  |.  53            push    ebx
02492A1C  |.  57            push    edi
02492A1D  |.  FF76 04       push    dword ptr ds:[esi+4]              先复制前7个字节
02492A20  |.  FF15 14F76502 call    dword ptr ds:[<&MSVCR90.memcpy\_s>]        ;  MSVCR90.memcpy\_s
02492A26  |.  83C4 20       add     esp, 20
02492A29  |.  8D47 FA       lea     eax, dword ptr ds:[edi-6]
02492A2C  |.  8945 FC       mov     dword ptr ss:[ebp-4], eax
02492A2F  |.  8D45 FC       lea     eax, dword ptr ss:[ebp-4]
02492A32  |.  50            push    eax
02492A33  |.  8B46 04       mov     eax, dword ptr ds:[esi+4]
02492A36  |.  83C0 07       add     eax, 7
02492A39  |.  50            push    eax                                      eax转换后的数组地址B
02492A3A  |.  8B45 0C       mov     eax, dword ptr ss:[ebp+C]
02492A3D  |.  83C0 F9       add     eax, -7
02492A40  |.  50            push    eax
02492A41  |.  83C3 07       add     ebx, 7                   长度
02492A44  |.  53            push    ebx                                      ebx要转换的数组地址A
02492A45  |.  E8 9F100000   call    02493AE9                          将21个字符转换为15个字符
02492A4A  |.  33C0          xor     eax, eax
02492A4C  |.  3846 08       cmp     byte ptr ds:[esi+8], al
02492A4F  |.  5B            pop     ebx
02492A50  |.  0F95C0        setne   al
02492A53  |.  8D0485 140000>lea     eax, dword ptr ds:[eax*4+14]
02492A5A  |.  8946 0C       mov     dword ptr ds:[esi+C], eax
02492A5D  |.  3BF8          cmp     edi, eax
02492A5F  |.  76 1A         jbe     short 02492A7B
02492A61  |.  8D4F FF       lea     ecx, dword ptr ds:[edi-1]
02492A64  |.  3BC8          cmp     ecx, eax
02492A66  |.  72 13         jb      short 02492A7B
02492A68  |.  8B56 04       mov     edx, dword ptr ds:[esi+4]           转换后的结果要求第15个字符为0，算法第一处验证
02492A6B  |>  803C0A 00     /cmp     byte ptr ds:[edx+ecx], 0
02492A6F  |.  75 07         |jnz     short 02492A78
02492A71  |.  49            |dec     ecx
02492A72  |.  3BC8          |cmp     ecx, eax
02492A74  |.^ 73 F5         \jnb     short 02492A6B
02492A76  |.  EB 03         jmp     short 02492A7B
02492A78  |>  897E 0C       mov     dword ptr ds:[esi+C], edi          此值要求为0
```

看`02493AE9`的处理过程，主要是移位操作，从要转换的数组A中，获取的每个值先转换为5位数值，然后填充转换后的数组B中，

先填充每个字节的高位，接下来的值填充低位，每个字节填充完毕后，才增加索引，如此循环，直到用完要转换的数组A的21个字符。

```
02493AE9  /$  55            push    ebp
02493AEA  |.  8BEC          mov     ebp, esp
02493AEC  |.  51            push    ecx
02493AED  |.  8B45 0C       mov     eax, dword ptr ss:[ebp+C]
02493AF0  |.  53            push    ebx
02493AF1  |.  6BC0 05       imul    eax, eax, 5                   与5相乘
02493AF4  |.  56            push    esi
02493AF5  |.  57            push    edi
02493AF6  |.  8B7D 10       mov     edi, dword ptr ss:[ebp+10]
02493AF9  |.  6A 07         push    7
02493AFB  |.  5B            pop     ebx
02493AFC  |.  03C3          add     eax, ebx
02493AFE  |.  33F6          xor     esi, esi
02493B00  |.  C1E8 03       shr     eax, 3                           除以8，eax为转换数组的长度 = 0xE
02493B03  |.  C645 FF 01    mov     byte ptr ss:[ebp-1], 1
02493B07  |.  3BFE          cmp     edi, esi                         edi为要转换数组的地址，简称B
02493B09  |.  0F84 D9000000 je      02493BE8
02493B0F  |.  8B4D 14       mov     ecx, dword ptr ss:[ebp+14]
02493B12  |.  8B09          mov     ecx, dword ptr ds:[ecx]
02493B14  |.  3BC1          cmp     eax, ecx
02493B16  |.  76 09         jbe     short 02493B21
02493B18  |.  C645 FF 00    mov     byte ptr ss:[ebp-1], 0
02493B1C  |.  E9 C7000000   jmp     02493BE8
02493B21  |>  51            push    ecx                                       ; /n
02493B22  |.  56            push    esi                                       ; |c
02493B23  |.  57            push    edi                                       ; |s
02493B24  |.  E8 853C0000   call    <jmp.&MSVCR90.memset>                     ; \memset
02493B29  |.  33D2          xor     edx, edx
02493B2B  |.  83C4 0C       add     esp, 0C
02493B2E  |.  8975 10       mov     dword ptr ss:[ebp+10], esi
02493B31  |.  3955 0C       cmp     dword ptr ss:[ebp+C], edx
02493B34  |.  0F86 A2000000 jbe     02493BDC
02493B3A  |>  8B45 08       /mov     eax, dword ptr ss:[ebp+8]      eax转换数组开始地址称为A
02493B3D  |.  8B4D 10       |mov     ecx, dword ptr ss:[ebp+10]      ecx索引
02493B40  |.  8A0C01        |mov     cl, byte ptr ds:[ecx+eax]           cl获取数组的值
02493B43  |.  8AC1          |mov     al, cl
02493B45  |.  2C 32         |sub     al, 32                              以下是判断该值属于哪个范围，然后减去相应的值
02493B47  |.  3AC3          |cmp     al, bl                              经过观察，相减后，范围为0-0x1f
02493B49  |.  77 05         |ja      short 02493B50
02493B4B  |.  80E9 1A       |sub     cl, 1A
02493B4E  |.  EB 42         |jmp     short 02493B92
02493B50  |>  8AC1          |mov     al, cl
02493B52  |.  2C 41         |sub     al, 41
02493B54  |.  3AC3          |cmp     al, bl
02493B56  |.  76 3C         |jbe     short 02493B94
02493B58  |.  8AC1          |mov     al, cl
02493B5A  |.  2C 4A         |sub     al, 4A                                   ;  分支 (案例 4A..A4)
02493B5C  |.  3C 04         |cmp     al, 4
02493B5E  |.  77 05         |ja      short 02493B65
02493B60  |.  80E9 42       |sub     cl, 42                                   ;  案例 4A,4B,4C,4D,4E --> 分支 02493B5A
02493B63  |.  EB 2D         |jmp     short 02493B92
02493B65  |>  8AC1          |mov     al, cl
02493B67  |.  2C 50         |sub     al, 50
02493B69  |.  3C 0A         |cmp     al, 0A
02493B6B  |.  77 05         |ja      short 02493B72
02493B6D  |.  80E9 43       |sub     cl, 43                                   ;  案例 9A,9B,9C,9D,9E,9F,A0,A1,A2,A3,A4 --> 分支 02493B5A
02493B70  |.  EB 20         |jmp     short 02493B92
02493B72  |>  8AC1          |mov     al, cl                                   ;  分支 02493B5A 默认案例
```
