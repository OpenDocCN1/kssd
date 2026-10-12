
# Unpack SafeDisc v2.90.40 (更新代码修复)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-611.htm`](https://www.pediy.com/kssd/pediy07/pediy7-611.htm)

Unpacking SafeDisc v2.90.40 -- 樱花大战 3 简体中文标准版（如果你的 HP 不是满的话，还是先去喝几个血瓶吧）

`[工具]`：`ODbyDYK`，`ImportREC`，`LordPE`

大家都知道 `SafeDisc` 好一个用驱动来欺负我们的 `OD`，但是我们真的会让 `OD` 被他欺负吗？

用 `OD` 载入 `Sakura3.exe`。

```
020B305E >   55               push     ebp    <-OD 停在了这里

020B305F     8BEC             mov      ebp, esp

020B3061     60               pushad   <-大家看，多么友好的提示。

020B3062     BB 5E300B02      mov      ebx, offset Sakura3.<ModuleEntryPoint>

020B3067     33C9             xor      ecx, ecx

020B3069     8A0D 3D300B02    mov      cl, byte ptr ds:[20B303D]

020B306F     85C9             test     ecx, ecx

向下找 POPAD。一共找到了三个大家可以看看哪个比较象。

020B30BA     85C9             test     ecx, ecx

020B30BC     74 09            je       short Sakura3.020B30C7

020B30BE     61               popad

020B30BF     5D               pop      ebp

020B30C0     B8 00000000      mov      eax, 0

020B30C5   ^ EB 97            jmp      short Sakura3.<ModuleEntryPoint>

020B30CD     FFD0             call     eax

020B30CF     61               popad

020B30D0     5D               pop      ebp

020B30D1     EB 46            jmp      short Sakura3.020B3119

020B30D3     807C24 08 00     cmp      byte ptr ss:[esp+8], 0

020B3115     61               popad

020B3116     1360 0D          adc      esp, dword ptr ds:[eax+D]

020B3119   - E9 09BF5FFE     jmp      Sakura3.006AF027

020B311E     CC               int3
```

然后你小按几下 `F8`，就会发现刚才的入口点被修正成了

```
020B305E >- E9 C4BF5FFE      jmp      Sakura3.006AF027

020B3063     5E               pop      esi

020B3064     300B             xor      byte ptr ds:[ebx], cl

020B3066     0233             add      dh, byte ptr ds:[ebx]
```

然而，事实证明，真正有用的是 `020B3119` 那里的跳转。

先说一下 `safedisc` 的运做过程，先把自己需要的连接库解压到临时目录，写为`*.tmp` 文件。然后 `LoadLibrary`。

再跳到载入的`~df394b.tmp` 模块进行调试器检测，加载光盘检测模块，解压缩大部分主程序，再进行光盘检测向主程序写入剩下的一些数据，返回。

下面进入正题：

对 `IsDebuggerPresent` 下断（软件/硬件断点都行，但是我选择硬件断点，感觉保险些。下面相同）。显得有些多此一举，用插件能避开的还搞那么麻烦，而且我也用插件避开了，但是我们需要定位后面的调试器检测。

断下后取消硬件断点返回到这里：

```
100099EC     8BF0             mov      esi, eax

100099EE     66:85F6          test     si, si

100099F1     74 13            je       short ~df394b.10009A06

100099F3     E8 8278FFFF      call     ~df394b.1000127A

100099F8     66:8BF0          mov      si, ax

100099FB     66:F7DE          neg      si

100099FE     1BF6             sbb      esi, esi

10009A00     46               inc      esi

10009A01     66:85F6          test     si, si

10009A04     75 13            jnz      short ~df394b.10009A19

10009A06     8B4424 08        mov      eax, dword ptr ss:[esp+8]

10009A0A     8B08             mov      ecx, dword ptr ds:[eax]

10009A0C     81E1 EA894267    and      ecx, 674289EA

10009A12     8908             mov      dword ptr ds:[eax], ecx

10009A14     66:8BC6          mov      ax, si

10009A17     5E               pop      esi

10009A18     C3               retn   <-继续返回
```

到这里：

