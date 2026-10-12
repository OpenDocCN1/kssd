
# Plato Video To iPod Converter V3.46 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-15.htm`](https://www.pediy.com/kssd/pediy09/pediy09-15.htm)
> Plato Video To iPod Converter V3.46 算法分析兼 RSA-64 实战

直接切入正题。

试炼码：

```
111111111111111122222222222222223333333333333333
```

1. `PEiD`：VC6 编写，`Kanal` 插件识别出了 `Miracl` 库，所以用 `IDA` + `CryptoSIG` 导出 `map` 文件；
2. `OD` 载入，`Loadmap` 上一步的 `map` 文件，根据出错提示轻松找到 `0040791C`，跟进这个 call：

```asm
00406920  /$> sub  esp, 300
00406926  |.> or   ecx, FFFFFFFF
00406929  |.> xor  eax, eax
0040692B  |.> push ebx
0040692C  |.> push esi
0040692D  |.> mov  esi, [esp+30C]
00406934  |.> push edi
00406935  |.> mov  edi, esi
00406937  |.> repne scas byte ptr es:[edi]
00406939  |.> not  ecx
0040693B  |.> dec  ecx
0040693C  |.> cmp  ecx, 30 ; 注册码长度：48=3*16
0040693F  |.> jnz  00406A2C
00406945  |.> xor  ebx, ebx
00406947  |.> mov  ecx, 3F
0040694C  |.> lea  edi, [esp+10D]
00406953  |.> mov  [esp+10C], bl
0040695A  |.> rep  stos dword ptr es:[edi]
0040695C  |.> stos word ptr es:[edi]
0040695E  |.> stos byte ptr es:[edi]
0040695F  |.> mov  ecx, 3F
00406964  |.> xor  eax, eax
00406966  |.> lea  edi, [esp+D]
0040696A  |.> mov  [esp+C], bl
0040696E  |.> rep  stos dword ptr es:[edi]
00406970  |.> stos word ptr es:[edi]
00406972  |.> stos byte ptr es:[edi]
00406973  |.> mov  ecx, 3F
00406978  |.> xor  eax, eax
0040697A  |.> lea  edi, [esp+20D]
00406981  |.> mov  [esp+20C], bl
00406988  |.> rep  stos dword ptr es:[edi]
0040698A  |.> stos word ptr es:[edi]
0040698C  |.> stos byte ptr es:[edi]
0040698D  |.> mov  edi, [<&MSVCRT.strncpy>] ; MSVCRT.strncpy
00406993  |.> push 10 ; /maxlen = 10 (16.)
00406995  |.> lea  eax, [esp+110] ; |
0040699C  |.> push esi ; |src
0040699D  |.> push eax ; |sn_1:取 sn 前 16 位
0040699E  |.> call edi ; \strncpy
004069A0  |.> lea  ecx, [esi+10]
004069A3  |.> push 10
004069A5  |.> lea  edx, [esp+1C]
004069A9  |.> push ecx
004069AA  |.> push edx ; sn_2:sn 中间 16 位
004069AB  |.> call edi
004069AD  |.> add  esi, 20
004069B0  |.> push 10
004069B2  |.> lea  eax, [esp+228]
004069B9  |.> push esi
004069BA  |.> push eax ; sn_3:sn 最后 16 位
004069BB  |.> call edi
004069BD  |.> lea  ecx, [esp+130]
004069C4  |.> push 0041B570 ; ASCII "C09EE4DD93F3D6D1"
004069C9  |.> push ecx ; sn_1
004069CA  |.> mov  [esp+148], bl
004069D1  |.> mov  [esp+48], bl
004069D5  |.> mov  [esp+248], bl
004069DC  |.> call <mir> ; 计算出 c1
```

`mir` 标签是我自己加的，跟进去可以看见 `miracl` 库函数。

