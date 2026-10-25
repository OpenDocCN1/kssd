
# ChinaZip 5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum803.htm`](https://www.pediy.com/kssd/pediy03/forum803.htm)

破解心得之 `ChinaZip` 5.0（中华压缩）篇

**作者**：时空幻影

**时间**：2001 年 4 月 6 日

**工具**：`W32DASM8.93` 黄金版中文版、`TRW2000` 1.22 已注册版、`CASPR`（一种脱壳工具）、`FILEINFO` 2.43a。

**过程**：

先用 `FILEINFO` 查看 `ChinaZip.exe` 有没有加壳，果然它是由 `ASPack` v2.001 加的壳，用 `CASPR` 脱掉它的壳，再用 `W32DASM` 反汇编，点工具栏上的“STING REF（串式参考）”，查找可疑的字符串，果然看到字符串“您的注册码不正确!”，双击它，再往上翻，看看有什么跳转指令，果然马上找到了，如下所示：

```
:004DF699 8D4DF8                lea ecx, dword ptr [ebp-08] <--在 TRW2000 中用 PMODULE 后再按几次 F10 就会到这
:004DF69C 8B55FC                mov edx, dword ptr [ebp-04]
:004DF69F 8BC3                  mov eax, ebx
:004DF6A1 E87AFEFFFF            call 004DF520               <--这里按 F8 进入这里，这里面计算注册码
:004DF6A6 8D55F4                lea edx, dword ptr [ebp-0C]
:004DF6A9 A1A4AD4E00            mov eax, dword ptr [004EADA4]
:004DF6AE 8B00                  mov eax, dword ptr [eax]
:004DF6B0 8B80E4020000           mov eax, dword ptr [eax+000002E4]
:004DF6B6 E8BD37F5FF            call 00432E78
:004DF6BB 8B45F4                mov eax, dword ptr [ebp-0C]
:004DF6BE 8B55F8                mov edx, dword ptr [ebp-08]
:004DF6C1 E8324AF2FF            call 004040F8               <--这里面注册码进行比较
:004DF6C6 7568                  jne 004DF730                <--跳转的话就 GAME OVER 了
:004DF6C8 A108AF4E00            mov eax, dword ptr [004EAF08]
:004DF6CD 8B00                  mov eax, dword ptr [eax]
:004DF6CF 8B8008040000           mov eax, dword ptr [eax+00000408]
:004DF6D5 B201                  mov dl, 01
:004DF6D7 E8F090FEFF            call 004C87CC
:004DF6DC 8D55F0                lea edx, dword ptr [ebp-10]
:004DF6DF A1F8AF4E00            mov eax, dword ptr [004EAFF8]
:004DF6E4 8B00                  mov eax, dword ptr [eax]
:004DF6E6 E89515F7FF            call 00450C80
:004DF6EB 8D45F0                lea eax, dword ptr [ebp-10]
* Possible StringData Ref from Code Obj ->" - 注册用户!"
                                  |
:004DF6EE BA7CF74D00            mov edx, 004DF77C
:004DF6F3 E8F848F2FF            call 00403FF0
:004DF6F8 8B55F0                mov edx, dword ptr [ebp-10]
:004DF6FB A108AF4E00            mov eax, dword ptr [004EAF08]
:004DF700 8B00                  mov eax, dword ptr [eax]
:004DF702 E8A137F5FF            call 00432EA8
:004DF707 6A40                  push 00000040
* Possible StringData Ref from Code Obj ->"注册成功！"
                                  |
:004DF709 B98CF74D00            mov ecx, 004DF78C
* Possible StringData Ref from Code Obj ->"谢谢您的宝贵支持."
                                  |
:004DF70E BA98F74D00            mov edx, 004DF798
:004DF713 A1F8AF4E00            mov eax, dword ptr [004EAFF8]
:004DF718 8B00                  mov eax, dword ptr [eax]
:004DF71A E8951BF7FF            call 004512B4
:004DF71F A1EC345100            mov eax, dword ptr [005134EC]
:004DF724 C7803402000001000000   mov dword ptr [ebx+00000234], 00000001
:004DF72E EB0A                  jmp 004DF73A
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF6C6(C)
|
* Possible StringData Ref from Code Obj ->"您的注册码不正确!"
                                  |
:004DF730 B8B4F74D00            mov eax, 004DF7B4       <--双击字符串“您的注册码不正确!”来到这，然后往上看
:004DF735 E81288F7FF            call 00457F4C
```

在那个 `CALL 004DF520` 按 `F8` 进入后就看到以下指令：

