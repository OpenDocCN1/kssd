
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
