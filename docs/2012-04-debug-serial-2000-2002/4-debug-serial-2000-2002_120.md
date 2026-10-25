
# Visual Zip Password Recovery Processor v3.2

> 原文：[`www.pediy.com/kssd/pediy01/forum007.htm`](https://www.pediy.com/kssd/pediy01/forum007.htm)

破解 Visual Zip Password Recovery Processor v3.2（转载希望保持完整）

**作者：** 丁丁虾 又名：DDXia \[CCG\]

**级别：** 初级

昨晚，赶写方案，第一次写真是累坏我，左忙忙，右忙忙，耗到晚上 11 点，

，写你不容易！今早又要陪领导去客户处导游，起得特别的早，去华军处看看有什么

好的软件，真的发现一个好软件。

**最新版本：** 3.2(4.0 Beta 3)

**文件大小：** 528KB

**软件授权：** 共享软件

**使用平台：** Win95/98/NT

**软件简介：**

ZIP 压缩文件密码恢复软件，最大的特点是支持多处理器和网络运算，使得其能够发挥更快的速度去寻找你丢失的密码。使用图形操作界面，可自定最小与最大密码长度及数字、字母、符号来'查'密码。亦有中断密码查询及储存功能，下次要恢复前次的查询时，只需将储存的文件读入即可继续上次的查询，亦提供多种不同暴力查询模式，请勿用于不法用途。

`http://www.newhua.com/down/vzprp32.zip`

安装后，发现它只有 315KB，一想功能可以支持多处理器和网络运算，功能如此强大，为何这么小呢？？肯定是 PACK 过的，CHECK 一下，果然是用 `ASPack 1.02b` 压缩，UNPACK IT（功力还没有到这一层，只好借助一下解压罗^\_^），依然用 `W32DASM`--破解中的精品。

老方法，查询出错信息-----"Password is invalid!"；可以看到以下代码：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0047B79C(C)`

`|^^^^^^^^^^^-------->这个是追查的关键。发现 REG 错误后，跳到这里。`

```assembly
:0047B930 6A00                     push 00000000
:0047B932 668B0D70B94700          mov cx, word ptr [0047B970]
:0047B939 B201                     mov dl, 01
```

`* Possible StringData Ref from Code Obj ->"Password is invalid!"`

```
|
:0047B93B B89CB94700             mov eax, 0047B99C
:0047B940 E89742FDFF             call 0044FBDC
```

看一看 `0047B79C` 处的代码：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0047B796(C)`

```
|
:0047B78F 8A10                     mov dl, byte ptr [eax]
:0047B791 0055D8                   add byte ptr [ebp-28], dl
:0047B794 40                       inc eax
:0047B795 4B                       dec ebx
:0047B796 75F7                     jne 0047B78F
:0047B798 807DD800                 cmp byte ptr [ebp-28], 00
:0047B79C 0F858E010000             jne 0047B930
```

`^^^^^^^^^^^^----->试一试把它 nop 掉`

```assembly
:0047B7A2 BB40000000              mov ebx, 00000040
:0047B7A7 8B35E0EC4700            mov esi, dword ptr [0047ECE0]
```

哈哈。。。一注册就 THANK YOU！

别急，它还有一个信息提示："Unregistered version!" 再把它给去掉吧！

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`:0047B0AB(C)`

```
|
:0047B0A4 8A10                     mov dl, byte ptr [eax]
:0047B0A6 0055D8                   add byte ptr [ebp-28], dl
:0047B0A9 40                       inc eax
:0047B0AA 4B                       dec ebx
:0047B0AB 75F7                     jne 0047B0A4
:0047B0AD 807DD800                 cmp byte ptr [ebp-28], 00
:0047B0B1 7473                     je 0047B126
```

`^^^^^^^^^^^----->改为 JMP 0047B126`

如何找到它呢？----查找"Unregistered version!"

```assembly
:0047B0B3 8B45EC                  mov eax, dword ptr [ebp-14]
:0047B0B6 8B80DC020000            mov eax, dword ptr [eax+000002DC]
```

`* Possible StringData Ref from Code Obj ->"Unregistered version!"`

```
|
:0047B0BC BAECB14700             mov edx, 0047B1EC
:0047B0C1 E87E24FBFF             call 0042D544
:0047B0C6 A100EF4700             mov eax, dword ptr [0047EF00]
:0047B0CB C60000                 mov byte ptr [eax], 00
:0047B0CE 8B45EC                 mov eax, dword ptr [ebp-14]
:0047B0D1 8B80DC020000           mov eax, dword ptr [eax+000002DC]
:0047B0D7 8B4058                 mov eax, dword ptr [eax+58]
:0047B0DA BAFF000000             mov edx, 000000FF
:0047B0DF E878AFF9FF             call 0041605C
```

收工，我要上班了。

完成时间
2000.2.25 上午 8:10
耗时 14 分钟 4dd466d0e1e0cc123afbe504da2462ca
