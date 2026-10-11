
# mirc v5.9

> 原文：[`www.pediy.com/kssd/pediy03/forum1113.htm`](https://www.pediy.com/kssd/pediy03/forum1113.htm)

前言：经常在论坛上游荡，不知不觉也学到了一点东西。现在放假了，也就有时间亲自动手破解。现贴上一个请各位大侠指点指点。

作者：`nanmu`

软件名称：`mirc v5.9`

软件功能：

`IRC` 是英文“`Internet Relay Chat`”的缩写，它是一种即时交谈工具，是目前全球最流行的聊天方式。它的特点是速度快，方便的自建和使用个人聊天室，同时开 `20` 多个小窗与人聊天不影响速度，强大的聊天室管理功能，是全球网友的网上聊天聚会的最佳选择。

软件限制：`30 day trial period`

下载地址：`http://202.102.245.82/soft/soft1/mirc59t.exe`

使用工具：`W32DASM`，`trw2000`

`mirc v5.9` 的破解过程：

1. 启动软件，点击“`帮助`”菜单中“`注册`”，任意输入注册名，注册码：弹出窗口："`The registration name and number you have entered do not match`"

2. 反汇编软件，查找该字符串，来到：

```
...

:004AD06B E817EAF6FF               call 0041BA87

:004AD070 50                       push eax

:004AD071 6A00                     push 00000000

\* Possible Reference to String Resource ID=01913: "The registration name and number you have entered do not mat"

                                      |

:004AD073 6879070000               push 00000779   <------停在这里.
```

3. 向上查找，经过分析，来到

```
.....

\* Referenced by a CALL at Addresses:

|:004ACBAB  , :004ACC1D

|

:004ACA56 55                       push ebp
:004ACA57 8BEC                     mov ebp, esp
:004ACA59 83C4F4                   add esp, FFFFFFF4
:004ACA5C 53                       push ebx           <------输入的注册名
:004ACA5D 56                       push esi           <------输入的注册码
:004ACA5E 57                       push edi
:004ACA5F 8B750C                   mov esi, dword ptr [ebp+0C]
:004ACA62 FF7508                   push [ebp+08]
:004ACA65 E8CE750800               call 00534038      <------计算注册名长度
:004ACA6A 59                       pop ecx
:004ACA6B 83F805                   cmp eax, 00000005  <------注册名长度是否大于 5
:004ACA6E 7307                     jnb 004ACA77       <------大于 5 就跳，不跳死!
:004ACA70 33C0                     xor eax, eax
:004ACA72 E9C9000000               jmp 004ACB40

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACA6E(C)

|

:004ACA77 6A2D                     push 0000002D      <------压入"-"(连接符)
:004ACA79 56                       push esi           <------压入输入的注册码
:004ACA7A E819750800              call 00533F98      <------判断输入的注册码中"-"的位置，赋给 eax
:004ACA7F 83C408                   add esp, 00000008
:004ACA82 8BD8                     mov ebx, eax
:004ACA84 85DB                     test ebx, ebx      <------输入的注册码中是否有"-"，没有 ebx=0
:004ACA86 7507                     jne 004ACA8F       <------不跳就死!
:004ACA88 33C0                     xor eax, eax
:004ACA8A E9B1000000               jmp 004ACB40

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACA86(C)

|

:004ACA8F C60300                   mov byte ptr [ebx], 00
:004ACA92 56                       push esi
:004ACA93 E8A40D0900               call 0053D83C      <------计算输入注册码的前半部分，将其十六进制值
                                                       赋给 eax
:004ACA98 59                       pop ecx
:004ACA99 8945FC                   mov dword ptr [ebp-04], eax  <------将输入注册码的前半部分赋给[ebp-04]
:004ACA9C C6032D                   mov byte ptr [ebx], 2D
:004ACA9F 43                       inc ebx
:004ACAA0 803B00                   cmp byte ptr [ebx], 00
:004ACAA3 7507                     jne 004ACAAC
:004ACAA5 33C0                     xor eax, eax
:004ACAA7 E994000000               jmp 004ACB40

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACAA3(C)

|

:004ACAAC 53                       push ebx
:004ACAAD E88A0D0900               call 0053D83C        <------计算输入注册码的后半部分，将其十六进制值
                                                           赋给 eax
:004ACAB2 59                       pop ecx
:004ACAB3 8945F8                   mov dword ptr [ebp-08], eax
:004ACAB6 FF7508                   push [ebp+08]
:004ACAB9 E87A750800               call 00534038
:004ACABE 59                       pop ecx
:004ACABF 8945F4                   mov dword ptr [ebp-0C], eax
:004ACAC2 33C0                     xor eax, eax
:004ACAC4 33DB                     xor ebx, ebx
:004ACAC6 BA03000000               mov edx, 00000003
:004ACACB 8B4D08                   mov ecx, dword ptr [ebp+08]
:004ACACE 83C103                   add ecx, 00000003          <---定位于输入注册名的第 3 个字符
:004ACAD1 3B55F4                   cmp edx, dword ptr [ebp-0C]
:004ACAD4 7D1C                     jge 004ACAF2

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACAF0(C)

|

:004ACAD6 0FB631                   movzx esi, byte ptr [ecx]  <---[ecx] 中的第一个字符的十六进制值
                                                      赋给 esi
:004ACAD9 0FAF3485B8035500         imul esi, dword ptr [4*eax+005503B8]<---将相乘结果赋给 esi
                                                  005503B8 中的值见最后.
:004ACAE1 03DE                     add ebx, esi              <---将相加结果赋给 ebx
:004ACAE3 40                       inc eax                   <---将 eax 加 1
:004ACAE4 83F826                   cmp eax, 00000026
:004ACAE7 7E02                     jle 004ACAEB               <---你的注册名不会大于 29 位吧!!!
:004ACAE9 33C0                     xor eax, eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACAE7(C)

|

:004ACAEB 42                       inc edx
:004ACAEC 41                       inc ecx
:004ACAED 3B55F4                   cmp edx, dword ptr [ebp-0C]  <---和注册名长度比较
:004ACAF0 7CE4                     jl 004ACAD6                  <---小于，继续计算

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACAD4(C)

|

:004ACAF2 3B5DFC                   cmp ebx, dword ptr [ebp-04]  <--关键!!比较前半部分注册码
                                                       ? ebx 得正确的前半部分注册码
:004ACAF5 7404                     je 004ACAFB                  <---不等就死!
:004ACAF7 33C0                     xor eax, eax
:004ACAF9 EB45                     jmp 004ACB40

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACAF5(C)

|

:004ACAFB 33C0                     xor eax, eax
:004ACAFD 33DB                     xor ebx, ebx
:004ACAFF BA03000000               mov edx, 00000003
:004ACB04 8B4D08                   mov ecx, dword ptr [ebp+08]
:004ACB07 83C103                   add ecx, 00000003       <---定位于输入注册名的第 3 个字符
:004ACB0A 3B55F4                   cmp edx, dword ptr [ebp-0C]
:004ACB0D 7D23                     jge 004ACB32

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACB30(C)

|

:004ACB0F 0FB631                   movzx esi, byte ptr [ecx]  <---[ecx] 中的第一个字符的十六进制值
                                                      赋给 esi
:004ACB12 0FB679FF                 movzx edi, byte ptr [ecx-01] <---[ecx] 中的前一个字符(即第 2 个)的十六进制值
                                                      赋给 edi
:004ACB16 0FAFF7                   imul esi, edi           <---将相乘结果赋给 esi
:004ACB19 0FAF3485B8035500         imul esi, dword ptr [4*eax+005503B8] <---将相乘结果赋给 esi
                                                  005503B8 中的值见最后.
:004ACB21 03DE                     add ebx, esi            <---将相加结果赋给 ebx
:004ACB23 40                       inc eax
:004ACB24 83F826                   cmp eax, 00000026
:004ACB27 7E02                     jle 004ACB2B            <---你的注册名不会大于 29 位吧!!!
:004ACB29 33C0                     xor eax, eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004ACB27(C)

|

:004ACB2B 42                       inc edx
:004ACB2C 41                       inc ecx
:004ACB2D 3B55F4                   cmp edx, dword ptr [ebp-0C]      <---和注册名长度比较
:004ACB30 7CDD                     jl 004ACB0F            <---小于，继续计算
```



\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004ACB0D(C)

|
```

```
:004ACB32 3B5DF8                 cmp ebx, dword ptr [ebp-08]      <--关键!!比较后半部分注册码

      ? ebx 得正确的后半部分注册码

:004ACB35 7404                   je 004ACB3B    <---不等就死!

:004ACB37 33C0                   xor eax, eax

:004ACB39 EB05                   jmp 004ACB40
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:004ACB35(C)

|
```

```
:004ACB3B B801000000             mov eax, 00000001
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
|:004ACA72(U), :004ACA8A(U), :004ACAA7(U), :004ACAF9(U), :004ACB39(U)

|
```

```
:004ACB40 5F                     pop edi

:004ACB41 5E                     pop esi

:004ACB42 5B                     pop ebx

:004ACB43 8BE5                   mov esp, ebp

:004ACB45 5D                     pop ebp

:004ACB46 C20800                 ret 0008
```

```
...
```

************************************************************************************************************************************

4、`005503B8` 中的值好象是一个事先列好的表,其值依次取

```
0B    06    11    0C

0C    0E    05    0C

10    0A    0B    06

0E    0E    04    0B

06    0E    0E    04

0B    09    0C    0B

0A    08    0A    0A

10    08    04    06

0A    0C    10    08

0A    04    10
```

不知道我的理解对不对，还请各位大侠指点！

5、我的注册信息：

```
Name:nanmu@263.net

   code:9277-812430
```

6、信息存放于：

```
HKEY_CURRENT_USER\Software\mIRC\License 9277-812430

HKEY_CURRENT_USER\Software\mIRC\UserName nanmu@263.net

HKEY_USERS\.DEFAULT\Software\mIRC\License  9277-812430

HKEY_USERS\.DEFAULT\Software\mIRC\UserName nanmu@263.net  4dd466d0e1e0cc123afbe504da2462ca
```
