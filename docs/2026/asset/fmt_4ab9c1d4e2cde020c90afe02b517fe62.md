
# [原创]分析用了点数学的五棋软件

- **文章标题**：有点数学的易解软体
- **文章作者**：P\_O\_K\_E\_R
- **作者邮箱**：p\_o\_k\_e\_r@sohu.com
- **软件名称**：Chesses V1.01
- **下载地址**：268K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Z5K9h3&6W2M7$3g2U0K9r3g2U0K9$3g2J5M7#2)9J5k6i4k6W2k6$3q4J5k6o6u0Q4x3X3g2F1L8#2)9J5c8X3y4Z5k6i4y4K6k6i4y4Q4y4h3k6W2P5r3g2U0N6i4c8W2i4K6u0W2k6i4S2W2
- **保护方式**：序列号
- **编写语言**：C++
- **使用工具**：OllyICE
- **操作平台**：WinXP
- **软件介绍**：黑白连珠跳中国际象棋五合一
- **作者声明**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

很快定位于：

```
00417FE0  |.  FF15 0CB34100 call    dword ptr [<&USER32.GetWindowTextA>]
```

往外跳几下就看到：

```
0040305F   .  E8 C5130100   call    00414429
00403064   .  8B47 60       mov     eax, dword ptr [edi+60]
00403067   .  50            push    eax
00403068   .  E8 F3110000   call    00404260
0040306D   .  83C4 04       add     esp, 4
00403070   .  85C0          test    eax, eax
00403072   .  0F84 90000000 je      00403108                 ;爆点
```

注册判断位置很明显：

```
00404260  /$  83EC 50         sub     esp, 50
00404263  |.  83C9 FF         or      ecx, FFFFFFFF
00404266  |.  33C0            xor     eax, eax
00404268  |.  8D5424 00       lea     edx, dword ptr [esp]
0040426C  |.  56              push    esi
0040426D  |.  57              push    edi
0040426E  |.  8B7C24 5C       mov     edi, dword ptr [esp+5C]

  堆栈 ss:[0012C8A0]=003F72F8, (ASCII "12345678")

00404272  |.  F2:AE           repne   scas byte ptr es:[edi]
00404274  |.  F7D1            not     ecx
00404276  |.  2BF9            sub     edi, ecx
00404278  |.  8BC1            mov     eax, ecx
0040427A  |.  8BF7            mov     esi, edi
0040427C  |.  8BFA            mov     edi, edx
0040427E  |.  C1E9 02         shr     ecx, 2
00404281  |.  F3:A5           rep     movs dword ptr es:[edi], dword ptr [esi]
00404283  |.  8BC8            mov     ecx, eax
00404285  |.  33C0            xor     eax, eax
00404287  |.  83E1 03         and     ecx, 3
0040428A  |.  F3:A4           rep     movs byte ptr es:[edi], byte ptr [esi]

  将字符串转到临时位置：003F72F8->0012C84C

0040428C  |.  8D7C24 08       lea     edi, dword ptr [esp+8]
00404290  |.  83C9 FF         or      ecx, FFFFFFFF
00404293  |.  F2:AE           repne   scas byte ptr es:[edi]
00404295  |.  F7D1            not     ecx
00404297  |.  49              dec     ecx
00404298  |.  83F9 08         cmp     ecx, 8
0040429B  |.  74 06           je      short 004042A3

  要求字符串为八字节长

0040429D  |.  5F              pop     edi
0040429E  |.  5E              pop     esi
0040429F  |.  83C4 50         add     esp, 50
004042A2  |.  C3              retn
004042A3  |>  8D4C24 0C       lea     ecx, dword ptr [esp+C]
004042A7  |.  C64424 10 00    mov     byte ptr [esp+10], 0
004042AC  |.  51              push    ecx
004042AD  |.  E8 481D0000     call    00405FFA
```

跟入：

```
00405FFA  /$  FF7424 04       push    dword ptr [esp+4]
00405FFE  |.  E8 6CFFFFFF     call    00405F6F
00406003  |.  59              pop     ecx
00406004  \.  C3              retn
```

压进的是炼码后半边`'5678'`。再入：

```
00405F6F  /$  53              push    ebx
00405F70  |.  55              push    ebp
00405F71  |.  56              push    esi
00405F72  |.  57              push    edi
00405F73  |.  8B7C24 14       mov     edi, dword ptr [esp+14]

  堆栈 ss:[0012C838]=0012C850, (ASCII "5678")

00405F77  |>  833D DC244200 0>/cmp     dword ptr [4224DC], 1

  数据 ds:[004224DC]=1

00405F7E  |.  7E 0F           |jle     short 00405F8F
00405F80  |.  0FB607          |movzx   eax, byte ptr [edi]
00405F83  |.  6A 08           |push    8
00405F85  |.  50              |push    eax
00405F86  |.  E8 0D360000     |call    00409598
00405F8B  |.  59              |pop     ecx
00405F8C  |.  59              |pop     ecx
00405F8D  |.  EB 0F           |jmp     short 00405F9E
00405F8F  |>  0FB607          |movzx   eax, byte ptr [edi]
00405F92  |.  8B0D D0224200   |mov     ecx, dword ptr [4222D0]         ;  chesses.004222DA
00405F98  |.  8A0441          |mov     al, byte ptr [ecx+eax\*2]
```