```asm
{
004067F0  >/$> sub  esp, 108 ; 因为这个函数调用了 3 次，所以我们用了分号和 sn_x(表示 sn_1,sn_2,sn_3)
004067F6  |.> push ebx
004067F7  |.> push ebp
004067F8  |.> push esi
004067F9  |.> push edi
004067FA  |.> push 0 ; 0
004067FC  |.> push 64 ; 0x64
004067FE  |.> call <mirsys>
00406803  |.> push 0
00406805  |.> mov  dword ptr [eax+234], 10 ; 16 进制
0040680F  |.> call <_mirvar>
00406814  |.> push 0
00406816  |.> mov  ebx, eax
00406818  |.> call <_mirvar>
0040681D  |.> push 0
0040681F  |.> mov  [esp+24], eax
00406823  |.> call <_mirvar>
00406828  |.> push 0
0040682A  |.> mov  ebp, eax
0040682C  |.> call <_mirvar>
00406831  |.> mov  [esp+2C], eax
00406835  |.> mov  ecx, 3F
0040683A  |.> xor  eax, eax
0040683C  |.> lea  edi, [esp+31]
00406840  |.> mov  byte ptr [esp+30], 0
00406845  |.> add  esp, 18
00406848  |.> rep  stos dword ptr es:[edi]
0040684A  |.> stos word ptr es:[edi]
0040684C  |.> stos byte ptr es:[edi]
0040684D  |.> mov  edi, [esp+11C]
00406854  |.> or   ecx, FFFFFFFF
00406857  |.> xor  eax, eax
00406859  |.> xor  esi, esi
0040685B  |.> repne scas byte ptr es:[edi]
0040685D  |.> not  ecx
0040685F  |.> dec  ecx
00406860  |.> mov  edi, ecx
00406862  |.> test edi, edi
00406864  |.> jle  short 0040688A
00406866  |> / mov  eax, [esp+11C]
0040686D  |. | movsx ecx, byte ptr [esi+eax]
00406871  |. | push ecx ; /c
00406872  |. | call [<&MSVCRT.isxdigit>] ; \isxdigit
00406878  |. | add  esp, 4
0040687B  |. | test eax, eax
0040687D  |. | je   00406905
00406883  |. | inc  esi
00406884  |. | cmp  esi, edi
00406886  |. \ jl   short 00406866
00406888  |.  test edi, edi
0040688A  |> je   short 00406905
0040688C  |.  mov  edx, [esp+11C]
00406893  |.  push edx ; sn_1;sn_2;sn_3
00406894  |.  push ebx
00406895  |.  call <_cinstr>
0040689A  |.  mov  eax, [esp+128]
004068A1  |.  push eax ; n1;n1;n2
004068A2  |.  push ebp ; n
004068A3  |.  call <_cinstr>
004068A8  |.  mov  edi, [esp+24]
004068AC  |.  push 0041B554 ; ASCII "10001"
004068B1  |.  push edi ; e
004068B2  |.  call <_cinstr>
004068B7  |.  push ebp ; n
004068B8  |.  push ebx ; sn_1;sn_2;sn_3
004068B9  |.  call <_compare> ; sn_x < n
004068BE  |.  add  esp, 20
004068C1  |.  cmp  eax, -1
004068C4  |.  jnz  short 00406905
004068C6  |.  mov  esi, [esp+10]
004068CA  |.  push esi ; c
004068CB  |.  push ebp ; n
004068CC  |.  push edi ; e
004068CD  |.  push ebx ; sn_x
004068CE  |.  call <_powmod> ; c = sn_x^e (mod n)
004068D3  |.  lea  ecx, [esp+28]
004068D7  |.  push 0
004068D9  |.  push ecx
004068DA  |.  push esi ; c
004068DB  |.  push 100
004068E0  |.  call <_big_to_bytes>
004068E5  |.  push ebx
004068E6  |.  call <_mirkill>
004068EB  |.  push esi
004068EC  |.  call <_mirkill>
004068F1  |.  push ebp
004068F2  |.  call <_mirkill>
004068F7  |.  push edi
004068F8  |.  call <_mirkill>
004068FD  |.  add  esp, 30
00406900  |.  call <_mirexit>
00406905  |> lea  edx, [esp+18] ; 深刻理解下面这个函数很重要^_^，请查阅 MSDN 这个函数下面的例子。
00406909  |.  push edx ; /s
0040690A  |.  call [<&MSVCRT.atoi>] ; \atoi
00406910  |.  add  esp, 4
00406913  |.  pop  edi
00406914  |.  pop  esi
00406915  |.  pop  ebp
00406916  |.  pop  ebx
00406917  |.  add  esp, 108
0040691D  \.  ret
}
```

上面的 RSA 清晰明了。

```asm
004069E1  |.> lea  edx, [esp+38]
004069E5  |.> push 0041B570 ; ASCII "C09EE4DD93F3D6D1"
004069EA  |.> push edx ; sn_2
004069EB  |.> mov  esi, eax
004069ED  |.> call <mir> ; 计算出 c2
004069F2  |.> mov  edi, eax
004069F4  |.> lea  eax, [esp+240]
004069FB  |.> push 0041B55C ; ASCII "9C70A3F13DE1C2F1"
00406A00  |.> push eax ; sn_3
00406A01  |.> call <mir> ; 计算出 c3
00406A06  |.> add  esp, 3C
00406A09  |.> cmp  esi, ebx
00406A0B  |> jle  short 00406A2A
00406A0D  |.> cmp  edi, ebx
00406A0F  |.> jle  short 00406A2A
00406A11  |.> cmp  eax, ebx
00406A13  |.> jle  short 00406A2A
00406A15  |.> add  edi, esi ; c1 + c2
00406A17  |.> xor  ecx, ecx
00406A19  |.> cmp  edi, eax ; 比较 c1 + c2 和 c3 是否相等，相等则成功
00406A1B  |.> pop  edi
00406A1C  |.> sete cl
00406A1F  |.> pop  esi
00406A20  |.> mov  eax, ecx
00406A22  |.> pop  ebx
00406A23  |.> add  esp, 300
00406A29  |.> ret
00406A2A  |> xor  eax, eax
00406A2C  |> pop  edi
00406A2D  |.> pop  esi
00406A2E  |.> pop  ebx
00406A2F  |.> add  esp, 300
00406A35  \.> ret
```

整理一下程序验证的思路：

(1) 使用 `n1` 和 `sn_1` 计算出 `c1`，并转换结果为十进制（`atoi` 函数）；
(2) 使用 `n1` 和 `sn_2` 计算出 `c2`，并转换结果为十进制；
(3) 使用 `n2` 和 `sn_3` 计算出 `c3`，并转换结果为十进制；
(4) 判断 `c1 + c2` 和 `c3` 相等否，相等则成功，反之则失败。

经 `RSATool` 计算：

```
=====================
n1=p1 * q1:
C09EE4DD93F3D6D1
p1:
BCAAC583
q1:
1055D561B
d1:
325A57214FA2FBF1

n2=p2 * q2:
9C70A3F13DE1C2F1
p2:
FEED9D0B
q2:
9D190573
d2:
4E31E75F8231BB79
=====================
```

好了，剩下就是写注册机了，下面是注册机的源代码和几组可用的注册码：

```c
//KeyGen for [Plato Video To iPod Converter V3.46]
//Cracked by [HappyTown]

/* RSA inside */
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <time.h>
```
