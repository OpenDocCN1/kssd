
# Folder Pilot 注册算法分析(SHA1 算法)

> 原文：[`www.pediy.com/kssd/pediy10/63260.html`](https://www.pediy.com/kssd/pediy10/63260.html)

呵呵，现在感觉论坛上对软件实例的注册分析越来越少了，下面分析的这个软件是德国老外写的一个快速进入 Windows 文件夹的工具软件，发出来应该不算违规吧！ 供我们菜鸟学习交流，高手飘过！

## 一、软件信息

1. 软件功能：快速进入 Windows 文件夹
2. 末注册限制：30 天试用期
3. 加壳方式：ASPack 2.12 -> Alexey Solodovnikov
4. 软件版本：1.0
5. 编译语言：Borland Delphi 4.0 - 5.0
6. 加密算法：SHA1
7. 下载地址：`http://www.baxbex.com/downloads.html`

## 二、脱壳

脱壳太简单了，脱壳后，`PeiD` 可查得有 `SHA1` 算法，知道算法类型后，分析起来就简单多了。

核心算法位置：

代码:

```
0045DB80  /$  BA 14000000   mov     edx, 14
0045DB85  |.  B8 BC1A4E00   mov     eax, 004E1ABC
0045DB8A  |>  C700 9979825A /mov     dword ptr [eax], 5A827999
0045DB90  |.  C740 50 A1EBD>|mov     dword ptr [eax+50], 6ED9EBA1
0045DB97  |.  C780 A0000000>|mov     dword ptr [eax+A0], 8F1BBCDC
0045DBA1  |.  C780 F0000000>|mov     dword ptr [eax+F0], CA62C1D6
0045DBAB  |.  83C0 04       |add     eax, 4
0045DBAE  |.  4A            |dec     edx
0045DBAF  |.^ 75 D9         \\jnz     short 0045DB8A
0045DBB1  |.  C705 28194E00>mov     dword ptr [4E1928], 67452301
0045DBBB  |.  C705 2C194E00>mov     dword ptr [4E192C], EFCDAB89
0045DBC5  |.  C705 30194E00>mov     dword ptr [4E1930], 98BADCFE
0045DBCF  |.  C705 34194E00>mov     dword ptr [4E1934], 10325476
0045DBD9  |.  C705 38194E00>mov     dword ptr [4E1938], C3D2E1F0
0045DBE3  \\.  C3            retn
```

## 三、跟踪注册按钮事件处理函数

先用 `DeDe` 找到注册按钮事件地址，然后就很容易找到注册的关键地方

找到按钮事件地址

代码:

```
0045F398   .  55           push    ebp
0045F399   .  8BEC         mov     ebp, esp
```

输入用户名：`abcdef`, 序列号：`123456`, 就可以进行算法跟踪了。

## 四、算法分析

```
0045E5A5  |.  8D4D EE       lea     ecx, dword ptr [ebp-12]    ;  存放 SHA 加密转换后的结果

0045E5A8  |.  BA 28E64500   mov     edx, 0045E628             ;  ASCII "&fp--fftxar"

0045E5AD  |.  E8 6EFCFFFF   call    0045E220                 ;  有 SHA 算法
```

跟进上面的 `CALL`

第 1 次 `SHA` 加密：("&fp--fftxar")

代码:

```
0045E276  |.  50           push    eax                   ;  用存放 SHA1 加密后的结果
0045E277  |.  8BC6         mov     eax, esi              ;  eax="&fp--fftxar"
0045E279  |.  E8 62B1FAFF  call    004093E0              ;  eax=0b (长度？)
0045E27E  |.  8BC8         mov     ecx, eax              ;  ecx=b
0045E280  |.  8BD6         mov     edx, esi              ;  edx="&fp--fftxar"
0045E282  |.  8BC3         mov     eax, ebx              ;  eax=00FBDFB0
0045E284  |.  E8 D3FCFFFF  call    0045DF5C              ;  对 "&fp--fftxar" 进行 SHA 加密

0045E289  |.  8D8C24 99000>lea     ecx, dword ptr [esp+99]
0045E290  |.  8D9424 C4000>lea     edx, dword ptr [esp+C4]
0045E297  |.  8BC3         mov     eax, ebx
0045E299  |.  E8 F6FEFFFF  call    0045E194              ;  [esp+99]得到 SHA 加密的结果
```

；

得到结果("&fp--fftxar") SHA1：

```
0012FEE5  B7 11 2C 57 64 65 CF F2 7A 3C CB 16 7A DE 8E 45

0012FEF5  2B 3E 2F 2F
```

第 2 次 `SHA` 加密：("\*abcdefenglish.")

代码:

```
0045E2B3  |.  8D8424 B0000>lea     eax, dword ptr [esp+B0]
0045E2BA  |.  50           push    eax                   ;  用存放 SHA1 加密后的结果
0045E2BB  |.  8D4424 08    lea     eax, dword ptr [esp+8]     ;  eax="\*abcdefenglish."
0045E2BF  |.  E8 1CB1FAFF  call    004093E0              ;  eax=0F (长度)
0045E2C4  |.  8BC8         mov     ecx, eax              ;  ecx=0f
0045E2C6  |.  8D5424 08    lea     edx, dword ptr [esp+8]     ;  edx="\*abcdefenglish."
0045E2CA  |.  8BC3         mov     eax, ebx
0045E2CC  |.  E8 8BFCFFFF  call    0045DF5C              ;  SHA 加密
；
0045E2D1  |.  8D8C24 85000>lea     ecx, dword ptr [esp+85]
0045E2D8  |.  8D9424 B0000>lea     edx, dword ptr [esp+B0]
0045E2DF  |.  8BC3         mov     eax, ebx
0045E2E1  |.  E8 AEFEFEFF  call    0045E194              ;  [esp+85]取得 SHA 加密的结果
```

得到结果("\*abcdefenglish.") SHA2：

```
0012FED1  4D 55 99 86 A7 89 92 68 EE B0 16 D1 07 1C 68 87

0012FEE1  A8 55 DC 2B
```

两次 `SHA` 结果进行运算转换：`SHA3[n]=BYTE(SHA2[n]+SHA2[n*2]+SHA1[n*2])` `n=0...9`

代码:

```
0045E30A  |> /33DB         /xor    ebx, ebx
0045E30C  |. |8A18         |mov    bl, byte ptr [eax]
0045E30E  |. |0FB62A       |movzx  ebp, byte ptr [edx]
0045E311  |. |03DD         |add    ebx, ebp                   ;  ebx=SHA2[n]+SHA2[n*2]
0045E313  |. |0FB629       |movzx  ebp, byte ptr [ecx]
0045E316  |. |03DD         |add    ebx, ebp                   ;  ebx=SHA2[n]+SHA2[n*2]+SHA1[n*2]
0045E318  |. |81E3 FF000000|and    ebx, 0FF                   ;  ebx=BYTE(SHA2[n]+SHA2[n*2]+SHA1[n*2])
0045E31E  |. |881E         |mov    byte ptr [esi], bl        ;  保存结果: SHA3[n]=BYTE(SHA2[n]+SHA2[n*2]+SHA1[n*2])[n*2]+SHA1[n*2])
0045E320  |. |46           |inc    esi
0045E321  |. |83C1 02      |add    ecx, 2                    ;  加 2
0045E324  |. |83C2 02      |add    edx, 2                    ;  加 2
0045E327  |. |40           |inc    eax                       ;  加 1
0045E328  |. |4F           |dec    edi
0045E329  |.^\\75 DF       \\jnz     short 0045E30A
```

得到结果：`0012FF5A  51 1A A4 E7 0F 6A 13 5E C1 BB`

然后再对 `SN` 进行变换：

代码:

```
0045E5B2  |.  8B45 F8      mov     eax, dword ptr [ebp-8]     ;  "123456"
0045E5B5  |.  E8 E25AFAFF  call    0040409C
0045E5BA  |.  8D55 E4      lea     edx, dword ptr [ebp-1C]
0045E5BD  |.  E8 7EFDFFFF  call    0045E340                  ;  变换 SN
```

跟进上面的 `CALL`

代码:

```
0045E366  |./EB 4A        jmp     short 0045E3B2            ;  ESI=SN 字符串
0045E368  |> |803C3E 58    /cmp    byte ptr [esi+edi], 58    ;  SN[n]
0045E36C  |. |75 04       |jnz     short 0045E372
0045E36E  |. |C6043E 30   |mov     byte ptr [esi+edi], 30    ;  若 SN[n]=58('X')，则 SN[n]='0'
0045E372  |> |803C3E 5A   |cmp    byte ptr [esi+edi], 5A
0045E376  |. |75 04       |jnz     short 0045E37C
0045E378  |. |C6043E 4F   |mov     byte ptr [esi+edi], 4F    ;  若 SN[n]=5A('Z')，则 SN[n]='O'(4F)
0045E37C  |> |8A043E      |mov     al, byte ptr [esi+edi]
0045E37F  |. |3C 2D       |cmp     al, 2D                    ;  Switch (cases 0..39)
0045E381  |. |74 2E       |je      short 0045E3B1            ;  若 SN[n]='-'，则转
0045E383  |. |3C 39       |cmp     al, 39
0045E385  |. |77 07       |ja      short 0045E38E
0045E387  |. |8A043E      |mov     al, byte ptr [esi+edi]
0045E38A  |. |2C 30       |sub     al, 30                    ;  若 SN[n]<=39,则转为对应的数值(0...9)
0045E38C  |. |EB 07       |jmp     short 0045E395
0045E38E  |> |8A043E      |mov     al, byte ptr [esi+edi]    ;  Default case of switch 0045E37F
0045E391  |. |2C 41       |sub     al, 41
0045E393  |. |04 08       |add     al, 8                     ;  否则 al=SN[n]-41+8
0045E395  |> |84DB        |test    bl, bl                    ;  bl 标志位
0045E397  |. |74 08       |je      short 0045E3A1
0045E399  |. |884424 04   |mov     byte ptr [esp+4], al      ;  暂存
0045E39D  |. |33DB        |xor     ebx, ebx                ;  标志位清 0
0045E39F  |. |EB 10       |jmp     short 0045E3B1
0045E3A1  |> |8B1424      |mov     edx, dword ptr [esp]      ;  保存地址
0045E3A4  |. |C1E0 04     |shl     eax, 4                    ;  上一字节处理结果移到 Byte 的高 4 位
0045E3A7  |. |024424 04   |add     al, byte ptr [esp+4]      ;  加上本字节处理结果, 组成一个字节
0045E3AB  |. |88042A      |mov     byte ptr [edx+ebp], al    ;  保存转换结果
0045E3AE  |. |45          |inc     ebp
0045E3AF  |. |B3 01       |mov     bl, 1
0045E3B1  |> |47          |inc     edi                       ;  下一字节; Case 2D of switch 0045E37F
0045E3B2  |> \8BC6        mov     eax, esi
0045E3B4  |. |E8 27B0FAFF |call    004093E0                 ;  得到 SN 字符串长度
0045E3B9  |. |3BF8        |cmp     edi, eax                 ;  是否处理完毕
0045E3BB  |.^ 72 AB       \\jb      short 0045E368
```

如：`511AA4E70F6A135EC1BB`，经变换得到：`15 81 48 7C D0 86 31 C5 1A 99`

**SN 变换总结,**

变换后存放为：`SN_R[10]`

1. `R1=Val(SN[n]),R2=Val(SN[n+1])`，其中 `Val` 的处理方式为：