这是比较有意思的地方，查表转换：

比如第一个：

`ds:[00422344]=84`

`al=35 ('5')`

表格如下：

```
004222DA    0020   0020   0020   0020   0020   0020   0020   0020
004222EA    0020   0028   0028   0028   0028   0028   0020   0020
004222FA    0020   0020   0020   0020   0020   0020   0020   0020
0042230A    0020   0020   0020   0020   0020   0020   0020   0020
0042231A    0048   0010   0010   0010   0010   0010   0010   0010
0042232A    0010   0010   0010   0010   0010   0010   0010   0010
0042233A    0084   0084   0084   0084   0084   0084   0084   0084
0042234A    0084   0084   0010   0010   0010   0010   0010   0010
0042235A    0010   0081   0081   0081   0081   0081   0081   0001
0042236A    0001   0001   0001   0001   0001   0001   0001   0001
0042237A    0001   0001   0001   0001   0001   0001   0001   0001
0042238A    0001   0001   0001   0010   0010   0010   0010   0010
0042239A    0010   0082   0082   0082   0082   0082   0082   0002
004223AA    0002   0002   0002   0002   0002   0002   0002   0002
004223BA    0002   0002   0002   0002   0002   0002   0002   0002
004223CA    0002   0002   0002   0010   0010   0010   0010   0020
004223DA    0000   0000   0000   0000   0000   0000   0000   0000
004223EA    0000   0000   0000   0000   0000   0000   0000   0000
```

表格再下面部分都是零。

```
00405F9B  |.  83E0 08         |and     eax, 8
00405F9E  |>  85C0            |test    eax, eax
00405FA0  |.  74 03           |je      short 00405FA5
00405FA2  |.  47              |inc     edi
00405FA3  |.^ EB D2           \jmp     short 00405F77
00405FA5  |>  0FB637          movzx   esi, byte ptr [edi]
00405FA8  |.  47              inc     edi
00405FA9  |.  83FE 2D         cmp     esi, 2D
00405FAC  |.  8BEE            mov     ebp, esi
00405FAE  |.  74 05           je      short 00405FB5
00405FB0  |.  83FE 2B         cmp     esi, 2B
00405FB3  |.  75 04           jnz     short 00405FB9
00405FB5  |>  0FB637          movzx   esi, byte ptr [edi]
00405FB8  |.  47              inc     edi
00405FB9  |>  33DB            xor     ebx, ebx
00405FBB  |>  833D DC244200 0>/cmp     dword ptr [4224DC], 1
00405FC2  |.  7E 0C           |jle     short 00405FD0
00405FC4  |.  6A 04           |push    4
00405FC6  |.  56              |push    esi
00405FC7  |.  E8 CC350000     |call    00409598
00405FCC  |.  59              |pop     ecx
00405FCD  |.  59              |pop     ecx
00405FCE  |.  EB 0B           |jmp     short 00405FDB
00405FD0  |>  A1 D0224200     |mov     eax, dword ptr [4222D0]
00405FD5  |.  8A0470          |mov     al, byte ptr [eax+esi\*2]
00405FD8  |.  83E0 04         |and     eax, 4
00405FDB  |>  85C0            |test    eax, eax
00405FDD  |.  74 0D           |je      short 00405FEC
00405FDF  |.  8D049B          |lea     eax, dword ptr [ebx+ebx\*4]
00405FE2  |.  8D5C46 D0       |lea     ebx, dword ptr [esi+eax\*2-30]
```

第一次经过寄存器为：

```
EAX 00000000
ECX 004222DA
EDX 0012C84C ASCII "12345678"
EBX 00000000
ESP 0012C824
EBP 00000035
ESI 00000035
EDI 0012C851 ASCII "678"
EIP 00405FE2 chesses.00405FE2
```

上面两句合起来是`ebx=esi+5*2*ebx-30h`......

```
00405FE6  |.  0FB637          |movzx   esi, byte ptr [edi]
00405FE9  |.  47              |inc     edi
00405FEA  |.^ EB CF           \jmp     short 00405FBB
00405FEC  |>  83FD 2D         cmp     ebp, 2D
00405FEF  |.  8BC3            mov     eax, ebx
00405FF1  |.  75 02           jnz     short 00405FF5
00405FF3  |.  F7D8            neg     eax
00405FF5  |>  5F              pop     edi
00405FF6  |.  5E              pop     esi
00405FF7  |.  5D              pop     ebp
00405FF8  |.  5B              pop     ebx
00405FF9  \.  C3              retn
```

虽然很有意思，但一见乘十便不免偷了懒，跳出看一看。

略略一算，就知是把四个数字字符转成十六进制数，结果为`eax=162Eh=5678`。