```
1000457A     83C4 04          add      esp, 4

1000457D     66:85C0          test     ax, ax

10004580     0F85 02010000    jnz      ~df394b.10004688

10004586     F6C3 04          test     bl, 4

10004589     76 12            jbe      short ~df394b.1000459D

1000458B     56               push     esi

1000458C     E8 3F530000      call     ~df394b.100098D0

10004591     83C4 04          add      esp, 4

10004594     66:85C0          test     ax, ax

10004597     0F85 EB000000    jnz      ~df394b.10004688

1000459D     F6C3 08          test     bl, 8

100045A0     76 12            jbe      short ~df394b.100045B4

100045A2     56               push     esi

100045A3     E8 58520000      call     ~df394b.10009800

100045A8     83C4 04          add      esp, 4

100045AB     66:85C0          test     ax, ax

100045AE     0F85 D4000000    jnz      ~df394b.10004688

100045B4     F7C3 00080000    test     ebx, 800

100045BA     76 12            jbe      short ~df394b.100045CE

100045BC     56               push     esi

100045BD     E8 5E510000      call     ~df394b.10009720

100045C2     83C4 04          add      esp, 4

100045C5     66:85C0          test     ax, ax

100045C8     0F85 BA000000    jnz      ~df394b.10004688  <-我们需要避开的是这个，不要让他跳转就行了。

100045CE     F7C3 00100000    test     ebx, 1000

100045D4     76 12            jbe      short ~df394b.100045E8

100045D6     56               push     esi

100045D7     E8 84500000      call     ~df394b.10009660

100045DC     83C4 04          add      esp, 4

100045DF     66:85C0          test     ax, ax

100045E2     0F85 A0000000    jnz      ~df394b.10004688

100045E8     F7C3 00200000    test     ebx, 2000

100045EE     76 12            jbe      short ~df394b.10004602

100045F0     56               push     esi

100045F1     E8 BA4F0000      call     ~df394b.100095B0

100045F6     83C4 04          add      esp, 4

100045F9     66:85C0          test     ax, ax

100045FC     0F85 86000000    jnz      ~df394b.10004688

10004602     F6C3 01          test     bl, 1

10004605     76 0E            jbe      short ~df394b.10004615

10004607     56               push     esi

10004608     E8 834E0000      call     ~df394b.10009490

1000460D     83C4 04          add      esp, 4

10004610     66:85C0          test     ax, ax

10004613     75 73            jnz      short ~df394b.10004688

10004615     F6C3 02          test     bl, 2

10004618     76 0E            jbe      short ~df394b.10004628

1000461A     56               push     esi

1000461B     E8 504D0000      call     ~df394b.10009370

10004620     83C4 04          add      esp, 4

10004623     66:85C0          test     ax, ax

10004626     75 60            jnz      short ~df394b.10004688

10004628     F6C3 10          test     bl, 10

1000462B     76 0E            jbe      short ~df394b.1000463B

1000462D     56               push     esi

1000462E     E8 DD4C0000      call     ~df394b.10009310

10004633     83C4 04          add      esp, 4

10004636     66:85C0          test     ax, ax

10004639     75 4D            jnz      short ~df394b.10004688

1000463B     F6C3 20          test     bl, 20

1000463E     76 0E            jbe      short ~df394b.1000464E

10004640     56               push     esi

10004641     E8 1A4C0000      call     ~df394b.10009260

10004646     83C4 04          add      esp, 4

10004649     66:85C0          test     ax, ax

1000464C     75 3A            jnz      short ~df394b.10004688

1000464E     F6C3 40          test     bl, 40

10004651     76 0E            jbe      short ~df394b.10004661

10004653     56               push     esi

10004654     E8 D74A0000      call     ~df394b.10009130

10004659     83C4 04          add      esp, 4

1000465C     66:85C0          test     ax, ax

1000465F     75 27            jnz      short ~df394b.10004688

10004661     F7C3 80000000    test     ebx, 80

10004667     76 0E            jbe      short ~df394b.10004677

10004669     56               push     esi

1000466A     E8 B1490000      call     ~df394b.10009020

1000466F     83C4 04          add      esp, 4

10004672     66:85C0          test     ax, ax

10004675     75 11            jnz      short ~df394b.10004688

10004677     F7C3 00400000    test     ebx, 4000

1000467D     76 09            jbe      short ~df394b.10004688

1000467F     56               push     esi

10004680     E8 CB480000      call     ~df394b.10008F50

10004685     83C4 04          add      esp, 4

10004688     5E               pop      esi

10004689     5B               pop      ebx

1000468A     C3               retn
```

注意：由于我的 `OD` 是修改版，所以只需要改一个跳转。你自己调试时，务必不能让其中任何一个 `jnz` 跳下去。改的时候，只能修改标志位 `Z=1`。因为后面这个库还有模块完整性的检查。

然后，我们走啊走，走到这里：

```
1000357D     FF75 F8          push     dword ptr ss:[ebp-8]                          ; ~df394b.10000000

10003580     E8 20750100      call     ~df394b.1001AAA5     <-进入这个 call

10003585     8B43 14          mov      eax, dword ptr ds:[ebx+14]

10003588     59               pop      ecx

1001AAA5             55                 push ebp

1001AAA6             8BEC               mov ebp,esp
```
