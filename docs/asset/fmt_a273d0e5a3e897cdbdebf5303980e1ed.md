
# 国图图书下载系统 9.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue293.htm`](https://www.pediy.com/kssd/pediy04/kanxue293.htm)

国图图书下载系统 9.0 的破解过程及注册码算法，谁能帮我写个注册机来？：）

## 国图图书下载系统 9.0 的破解过程及注册码算法

**作者**：Crack007

**主页**：`http://www.crack007.com` ( ^-^ )

**破解日期**：2001-11-26

**破解工具**：`TRW2000 1.22` `Fi2.49` `Wasm 8.93` 黄金版

**难度**：较易

**软件下载地址**：`http://www.widenstage.com/artech/product/nlcdownload/nlcdownload.exe`（1633KB）

**软件简介**：国家图书馆是继超星图书馆以来的又一家大型电子数字图书馆，现有图书 10 余万册，并且不断增加。本软件就是专门为下载国家图书馆的电子图书而设计。共享版只能下载不足 150 页的图书，或者图书的前 150 页。 您注册以后，将没有此限制。

主程序用 `NeoLite v2.0` 加壳，简单脱壳后反汇编得到如下信息：

```
:0040F484 E827370C00                   call 004D2BB0
:0040F489 8BC3                         mov eax, ebx
:0040F48B E864090000                   call 0040FDF4
:0040F490 8BC3                         mov eax, ebx
:0040F492 E879030000                   call 0040F810  //看看下面提示语句就知道很经典的，让我们跟进这个 Call
:0040F497 84C0                         test al, al
:0040F499 7460                         je 0040F4FB     //跳则死
:0040F49B 6A00                         push 00000000

* Possible StringData Ref from Data Obj ->"感谢您，您已经注册成功!

重新启动系统，注册将会"

                                                          ->"生效!"

                                                    |
:0040F49D BA15E24E00                   mov edx, 004EE215
:0040F4A2 66C746102C00                 mov [esi+10], 002C
:0040F4A8 8D45E4                       lea eax, dword ptr [ebp-1C]
:0040F4AB E864350C00                   call 004D2A14
:0040F4B0 FF461C                       inc [esi+1C]
:0040F4B3 8B10                         mov edx, dword ptr [eax]
:0040F4B5 52                           push edx
:0040F4B6 6A02                         push 00000002
:0040F4B8 6A00                         push 00000000
:0040F4BA 8D4DBA                       lea ecx, dword ptr [ebp-46]
:0040F4BD 51                           push ecx
:0040F4BE E885940B00                   call 004C8948
:0040F4C3 83C40C                       add esp, 0000000C
:0040F4C6 8D45BA                       lea eax, dword ptr [ebp-46]
:0040F4C9 804DBA04                     or byte ptr [ebp-46], 04
:0040F4CD 668B08                       mov cx, word ptr [eax]
:0040F4D0 58                           pop eax
:0040F4D1 B202                         mov dl, 02
:0040F4D3 E8E8310700                   call 004826C0
:0040F4D8 FF4E1C                       dec [esi+1C]
:0040F4DB 8D45E4                       lea eax, dword ptr [ebp-1C]
:0040F4DE BA02000000                   mov edx, 00000002
:0040F4E3 E8C8360C00                   call 004D2BB0
:0040F4E8 C7833402000001000000         mov dword ptr [ebx+00000234], 00000001
:0040F4F2 8BC3                         mov eax, ebx
:0040F4F4 E83BAB0600                   call 0047A034
:0040F4F9 EB58                         jmp 0040F553

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040F499(C)
|
:0040F4FB 6A00                         push 00000000
* Possible StringData Ref from Data Obj ->"注册码错误!请重新输入!"
                                                          |
:0040F4FD BA49E24E00                   mov edx, 004EE249
:0040F502 66C746103800                 mov [esi+10], 0038

---------------- 来自 0040F492 E879030000        call 0040F810----
:0040F810 55                           push ebp
:0040F811 8BEC                         mov ebp, esp
:0040F813 83C4A8                       add esp, FFFFFFA8
:0040F816 53                           push ebx
:0040F817 56                           push esi
:0040F818 57                           push edi
:0040F819 8BD8                         mov ebx, eax
:0040F81B 8D75B8                       lea esi, dword ptr [ebp-48]
:0040F81E B8B0E64E00                   mov eax, 004EE6B0
:0040F823 E83C940B00                   call 004C8C64
:0040F828 8BC3                         mov eax, ebx
:0040F82A E8BD030000                   call 0040FBEC
:0040F82F 66C746100800                 mov [esi+10], 0008
:0040F835 33D2                         xor edx, edx
:0040F837 33C9                         xor ecx, ecx
:0040F839 8955FC                       mov dword ptr [ebp-04], edx
:0040F83C 33C0                         xor eax, eax
:0040F83E FF461C                       inc [esi+1C]
:0040F841 8D933C030000                 lea edx, dword ptr [ebx+0000033C]
:0040F847 66C746101400                 mov [esi+10], 0014
:0040F84D 66C746102000                 mov [esi+10], 0020
:0040F853 894DF8                       mov dword ptr [ebp-08], ecx
:0040F856 FF461C                       inc [esi+1C]
:0040F859 66C746101400                 mov [esi+10], 0014
:0040F85F 66C746102C00                 mov [esi+10], 002C
.......省略基本无关初始化的语句，来到下面的计算注册码的核心代码：

* Possible StringData Ref from Data Obj ->"0x"

                                                          |
:0040F9C6 B8F2E24E00                   mov eax, 004EE2F2
:0040F9CB E828350C00                   call 004D2EF8
:0040F9D0 8D45E4                       lea eax, dword ptr [ebp-1C]
:0040F9D3 8B00                         mov eax, dword ptr [eax]
:0040F9D5 E89AD00A00                   call 004BCA74
:0040F9DA 8945B0                       mov dword ptr [ebp-50], eax
:0040F9DD 8955B4                       mov dword ptr [ebp-4C], edx
:0040F9E0 FF4E1C                       dec [esi+1C]
:0040F9E3 8D45E4                       lea eax, dword ptr [ebp-1C]
:0040F9E6 BA02000000                   mov edx, 00000002
:0040F9EB E8C0310C00                   call 004D2BB0
:0040F9F0 FF4E1C                       dec [esi+1C]
:0040F9F3 8D45E8                       lea eax, dword ptr [ebp-18]
:0040F9F6 BA02000000                   mov edx, 00000002
:0040F9FB E8B0310C00                   call 004D2BB0
:0040FA00 66C746101400                 mov [esi+10], 0014
:0040FA06 33DB                         xor ebx, ebx
:0040FA08 EB6E                         jmp 0040FA78

-------------- 以上作用主要是把机器码转为 16 进制数，即 0x...... ----------------

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040FAA4(C)
|
:0040FA0A 837DFC00                     cmp dword ptr [ebp-04], 00000000  <---[ebp-04]保存的是我们填入的 E-mail。判断是否为空
:0040FA0E 7408                         je 0040FA18
:0040FA10 8B45FC                       mov eax, dword ptr [ebp-04]      <---E-mail 地址
:0040FA13 8B50FC                       mov edx, dword ptr [eax-04]      <---E-mail 字符位数
:0040FA16 EB02                         jmp 0040FA1A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040FA0E(C)
|
:0040FA18 33D2                         xor edx, edx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040FA16(U)
|
:0040FA1A 3BDA                         cmp ebx, edx          <--- ebx 初始值为 0 ，并判断该计算 E-mail 字符位数是否大于已经循环的次数
:0040FA1C 7D3B                         jge 0040FA59          <--- 若大于已经循环的次数，则转到 0040FA59 进行处理
:0040FA1E 8D7B01                       lea edi, dword ptr [ebx+01]
:0040FA21 57                          push edi
:0040FA22 8D45FC                       lea eax, dword ptr [ebp-04]
:0040FA25 50                          push eax
:0040FA26 E8652F0C00                   call 004D2990
:0040FA2B 83C408                       add esp, 00000008
:0040FA2E 8D45FC                       lea eax, dword ptr [ebp-04]
:0040FA31 E8D2320C00                   call 004D2D08
:0040FA36 037DFC                       add edi, dword ptr [ebp-04]
:0040FA39 4F                          dec edi
:0040FA3A 0FBE07                       movsx eax, byte ptr [edi]
:0040FA3D 99                          cdq                     <-----CDQ 指令以方便做 64 位运算
:0040FA3E 52                          push edx
:0040FA3F 50                          push eax
:0040FA40 8B45B0                       mov eax, dword ptr [ebp-50]      <---保存机器码后 8 位计算的累计结果
:0040FA43 8B55B4                       mov edx, dword ptr [ebp-4C]      <---保存机器码前 8 位计算的累计结果
:0040FA46 E8D1BE0B00                   call 004CB91C

----------------------call 004CB91C---start--------------------------
```
