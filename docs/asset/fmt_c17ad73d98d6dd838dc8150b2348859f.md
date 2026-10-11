
# LanSentry

> 原文：[`www.pediy.com/kssd/pediy04/kanxue347.htm`](https://www.pediy.com/kssd/pediy04/kanxue347.htm)

LanSentry 的破解过程

破解：D-X-C

破解时间：2002.1.19

该软件用狗保护，用 `VC` 编写，先反汇编，找线索，我前几天还解过一个狗，发现解狗的一种较为简便和省力的方法是在 `DOG` 之外做文章。大家请看下文：

```asm
:00406C68 A1BC7F5500                 mov eax, dword ptr [00557FBC] //可见这是一个关键地址。
:00406C6D 85C0                       test eax, eax
:00406C6F 7407                       je 00406C78
:00406C71 68C07F5500                 push 00557FC0
:00406C76 EB05                       jmp 00406C7D
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| `:00406C6F`(C)
|

\* Possible StringData Ref from Data Obj -> `"[未授权用户！试用时间:一小时]"`

```asm
:00406C78 6828D84400                 push 0044D828
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| `:00406C76`(U)
|

```asm
:00406C7D 8D8EDC010000               lea ecx, dword ptr [esi+000001DC]
```

下 `BPM` `557FBC` 跟踪，

```asm
:00409C76 E825010000                 call 00409DA0
:00409C7B E890AC0000                 call 00414910
:00409C80 8B0D404C4500               mov ecx, dword ptr [00454C40] //由地址 454C40 送入。
```

\* Reference To: `KERNEL32.CreateThread`, Ord:`004Ah`

```asm
:00409C86 8B2D54F24300               mov ebp, dword ptr [0043F254]
:00409C8C 8D542414                   lea edx, dword ptr [esp+14]
:00409C90 89442410                   mov dword ptr [esp+10], eax
:00409C94 52                         push edx
:00409C95 6A00                       push 00000000
:00409C97 56                         push esi
:00409C98 6850514100                 push 00415150
:00409C9D 6A00                       push 00000000
:00409C9F 6A00                       push 00000000
:00409CA1 890DBC7F5500               mov dword ptr [00557FBC], ecx  //此处置 557FBC 的值，向上看这个值是哪来的。
:00409CA7 FFD5                       call ebp
:00409CA9 8BF8                       mov edi, eax
```

下 `BPM` `454C40` 重新跟踪，

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| `:00409AE1`(C)
|

```asm
:00409AFC E82FFBFFFF                 call 00409630 //此 CALL 是关键，跟入
:00409B01 F7D8                       neg eax
:00409B03 1BC0                       sbb eax, eax
:00409B05 40                         inc eax
:00409B06 A3404C4500                 mov dword ptr [00454C40], eax  //此处送入
:00409B0B 0F8587000000               jne 00409B98
:00409B11 E8BAB60000                 call 004151D0
:00409B16 85C0                       test eax, eax
:00409B18 A3404C4500                 mov dword ptr [00454C40], eax
:00409B1D 7579                       jne 00409B98
:00409B1F 50                         push eax
```

》》》

\* Referenced by a CALL at Address:
| `:00409AFC`
|

```asm
:00409630 83EC48                     sub esp, 00000048
:00409633 B90A000000                 mov ecx, 0000000A
:00409638 33C0                       xor eax, eax
:0040963A 66C705BA7F55000000         mov word ptr [00557FBA], 0000
:00409643 56                         push esi
:00409644 57                         push edi
:00409645 BFC07F5500                 mov edi, 00557FC0
:0040964A F3                         repz
:0040964B AB                         stosd
:0040964C AA                         stosb
:0040964D B911000000                 mov ecx, 00000011
:00409652 33C0                       xor eax, eax
:00409654 8D7C240C                   lea edi, dword ptr [esp+0C]
:00409658 F3                         repz
:00409659 AB                         stosd
:0040965A 8D442414                   lea eax, dword ptr [esp+14]
```

\* Possible StringData Ref from Data Obj -> `"LAN-SENTRY"`

```asm
:0040965E BF14E84400                 mov edi, 0044E814
:00409663 A3B47F5500                 mov dword ptr [00557FB4], eax
:00409668 83C9FF                     or ecx, FFFFFFFF
:0040966B 33C0                       xor eax, eax
:0040966D F2                         repnz
:0040966E AE                         scasb
:0040966F F7D1                       not ecx
:00409671 49                         dec ecx
:00409672 66890DB87F5500             mov word ptr [00557FB8], cx
:00409679 E86E040100                 call 00419AEC
:0040967E 85C0                       test eax, eax
:00409680 740B                       je 0040968D  //此处不跳
:00409682 5F                         pop edi
:00409683 B801000000                 mov eax, 00000001 //EAX 被送入 1，所以将其改为 mov eax, 00000000
:00409688 5E                         pop esi
:00409689 83C448                     add esp, 00000048
:0040968C C3                         ret
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
| `:00409680`(C)
|

```asm
:0040968D 53                         push ebx
```

\* Possible StringData Ref from Data Obj -> `"LAN-SENTRY"`

```asm
:0040968E BE14E84400                 mov esi, 0044E814
:00409693 8D442418                   lea eax, dword ptr [esp+18]
```

》》》 》》》

并没完，经过上面的修改后，开始运行前的 `NAG` 没有了，但打开欲设置时软件就没了响应。

用 `BPM` `454C40`、`BPM` `557FBC` 跟踪，到下面

```asm
:0040A53D FFD7                       call edi
:0040A53F A1BC7F5500                 mov eax, dword ptr [00557FBC]
:0040A544 85C0                       test eax, eax //这句改成 JNE 0040A570
:0040A546 7428                       je 0040A570  //发现此处若不跳就死住了
:0040A548 BFC07F5500                 mov edi, 00557FC0
:0040A54D 83C9FF                     or ecx, FFFFFFFF
:0040A550 33C0                       xor eax, eax
:0040A552 F2                         repnz
```

》》》 》》》

跟踪时可发现 `00557FBC` 地址还用来判断是否到一小时后退出。

相关地址：`4063DA`、`405B00`、`406431`。  `4dd466d0e1e0cc123afbe504da2462ca`
