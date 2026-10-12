
# Magic Photo Recovery 2.0 程序完全破解

> 原文：[`www.pediy.com/kssd/pediy12/133684.html`](https://www.pediy.com/kssd/pediy12/133684.html)

【文章标题】: `Magic Photo Recovery 2.0` 程序完全破解

【文章作者】: `jiaosc`

【作者邮箱】: `jiaosc@163.com`

【软件名称】: `Magic Photo Recovery 2.0`

【软件大小】: `569kB`

【下载地址】: 自己搜索下载

【加壳方式】: `ASPack 2.12` -> `Alexey Solodovnikov`

【保护方式】: 注册码

【编写语言】: `Borland Delphi 6.0 - 7.0`

【使用工具】: `OD`, `Import Fix 1.6`, `keymake2.0`

【操作平台】: `WIN XP`

【软件介绍】: 不可思议的照片补救软件，它可以恢复被删除的图象!

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

【详细过程】

`PEiD v0.95` 查壳显示为: `ASPack 2.12` -> `Alexey Solodovnikov`

用 `OD` 手动脱壳

`OD` 载入，停在这里:

```
005E4001 >   60                 pushad
 005E4002    E8 03000000        call    Magic_Ph.005E400A
 005E4007  - E9 EB045D45        jmp     45BB44F7
 005E400C    55                 push    ebp
 005E400D    C3                 retn
 005E400E    E8 01000000        call    Magic_Ph.005E4014
 005E4013    EB 5D             jmp     short Magic_Ph.005E4072
 005E4015    BB EDFFFFFF        mov     ebx, -13
 005E401A    03DD              add     ebx, ebp
 005E401C    81EB 00401E00      sub     ebx, 1E4000
 005E4022    83BD 22040000 0>  cmp     dword ptr ss:[ebp+422], 0
 005E4029    899D 22040000      mov     dword ptr ss:[ebp+422], ebx
 005E402F    0F85 65030000      jnz     Magic_Ph.005E439A
```

用 `ESP` 定律脱壳，在 `0012ffa4` 处设硬件访问断点，按 `F9` 运行，停在这里:

```
005E43B0   /75 08             jnz     short Magic_Ph.005E43BA
 005E43B2   |B8 01000000       mov     eax, 1
 005E43B7   |C2 0C00          retn    0C
 005E43BA   \68 489E5200       push    Magic_Ph.00529E48
 005E43BF    C3               retn
 005E43C0    8B85 26040000     mov     eax, dword ptr ss:[ebp+426]
 005E43C6    8D8D 3B040000     lea     ecx, dword ptr ss:[ebp+43B]
```

按几下 `F7` 到达 `OEP`：

```
 00529E48    55               push    ebp       这里就是 OEP,     OEP 偏移地址=00129E48
 00529E49    8BEC             mov     ebp, esp
 00529E4B    83C4 F0          add     esp, -10
 00529E4E    53               push    ebx
 00529E4F    B8 D0735200      mov     eax, Magic_Ph.005273D0
 00529E54    E8 D3D5EDFF      call    Magic_Ph.0040742C
 00529E59    8B1D 48E65200    mov     ebx, dword ptr ds:[52E648]  ;Magic_Ph. 005325B0
 00529E5F    8B03             mov     eax, dword ptr ds:[ebx]
 00529E61    E8 AE65F4FF      call    Magic_Ph.00470414
 00529E66    8B03             mov     eax, dword ptr ds:[ebx]
```

`OEP` 处用 `OD` 插件 `OllyDump` `Dump debugged process` 脱壳，用 `Import Fix 1.6` 修复一下。

再用 `PEiD v0.95` 查壳显示为: `Borland Delphi 6.0 - 7.0` 脱壳干净，点击运行正常，

再次用 `OD` 将脱壳后的程序载入，按 `F9` 运行，出现主界面.

输入假注册名和假注册码，点确定，出现错误注册框.

用 `F12` 堆栈调用法找到调用错误注册框的地方，操作方法：按 `F12`，再按 `Alt+K` 调用堆栈，堆栈内出现下列内容：

调用堆栈： 主线程

地址 堆栈 函数过程 / 参数 调用来自 结构
```
0012E580   77D19418   包含 ntdll.KiFastSystemCallRet           USER32.77D19416             0012E5B4
0012E584   77D2770A   USER32.WaitMessage                      USER32.77D27705             0012E5B4
0012E5B8   77D249C4   USER32.77D2757B                         USER32.77D249BF             0012E5B4
0012E5E0   77D3A956   USER32.77D2490E                         USER32.77D3A951             0012E5DC
0012E8A0   77D3A2BC   USER32.SoftModalMessageBox              USER32.77D3A2B7             0012E89C
0012E9F0   77D663FD   USER32.77D3A147                         USER32.77D663F8             0012E9EC
0012EA48   77D664A2   USER32.MessageBoxTimeoutW               USER32.77D6649D             0012EA44
0012EA7C   77D50877   ? USER32.MessageBoxTimeoutA            USER32.77D50872             0012EA78
0012EA9C   77D5082F   ? USER32.MessageBoxExA                 USER32.77D5082A             0012EA98
0012EAA0   00090332     hOwner = 00090332 ('输入注册信息',
0012EAA4   00B2C810     Text = ""B4,"砦蟮?,D7,"",A2,"",B2
0012EAA8   00B38290     Title = "Magic Photo Recovery 2.0"
0012EAAC   00000030     Style = MB_OK|MB_ICONEXCLAMATION|M
0012EAB0   00000000     LanguageID = 0 (LANG_NEUTRAL)
0012EAB8   00470796   ? <jmp.&USER32.MessageBoxA>   Magic_Ph.00470791             0012EAB4
0012EABC   00090332     hOwner = 00090332 ('输入注册信息',
0012EAC0   00B2C810     Text = ""B4,"砦蟮?,D7,"",A2,"",B2
0012EAC4   00B38290     Title = "Magic Photo Recovery 2.0"
0012EAC8   00000030     Style = MB_OK|MB_ICONEXCLAMATION|M
0012EB40   00524A69   ? Magic_Ph.00470698   Magic_Ph.00524A64   0012EB3C
```

选中最后一行右键，选 `show call`，停在这里:

```
 00524A64   .  E8 2FBCF4FF   call    Magic_Ph.00470698
 00524A69   >  33C0          xor     eax, eax
 00524A6B   .  5A            pop     edx
 00524A6C   .  59            pop     ecx
 00524A6D   .  59            pop     ecx
 00524A6E   .  64:8910       mov     dword ptr fs:[eax], edx
 00524A71   .  EB 0A         jmp     short Magic_Ph.00524A7D
 00524A73   .^ E9 DCFCEDFF   jmp     Magic_Ph.00404754
```

上面这个 `Call` 就是调用注册错误提示框.我们往前翻找到段首:

```
 005246B8   .  55            push    ebp
 005246B9   .  8BEC          mov     ebp, esp
 005246BB   .  B9 0E000000   mov     ecx, 0E
 005246C0   >  6A 00         push    0
 005246C2   .  6A 00         push    0
 005246C4   .  49            dec     ecx
 005246C5   .^ 75 F9        jnz     short Magic_Ph.005246C0
 005246C7   .  53            push    ebx
 005246C8   .  56            push    esi
 005246C9   .  57            push    edi
 005246CA   .  8945 FC       mov     dword ptr ss:[ebp-4], eax
```

在 `005246B8` 段首处 `F2` 下断：跟一下，

```
 005246F7   .  E8 3CD7F2FF   call    Magic_Ph.00451E38
 005246FC   .  8B45 F0       mov     eax, dword ptr ss:[ebp-10]    ;  这里提取假注册名
 005246FF   .  8D55 F4       lea     edx, dword ptr ss:[ebp-C]
 00524702   .  E8 3151EEFF   call    Magic_Ph.00409838
 00524707   .  8B55 F4       mov     edx, dword ptr ss:[ebp-C]
 0052470A   .  8BC3          mov     eax, ebx
 0052470C   .  E8 57D7F2FF   call    Magic_Ph.00451E68
 00524711   .  8D55 E8       lea     edx, dword ptr ss:[ebp-18]
 00524714   .  8B45 FC       mov     eax, dword ptr ss:[ebp-4]
 00524717   .  8B98 64030000 mov     ebx, dword ptr ds:[eax+364]
 0052471D   .  8BC3          mov     eax, ebx
 0052471F   .  E8 14D7F2FF   call    Magic_Ph.00451E38
 00524724   .  8B45 E8       mov     eax, dword ptr ss:[ebp-18]    ;  这里提取假注册码
 00524727   .  8D55 EC       lea     edx, dword ptr ss:[ebp-14]
 0052472A   .  E8 0951EEFF   call    Magic_Ph.00409838
 0052472F   .  8B55 EC       mov     edx, dword ptr ss:[ebp-14]
 00524823   .  50            push    eax                       ;  这里 EAX 出现假注册码
 00524824   .  8D55 CC       lea     edx, dword ptr ss:[ebp-34]
 00524827   .  8B45 FC       mov     eax, dword ptr ss:[ebp-4]
 0052482A   .  8B80 60030000 mov     eax, dword ptr ds:[eax+360]
 00524830   .  E8 03D6F2FF   call    Magic_Ph.00451E38
 00524835   .  8B45 CC       mov     eax, dword ptr ss:[ebp-34]
 00524838   .  8D55 D0       lea   edx, dword ptr ss:[ebp-30] ;  这里 EAX 出现假注册名
 0052483B   .  E8 A03EFFFF   call    Magic_Ph.005186E0
 00524840   .  8B55 D0       mov     edx, dword ptr ss:[ebp-30]
 00524843   .  58            pop     eax
 00524844   .  E8 C70CEEFF   call    Magic_Ph.00405510  这里是关键 call;  啊,太阳出来了!寄存器 EAX 出现假注册码,EDX 显示真注册码.
 00524849   .  0F85 AA010000 jnz     Magic_Ph.005249F9  这里是关键跳转;  这里不能跳,一跳就注册失败。如爆破就 nop 掉.
 0052484F   .  8B45 FC       mov     eax, dword ptr ss:[ebp-4]
 00524852   .  C680 84030000>mov      byte ptr ds:[eax+384], 1
 00524859   .  8D55 C8       lea     edx, dword ptr ss:[ebp-38]
 0052485C   .  8B45 FC       mov     eax, dword ptr ss:[ebp-4]
```
