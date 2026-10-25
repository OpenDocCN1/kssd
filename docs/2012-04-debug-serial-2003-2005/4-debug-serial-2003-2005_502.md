
# 完美卸载 V7.0

> 原文：[`www.pediy.com/kssd/pediy05/pediy50450.htm`](https://www.pediy.com/kssd/pediy05/pediy50450.htm)

目标软件： `完美卸载 V7.0`

软件简介：

软件共有 3 个小程序，分别是：磁盘垃圾清理软件、软件安装监视器、软件卸载工具，是一套功能比较强大的安装/卸载/清理工具，新版更加全面的系统监视，让软件安装的一举一动历历在目，二次清理技术，软件卸载后将不留任何垃圾，磁盘垃圾清理的数量更多，软件运行更加稳定。注册费为 20 元人名币。

使用工具： `W32Dasm 中文版(GOLD)`、`TRW2000 v1.22`。

破解过程：

`W32Dasm 中文版(GOLD)` 反汇编，串式参考："序列号输入错误!"

```
:00408602 E841BA0100              call 00424048       <===获得你输入的序列号。
:00408607 83C414                 add esp, 00000014
:0040860A 85C0                   test eax, eax       <====测试 eax.
:0040860C 7E3B                   jle 00408649        <====没输或不是数字则跳向出错点 A.
:0040860E 3DC8000000             cmp eax, 000000C8   <====序列号的第 5 组数字和 C8h 比较.(5)
:00408613 7F34                   jg 00408649         <====否则跳向出错点 A.
:00408615 8B542414               mov edx, dword ptr [esp+14]
:00408619 8D4C2410               lea ecx, dword ptr [esp+10]
:0040861D 51                     push ecx            <====ecx 入栈。
:0040861E 50                     push eax            <====第 5 组序列号入栈。
:0040861F 52                     push edx            <====第 4 组序列号入栈。
:00408620 55                     push ebp            <====第 3 组序列号入栈。
:00408621 53                     push ebx            <====第 2 组序列号入栈。
:00408622 57                     push edi            <====第 1 组序列号入栈。
:00408623 E8080B0100             call 00419130       <=========关键 CALL。
:00408628 83C418                 add esp, 00000018
:0040862B 84C0                   test al, al         <====测试 al.
:0040862D 6A30                   push 00000030
:0040862F 740C                   je 0040863D         <====不跳注册成功。

* Possible Reference to Dialog:
                |
:00408631 6888074500             push 00450788

* Possible StringData Ref from Data Obj ->"感谢您的支持与厚爱,在您的支持下我会开发出更好?
                                 ->"娜砑? 
如软件变动造成您的注册码不能注册，您"
                                 ->"可以凭您的姓名、申请码、详细资料来免费升级本软"
                                 ->"件！
软件自动升级功能：只需下载最新版本，软件"
                                 ->"自动识别注册信息！
请新启动本软件！"
                |
:00408636 68A8064500             push 004506A8
:0040863B EB15                   jmp 00408652

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040862F(C)
|

* Possible StringData Ref from Data Obj ->"错误"
                |
:0040863D 68A0064500             push 004506A0

* Possible StringData Ref from Data Obj ->"序列号输入错误或试图一码多用!"
                |
:00408642 6880064500             push 00450680
:00408647 EB09                   jmp 00408652

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040860C(C), :00408613(C)
|
:00408649 6A00                   push 00000000
:0040864B 6A00                   push 00000000

* Possible StringData Ref from Data Obj ->"序列号输入错误!"
                |
:0040864D 6870064500             push 00450670

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040863B(U), :00408647(U)
|
:00408652 8BCE                   mov ecx, esi
:00408654 E83ACF0200             call 00435593
:00408659 8D4C2410               lea ecx, dword ptr [esp+10]
:0040865D C78424C0000000FFFFFFFF  mov dword ptr [esp+000000C0], FFFFFFFF
:00408668 E82CEB0200             call 00437199
```

进入 `408623` 处的关键 `CALL`，直到：

```
:0041B82E E8EDD2FFFF             call 00418B20
:0041B833 8B8424C4020000         mov eax, dword ptr [esp+000002C4] <===将第 5 组序列号送入 eax.
:0041B83A 8B8C24B4020000         mov ecx, dword ptr [esp+000002B4] <===将第 1 组序列号送入 ecx.
:0041B841 8B6C242C               mov ebp, dword ptr [esp+2C]      <===将比较码 X1 送入 ebp.*
:0041B845 83C410                 add esp, 00000010                 <====esp+10H.
:0041B848 8B148588464500         mov edx, dword ptr [4*eax+00454688] <===将比较码 X2 送入 edx.*
:0041B84F 2BCD                   sub ecx, ebp                      <====ecx-ebp.
:0041B851 3BD1                   cmp edx, ecx                      <====ecx 和 edx 比较.(1)
:0041B853 0F851B020000           jne 0041BA74                      <====不等跳向出错点 B.
:0041B859 8B9424A8020000         mov edx, dword ptr [esp+000002A8] <===将第 2 组序列号送入 edx.
:0041B860 8B742418               mov esi, dword ptr [esp+18]       <===将比较码 X3 送入 esi.*
:0041B864 8B0C8554434500         mov ecx, dword ptr [4*eax+00454354] <===将比较码 X4 送入 ecx.*
:0041B86B 2BD6                   sub edx, esi                      <===edx-esi.
:0041B86D 3BCA                   cmp ecx, edx                      <===edx 和 ecx 比较.(2)
:0041B86F 0F85FF010000           jne 0041BA74                      <====不等跳向出错点 B.
:0041B875 8B8C24AC020000         mov ecx, dword ptr [esp+000002AC] <===将第 3 组序列号送入 ecx.
:0041B87C 8B742410               mov esi, dword ptr [esp+10]       <===将比较码 X5 送入 esi.*
:0041B880 8B148520404500         mov edx, dword ptr [4*eax+00454020] <===将比较码 X6 送入 edx.*
:0041B887 2BCE                   sub ecx, esi                      <===ecx-esi.
:0041B889 3BD1                   cmp edx, ecx                      <====ecx 和 edx 比较.(3)
:0041B88B 0F85E3010000           jne 0041BA74                      <====不等跳向出错点 B.
:0041B891 8B9424B0020000         mov edx, dword ptr [esp+000002B0] <===将第 4 组序列号送入 edx.
:0041B898 8B742414               mov esi, dword ptr [esp+14]       <===将比较码 X7 送入 esi.*
:0041B89C 8B0C85EC3C4500         mov ecx, dword ptr [4*eax+00453CEC] <===将比较码 X8 送入 ecx.*
:0041B8A3 2BD6                   sub edx, esi                      <====edx-esi.
:0041B8A5 3BCA                   cmp ecx, edx                      <====edx 和 ecx 比较.(4)
:0041B8A7 0F85C7010000           jne 0041BA74                      <====不等跳向出错点 B.
...............................................................<====下面是漫长处理过程，略。
                                 (注册信息放在"\AGPDRV.386")
```

`OK`，用 `TRW2000` 分别设断获得上面 `*` 处的比较码，然后通过计算获得正确的序列号。

例：输入序列号 `10000-20000-30000-40000-52`，`X1=2015`，`X2=10905`
(1)：`ecx=edx=ecx-X1=X2` => `ecx=12920`
(2)、(3)、(4)，略
(5)：不大于 `200`。

成功注册之后，`完美卸载 V7.0` 将在你的系统目录的 `"SYSTEM"` 文件夹中产生一个名为 `"AGPDRV.386"` 的隐藏文件（一般在 `C:\WINDOWS\SYSTEM\AGPDRV.386`），将其删除之后又变为未注册版本。

整理：

用户名：`fengma`

序列号：`12770-8388-30077-31046-7`

风马 于 `21:08 2002-8-18` `4dd466d0e1e0cc123afbe504da2462ca`