```
* Referenced by a CALL at Address:
|:004DF6A1
|
:004DF520 55                    push ebp
:004DF521 8BEC                  mov ebp, esp
:004DF523 6A00                  push 00000000
:004DF525 6A00                  push 00000000
:004DF527 6A00                  push 00000000
:004DF529 6A00                  push 00000000
:004DF52B 6A00                  push 00000000
:004DF52D 6A00                  push 00000000
:004DF52F 6A00                  push 00000000
:004DF531 53                    push ebx
:004DF532 56                    push esi
:004DF533 57                    push edi
:004DF534 894DF8                mov dword ptr [ebp-08], ecx
:004DF537 8955FC                mov dword ptr [ebp-04], edx
:004DF53A 8B45FC                mov eax, dword ptr [ebp-04]
:004DF53D E85A4CF2FF            call 0040419C
:004DF542 33C0                  xor eax, eax
:004DF544 55                    push ebp
:004DF545 6808F64D00            push 004DF608
:004DF54A 64FF30                push dword ptr fs:[eax]
:004DF54D 648920                mov dword ptr fs:[eax], esp
:004DF550 33F6                  xor esi, esi
:004DF552 8D45F4                lea eax, dword ptr [ebp-0C]
:004DF555 8B55FC                mov edx, dword ptr [ebp-04]
:004DF558 E8A348F2FF            call 00403E00
:004DF55D 8B45F4                mov eax, dword ptr [ebp-0C]
:004DF560 E8834AF2FF            call 00403FE8                   <--这里计算用户名的长度
:004DF565 8BF8                  mov edi, eax
:004DF567 85FF                  test edi, edi                   <--判断用户名长度是否为 0
:004DF569 7E57                  jle 004DF5C2
:004DF56B BB01000000            mov ebx, 00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF5C0(C)
|
:004DF570 8B45F4                mov eax, dword ptr [ebp-0C]
:004DF573 8A4418FF              mov al, byte ptr [eax+ebx-01] <--[EAX]为用户名的内存表示
:004DF577 E858FFFFFF            call 004DF4D4                   <--判断 AL 里的数是否为素数
:004DF57C 84C0                  test al, al                     <--为素数的话 AL 为 1，否则为 0
:004DF57E 7425                  je 004DF5A5
:004DF580 8D45E8                lea eax, dword ptr [ebp-18]     <--以下为计算注册码字符部分
:004DF583 8B55F4                mov edx, dword ptr [ebp-0C]
:004DF586 8A541AFF              mov dl, byte ptr [edx+ebx-01]
:004DF58A E88149F2FF            call 00403F10
:004DF58F 8B45E8                mov eax, dword ptr [ebp-18]
:004DF592 8D55EC                lea edx, dword ptr [ebp-14]
:004DF595 E81294F2FF            call 004089AC                     <--小写转换成大写
:004DF59A 8B55EC                mov edx, dword ptr [ebp-14]
:004DF59D 8D45F0                lea eax, dword ptr [ebp-10]
:004DF5A0 E84B4AF2FF            call 00403FF0                     <--合并字符
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF57E(C)
|
:004DF5A5 83FB01                cmp ebx, 00000001                 <--以下为计算注册码数字部分
:004DF5A8 740A                  je 004DF5B4
:004DF5AA 8B45F4                mov eax, dword ptr [ebp-0C]
:004DF5AD 0FB64418FE            movzx eax, byte ptr [eax+ebx-02]
:004DF5B2 EB06                  jmp 004DF5BA
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF5A8(C)
|
:004DF5B4 8B45F4                mov eax, dword ptr [ebp-0C]
:004DF5B7 0FB600                movzx eax, byte ptr [eax]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF5B2(U)
|
:004DF5BA 8D748612              lea esi, dword ptr [esi+4*eax+12]
:004DF5BE 43                    inc ebx
:004DF5BF 4F                    dec edi
:004DF5C0 75AE                  jne 004DF570
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF569(C)
|
:004DF5C2 8D55E4                lea edx, dword ptr [ebp-1C]
:004DF5C5 8BC6                  mov eax, esi        <--ESI 为注册码数字部分的十六进制
:004DF5C7 E8A497F2FF            call 00408D70        <--十六进制转换成十进制的字符串
:004DF5CC 8B4DE4                mov ecx, dword ptr [ebp-1C]
```



```
:004DF5CF 8D45F4             lea eax, dword ptr [ebp-0C]
:004DF5D2 8B55F0             mov edx, dword ptr [ebp-10]
:004DF5D5 E85A4AF2FF         call 00404034      <-- 把注册码的字符部分与数字部分合并
:004DF5DA 8B45F8             mov eax, dword ptr [ebp-08]
:004DF5DD 8B55F4             mov edx, dword ptr [ebp-0C]
:004DF5E0 E8D747F2FF         call 00403DBC
:004DF5E5 33C0               xor eax, eax
:004DF5E7 5A                 pop edx
:004DF5E8 59                 pop ecx
:004DF5E9 59                 pop ecx
:004DF5EA 648910             mov dword ptr fs:[eax], edx
:004DF5ED 680FF64D00         push 004DF60F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004DF60D(U)
|
:004DF5F2 8D45E4             lea eax, dword ptr [ebp-1C]
:004DF5F5 BA05000000         mov edx, 00000005
:004DF5FA E88D47F2FF         call 00403D8C
:004DF5FF 8D45FC             lea eax, dword ptr [ebp-04]
:004DF602 E86147F2FF         call 00403D68
:004DF607 C3                 ret
```

这是我第一次写破解心得，错误遗漏在所难免，希望大家指正。

由于我没时间写注册机了，哪位编程爱好者写一下吧！！呵呵！！！`4dd466d0e1e0cc123afbe504da2462ca`
