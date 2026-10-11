
## 分析思路
在 `feiq` 中，由于等级设置窗口有计算机标识码字符串，我猜想授权码是通过它来生成的，理想情况下，那个最深层子函数就是对计算机标识码字符串直接作一些例如加减乘除与或异或移位等运算，然后生成授权码。可惜，事实没有那么简单，授权码的计算并非“一步”完成。

因此，我的主要思路是：回溯，授权码可能由中间量 `A` 生成，通过分析算法找出 `A`，而 `A` 可能由中间量 `B` 和 `C` 生成，`B` 可能由中间量 `D`、`E` 等和一些已知量(例如计算机标识码，程序里的常量等)生成，这样一步一步追根溯源，逆流而上，最终找出所有已知量，综合当中由已知量生成未知量的算法就得到生成授权码的算法。

## 逆向分析过程
用 `OD` 载入 `Feiq`，进入之前爆破所在的函数 `00463480`，找到之前分析确认是传入计算机标识码生成授权码的函数 `004938D0`，通过边搜索内存边步进，在 `call 00568C0E` 之后出现了授权码，如果忽略大小写，在 `call 00493B60` 之后会出现大小写混杂的“授权码”。用 `IDA` 分析，`00568C0E` 正是 `CStrng::MackUpper`。

```
//函数 004938D0(GenLv1\_MacID\_RegCode)
004938D0  /$  6A FF         push    -1
004938D2  |.  68 299E5800   push    00589E29               ;  SE 处理程序安装
004938D7  |.  64:A1 0000000>mov     eax, dword ptr fs:\[0\]
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
0049391E  |.  8D4424 24     lea     eax, dword ptr \[esp+24\]
00493922  |.  50            push    eax
00493923  |.  E8 38020000   call    00493B60               ;  JASS: 生成字串 JOLG...
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
00493954  |.  55            push    ebp
00493955  |.  8BD1          mov     edx, ecx
00493957  |.  8BF7          mov     esi, edi
00493959  |.  8BFB          mov     edi, ebx
0049395B  |.  53            push    ebx
0049395C  |.  C1E9 02       shr     ecx, 2
0049395F  |.  F3:A5         rep     movs dword ptr es:\[edi\], dword p>
00493961  |.  8BCA          mov     ecx, edx
00493963  |.  83E1 03       and     ecx, 3
00493966  |.  F3:A4         rep     movs byte ptr es:\[edi\], byte ptr>
00493968  |.  8D4C24 30     lea     ecx, dword ptr \[esp+30\]
0049396C  |.  E8 1F11F7FF   call    00404A90               ;  JASS: 生成 4055c0 用到的参考数组 S
00493971  |.  8B8424 440800>mov     eax, dword ptr \[esp+844\]
00493978  |.  C68424 380800>mov     byte ptr \[esp+838\], 3
00493980  |.  8B68 F8       mov     ebp, dword ptr \[eax-8\]
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
00493A01  |.  6A 22         push    22
00493A03  |.  8D8424 3C0400>lea     eax, dword ptr \[esp+43C\]
00493A0A  |.  55            push    ebp
00493A0B  |.  8D4C24 40     lea     ecx, dword ptr \[esp+40\]
00493A0F  |.  50            push    eax
00493A10  |.  51            push    ecx
00493A11  |.  8D4C24 40     lea     ecx, dword ptr \[esp+40\]
00493A15  |.  E8 A61BF7FF   call    004055C0               ;  JASS: 传入机器标识码，生成产生数字串"xxx"的二进制串的函数
00493A1A  |.  8B15 889F5F00 mov     edx, dword ptr \[5F9F88\]     ;  飞秋 FeiQ.005F9F9C
00493A20  |.  8BF8          mov     edi, eax
00493A22  |.  895424 18     mov     dword ptr \[esp+18\], edx
00493A26  |.  33F6          xor     esi, esi
00493A28  |.  C68424 380800>mov     byte ptr \[esp+838\], 4
00493A30  |.  85FF          test    edi, edi
00493A32  |.  7E 51         jle     short 00493A85
00493A34  |>  A1 889F5F00   /mov     eax, dword ptr \[5F9F88\]
00493A39  |.  894424 10     |mov     dword ptr \[esp+10\], eax
00493A3D  |.  33C9          |xor     ecx, ecx
00493A3F  |.  8D5424 10     |lea     edx, dword ptr \[esp+10\]
00493A43  |.  8A8C34 300400>|mov     cl, byte ptr \[esp+esi+430\]
00493A4A  |.  C68424 380800>|mov     byte ptr \[esp+838\], 5
00493A52  |.  51            |push    ecx
00493A53  |.  68 88FD5E00   |push    005EFD88            ;  ASCII "%u"
00493A58  |.  52            |push    edx
00493A59  |.  E8 56040D00   |call    00563EB4            ;  JASS: 类似 sprintf 函数
00493A5E  |.  83C4 0C       |add     esp, 0C
00493A61  |.  8D4424 10     |lea     eax, dword ptr \[esp+10\]
00493A65  |.  8D4C24 18     |lea     ecx, dword ptr \[esp+18\]
00493A69  |.  50            |push    eax
00493A6A  |.  E8 B5500D00   |call    00568B24            ;  JASS: 类似 CString::operator+=
00493A6F  |.  8D4C24 10     |lea     ecx, dword ptr \[esp+10\]
00493A73  |.  C68424 380800>|mov     byte ptr \[esp+838\], 4
00493A7B  |.  E8 8C4C0D00   |call    0056870C
00493A80  |.  46            |inc     esi
00493A81  |.  3BF7          |cmp     esi, edi
00493A83  |.^ 7C AF         \jl      short 00493A34
00493A85  |>  8B8C24 480800>mov     ecx, dword ptr \[esp+848\]      ;  JASS:数字串"xxx"已生成
00493A8C  |.  8D5424 18     lea     edx, dword ptr \[esp+18\]
00493A90  |.  51            push    ecx
00493A91  |.  51            push    ecx
00493A92  |.  8BCC          mov     ecx, esp
00493A94  |.  896424 18     mov     dword ptr \[esp+18\], esp
00493A98  |.  52            push    edx
00493A99  |.  E8 E3490D00   call    00568481
00493A9E  |.  8D4424 1C     lea     eax, dword ptr \[esp+1C\]
00493AA2  |.  50            push    eax
00493AA3  |.  E8 B8000000   call    00493B60               ;  JASS:生成 lmto.....
00493AA8  |.  53            push    ebx
00493AA9  |.  C68424 480800>mov     byte ptr \[esp+848\], 6
00493AB1  |.  E8 76400D00   call    00567B2C
00493AB6  |.  8B4C24 30     mov     ecx, dword ptr \[esp+30\]
00493ABA  |.  51            push    ecx
00493ABB  |.  E8 6C400D00   call    00567B2C
00493AC0  |.  83C4 14       add     esp, 14
00493AC3  |.  8D4C24 14     lea     ecx, dword ptr \[esp+14\]
00493AC7  |.  E8 42510D00   call    00568C0E               ;  JASS:CString::MakeUpper 生成 LMTO.....
00493ACC  |.  8BB424 400800>mov     esi, dword ptr \[esp+840\]
00493AD3  |.  8D5424 14     lea     edx, dword ptr \[esp+14\]
00493AD7  |.  52            push    edx
00493AD8  |.  8BCE          mov     ecx, esi
00493ADA  |.  E8 A2490D00   call    00568481
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
00493B59  \\.  C3            retn

```

进入函数 `00493B60`，会看到有 5 个循环。第一个循环里充满令人发昏的浮点运算；第二、三、四个循环很相似，而且分别循环 5 次；第五个循环里有和数字、大小写字母比较的判断。跟踪分析得，一串很长的数字 `ASCII` 码串经过第一循环得到了三个 32 位数，第二、三、四循环分别用这三个数生成了一个 15byte 的数组，这个数组经过第五循环的变换得到了 15 个字符的大小写混杂的“授权码”。
