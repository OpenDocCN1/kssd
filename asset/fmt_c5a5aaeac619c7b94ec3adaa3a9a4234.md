
# freeRes

> 原文：[`www.pediy.com/kssd/pediy04/kanxue089.htm`](https://www.pediy.com/kssd/pediy04/kanxue089.htm)

用 `Keymake` `1.2` 版本做它的注册机！

第一、用 `FI` 得知是用 `ASP` 加壳！先用 `UNASP` 脱壳！

第二、反汇编如下：

```
反汇编可见：
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004B6D49(C)

:004B6D5B 8B45F0               mov eax, dword ptr [ebp-10]
:004B6D5E E87DD1F4FF           call 00403EE0
:004B6D63 83F828               cmp eax, 00000028  // 比较注册码是否为 40 位 28 十六进制＝40 十进制！
:004B6D66 7538                 jne 004B6DA0
:004B6D68 8B45F4               mov eax, dword ptr [ebp-0C]
:004B6D6B E870D1F4FF           call 00403EE0
:004B6D70 85C0                 test eax, eax
:004B6D72 7E2C                 jle 004B6DA0
:004B6D74 68348C0000           push 00008C34
:004B6D79 8D45EC               lea eax, dword ptr [ebp-14]
:004B6D7C 50                   push eax
:004B6D7D B983310000           mov ecx, 00003183
:004B6D82 BAD6030000           mov edx, 000003D6
:004B6D87 8B45F4               mov eax, dword ptr [ebp-0C]
:004B6D8A E80DFCFFFF           call 004B699C
:004B6D8F 8B45EC               mov eax, dword ptr [ebp-14]
:004B6D92 8B55F0               mov edx, dword ptr [ebp-10]
:004B6D95 E856D2F4FF           call 00403FF0  // 经典故事 进入去！
:004B6D9A 7504                 jne 004B6DA0  // 一跳就玩完！
:004B6D9C C645FF01             mov [ebp-01], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
```

进入前面的 `:004B6D95 E856D2F4FF call 00403FF0` 如下：

```
:00403FF0 53                   push ebx
:00403FF1 56                   push esi
:00403FF2 57                   push edi
:00403FF3 89C6                 mov esi, eax
:00403FF5 89D7                 mov edi, edx
:00403FF7 39D0                 cmp eax, edx // 绝对经典 D EAX ＝真的注册码 40 位长 D EDX ＝假的注册码
:00403FF9 0F848F000000         je 0040408E
:00403FFF 85F6                 test esi, esi
:00404001 7468                 je 0040406B
:00404003 85FF                 test edi, edi
:00404005 746B                 je 00404072
:00404007 8B46FC               mov eax, dword ptr [esi-04]
:0040400A 8B57FC               mov edx, dword ptr [edi-04]
:0040400D 29D0                 sub eax, edx
:0040400F 7702                 ja 00404013
:00404011 01C2                 add edx, eax
```

用 `Keymake` `1.2` 版本做它的注册机！

一）选择 `F8`－另类注册机！

1、程序名称：`freeRes.exe`

2、添加数据：

中断地址：`4b6d95`

中断次数：`1`

第一字节：`E8`

指令长度：`5`

再次添加数据：

中断地址：`403ff7`

中断次数：`1`

第一字节：`39`

指令长度：`2`

二、选择内存方式 `EAX`

先第一次运行注册信息后，再运行本注册机！一切 `OK`！能显示 `40` 位长注册码！！！

绝密档案

`http://hongjian.126.com`

`4dd466d0e1e0cc123afbe504da2462ca`
