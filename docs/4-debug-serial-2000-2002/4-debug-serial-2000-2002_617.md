
# 飞天餐饮娱乐管理系统

> 原文：[`www.pediy.com/kssd/pediy03/forum434.htm`](https://www.pediy.com/kssd/pediy03/forum434.htm)

## 暴力破解法一

1）在 `004A2A48` 处

改：
`0F 85 8A 00 00 00 C7 05 74 2C 52 00 01 00 00 00`
`^^ ^^ ^^ ^^ ^^ ^^`

为：
`90 90 90 90 90 90`

2）在 `004A1B29` 处

改：
`74 04 33 C0 EB 0A 42 40 49 75 F1`
`^^`

为：
`EB`

然后，随便用你的姓名和足够长的 `Code` 注册，注册完以后关闭程序，重新运行你就可以看到结果了。

## 算法代码分析

此软件的注册码算法很复杂（真的不知道有没有弄懂的必要，呵呵）。

假设姓名为 `Name` 简称 `N` 字符串；注册码为 `Code` 简称 `C` 字符串；另外此算法还根据 `C` 得到了一个 `KEY` 简称 `K` 字符串。

令 `C' = Fc(C)` —— 其中 `Fc()` 为函数其意义为：将字符串 `C` 去掉 `-` 后两两合并，例如：`1234-56789012-3456` 合并成 `12 34 56 78 90 12 34 56`（16 进制）共八个字节。又如：`1a2b-3c4d5e6f-7g8h` 合并成：`1a 2b 3c 4d 5e 6f 7g 8h`。

这个函数是 `004A1AE3` 处的 `call 004A187C`。

```
:004A1AD0 55   push ebp
:004A1AD1 8BEC mov ebp, esp
:004A1AD3 83C4F0 add esp, FFFFFFF0
:004A1AD6 53   push ebx
:004A1AD7 56   push esi
:004A1AD8 57   push edi
:004A1AD9 8BD9 mov ebx, ecx
:004A1ADB 8BF8 mov edi, eax
:004A1ADD 8B7508 mov esi, dword ptr [ebp+08]
:004A1AE0 8D45F0 lea eax, dword ptr [ebp-10]
:004A1AE3 E894FDFFFF call 004A187C <-- Fc()函数，在这里下中断，用 do "d eax; p;"可以看到 C'字符串
:004A1AE8 85C0 test eax, eax
:004A1AEA 7504 jne 004A1AF0
:004A1AEC 33C0 xor eax, eax
:004A1AEE EB49 jmp 004A1B39
```

令 `K = Fk(C') = Fk(Fc(C))` —— 其中 `Fk()` 的定义如下面代码段：

```
:004A1AF0 8BC6 mov eax, esi
:004A1AF2 8D55F0 lea edx, dword ptr [ebp-10]
:004A1AF5 E806FDFFFF call 004A1800 <-- 这个 Call 是根据 C'得到一个 Key，简称 K 字符串
```

具体的定义为：将 `Fc(C)` 后面的 4 个字节的值分别 `*4+xxxx`（`xxxx` 表示 `0x0004`, `0x0404`, `0x0804`, `0x0C04`），将得到的值从一个长度为 `1000H` 的表中查找出值进行加和 `xor` 运算，得到的值作为入口参数再进行运算，共循环 `0x10` 次。代码如：

```
:004A14A6 C645F7F0 mov [ebp-09], F0
:004A14AA 8D75F0 lea esi, dword ptr [ebp-10]
:004A14AD 331E xor ebx, dword ptr [esi]
:004A14AF 8D95ACEFFFFF lea edx, dword ptr [ebp+FFFFEFAC]
:004A14B5 8BC3 mov eax, ebx
:004A14B7 E8D4FEFFFF call 004A1390 <-- 运算在这个 Call 中运行
:004A14BC 33F8 xor edi, eax
:004A14BE 8BC3 mov eax, ebx
:004A14C0 8BDF mov ebx, edi
:004A14C2 8BF8 mov edi, eax
:004A14C4 83EE04 sub esi, 00000004
:004A14C7 FE45F7 inc [ebp-09]
:004A14CA 75E1 jne 004A14AD
```

`Call 004A1390` 的代码为：

```
:004A1390 56 push esi
:004A1391 57 push edi
:004A1392 81C404F0FFFF add esp, FFFFF004
:004A1398 50 push eax
:004A1399 83C4FC add esp, FFFFFFFC
:004A139C 8BF2 mov esi, edx
:004A139E 8D7C2404 lea edi, dword ptr [esp+04]
:004A13A2 B900040000 mov ecx, 00000400
:004A13A7 F3 repz
:004A13A8 A5 movsd
:004A13A9 890424 mov dword ptr [esp], eax
:004A13AC 8BC4 mov eax, esp
:004A13AE 33D2 xor edx, edx
:004A13B0 8A5003 mov dl, byte ptr [eax+03]
:004A13B3 8B549404 mov edx, dword ptr [esp+4*edx+04]
:004A13B7 33C9 xor ecx, ecx
:004A13B9 8A4802 mov cl, byte ptr [eax+02]
:004A13BC 03948C04040000 add edx, dword ptr [esp+4*ecx+00000404]
:004A13C3 33C9 xor ecx, ecx
:004A13C5 8A4801 mov cl, byte ptr [eax+01]
:004A13C8 33948C04080000 xor edx, dword ptr [esp+4*ecx+00000804]
:004A13CF 0FB600 movzx eax, byte ptr [eax]
:004A13D2 039484040C0000 add edx, dword ptr [esp+4*eax+00000C04]
:004A13D9 8BC2 mov eax, edx
:004A13DB 81C404100000 add esp, 00001004
:004A13E1 5F pop edi
:004A13E2 5E pop esi
:004A13E3 C3 ret

--------------------------------------------------------------------------------

:004A1AFA 8903 mov dword ptr [ebx], eax
:004A1AFC 8B06 mov eax, dword ptr [esi]
```

下面的一个 `Call` 是计算另一个字符串的，假定为 `C''`，共分三步走：第一步：计算出 `N'=Fn(N)`；第二步：然后将 `N'` 的前 3 个字符用 `K` 字符串的前三个字符代替，假定为 `N''=F3(K, N')=F3(Fk(C'), N')=F3(Fk(Fc(C)), Fn(N))`；第三步：将得到的 `N''` 字符串按照 `Fk()` 类似的方法通过一定量的 `Add` 和 `xor` 运算后(也是调用了 `Call 004A1390`，只是不同的输入参数，哪怕是 `1bit` 的不同也会得到一个完全不同的新字符串，痛苦)，得到 `C''`，可以令 `C''=F(N'')`，那么如果 `C'==C''`，那么，一切就是 `OK`。嘿嘿

```
:004A1AFE 50 push eax
:004A1AFF 8B0B mov ecx, dword ptr [ebx]
:004A1B01 8D45F8 lea eax, dword ptr [ebp-08]
:004A1B04 8BD7 mov edx, edi
:004A1B06 E845FBFFFF call 004A1650 <-- 这个 Call 是根据 N 和 K 字符串得到一个新的字符串 C''=F0(N, K)
:004A1B0B 85C0 test eax, eax
:004A1B0D 7504 jne 004A1B13
:004A1B0F 33C0 xor eax, eax
:004A1B11 EB26 jmp 004A1B39
:004A1B13 8B0DC0F15100 mov ecx, dword ptr [0051F1C0]
:004A1B19 49 dec ecx
:004A1B1A 85C9 test ecx, ecx
:004A1B1C 7C16 jl 004A1B34
:004A1B1E 41 inc ecx
:004A1B1F 8D45F8 lea eax, dword ptr [ebp-08]
:004A1B22 8D55F0 lea edx, dword ptr [ebp-10]
:004A1B25 8A18 mov bl, byte ptr [eax]    |
:004A1B27 3A1A cmp bl, byte ptr [edx]     |
:004A1B29 EB04 jmp 004A1B2F              |不用看就知道是在比较两个运算好的
:004A1B2B 33C0 xor eax, eax             |新的字符串，其实就是我上面提到的
:004A1B2D EB0A jmp 004A1B39             |那个 C' 和 C''。相等就是了
:004A1B2F 42 inc edx                    |
:004A1B30 40 inc eax                    |
:004A1B31 49 dec ecx                    |
:004A1B32 75F1 jne 004A1B25             |
:004A1B34 B801000000 mov eax, 00000001 <--关键就是要得到这个 Eax=1；
:004A1B39 5F pop edi
:004A1B3A 5E pop esi
:004A1B3B 5B pop ebx
:004A1B3C 8BE5 mov esp, ebp
:004A1B3E 5D pop ebp
:004A1B3F C20400 ret 0004
```

最终你会得到一个表达式 `C'==C''` 即 `Fc(C)=F(N'')=F(F3(K, N'))=F(F3(Fk(Fc(C)), Fn(N)))`，那么谁能告诉我如何根据这个表达式用 `N` 求出 `C` 来？

前面提到的：`1a2b-3c4d5e6f-7g8h` 合并成：`1a 2b 3c 4d 5e 6f 7g 8h` 其实应该是：`1A 2B 3C 4D 5E 6F 7G 8H`，也就是说最后的转换都是数字和大写，如果你输入 `1a2B-3c4D5e6F-7g8H` 那么得到也是：`1A 2B 3C 4D 5E 6F 7G 8H`

## 暴力破解法二

1）在 `004A2B6B` 处

改：
`0F 86 9F 00 00 00 C7 05 74 2C 52 00 64 00 00 00`
`^^ ^^ ^^ ^^ ^^ ^^`

为：
`90 90 90 90 90 90`

2）在 `004A1B29` 处

改：
`74 04 33 C0 EB 0A 42 40 49 75 F1`
`^^`

为：
`EB`

然后，随便用你的姓名和足够长的 `Code` 注册，注册完以后关闭程序，重新运行你就可以看到结果了。

具体我没有仔细用过，不过好像网络版的功能全一些。。呵呵 `4dd466d0e1e0cc123afbe504da2462ca`
