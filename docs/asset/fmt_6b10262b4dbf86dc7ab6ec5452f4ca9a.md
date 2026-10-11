
# 突破封锁线：第一章--脱壳篇

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-590.htm`](https://www.pediy.com/kssd/pediy08/pediy8-590.htm)

【文章标题】: 突破封锁线：第一章--脱壳篇

【文章作者】: `Austin`

【作者邮箱】: `austiny.cn@gmail.com`

【软件名称】: 神州数码网络客户登陆程序

【软件大小】: 不重要

【下载地址】: `http://www.dcnetworks.com.cn`

【加壳方式】: `UltraProtect 1.x`

【保护方式】: 加壳 网络封禁

【编写语言】: `VC6.0`

【使用工具】: `PEid` `OllyDbg` `LordPE` `ImportRec`

【操作平台】: `Windows2003`

【软件介绍】: 神州数码小区宽带网络登陆客户端，诸多限制

【作者声明】: 技术交流中。。。

--------------------------------------------------------------------------------

【详细过程】

## 一、（背景）山和云的彼端

那时候天很蓝，网很宽，资源可以共享，`BT` 随便可以下。后来有了客户端，一切化为泡影。

神州数码是一套小区宽带解决方案，使用 `802.x` 认证等技术，限制了网络的很多使用：

1. 帐号与机器绑定（换机器不能登陆）
2. 限制共享网络连接（发现代理，强制下线）
3. 限制 `BT` 下载（强制下线）
4. 无法从路由后连接（拒绝登陆）
...

诸多不便，一一破之。

## 二、（调查）英雄莫问出处

神州数码（`DigitalChina` 下简称 `DIGI`）安装文件夹大概就是一个主 `EXE` 文件外加一堆 `DLL` 文件和配置文件。

`DLL` 文件都是系统常用如 `MFC42.DLL` 和 `AMT.DLL` 等，故搞定 `EXE` 一个即可。

### 1. 先看看加壳否

用 `PEid`（侦壳工具）打开 `DIGI.EXE`，得到 `"UltraProtect 1.x -> RISCO Software Inc."`。（看来盔甲穿得挺厚）

### 2. 再看仔细些

用 `LordPE`（`DUMP` 和 `PE` 修改工具）打开，看到程序有 `5` 个段。

> 出特别指明，文中都使用十六进制

| 段名 | 地址 | 长度 |
| :--- | :--- | :--- |
| `.text` | `00410000` | `10000` |
| `.rdata` | `00411000` | `4000` |
| `.data` | `00415000` | `204000` |
| `.rsrc` | `00619000` | `3000` |
| `.perplex` | `0061C000` | `1D000` |

前 4 个都很典型（正常编译的程序一般都有这几个段），故不出意外，壳代码都在 `perplex` 段中。

### 3. 战略

`UltraProtect` 壳没有脱壳机，只有自己动手脱光它了：

`调试 -> 找 OEP -> dump -> 修复 IAT -> 修复资源 -> 减肥`

## 三、（行动）骡子还是马，拉出来遛遛先

用 `OllyDbg`（调试工具）装载，运行。啪！`OllyDbg` 退出了。（烈马！）看来壳中有反调试检查。

难不倒我们：打开 `HideOD` 插件的隐藏选项，再运行。再啪！`OllyDbg` 再次退出了。

（我看这马挺烈吧）（那是相当烈啊）

去看雪搜了一下，发现 `UltraProtect` 有检查加载者并于进程列表比对的习惯，而非常规的检查调试标志。

故 `HideOD` 骗不倒它。

再来，加载之，停在 `0061c000` 处：（正是在 `.perplex` 段中）

```assembly
0061C000 >  60              pushad
0061C001    FC              cld
0061C002    4D              dec     ebp
0061C003    72 01           jb      short 0061C006
0061C005    4E              dec     esi
0061C006    50              push    eax
0061C007    E8 01000000     call    0061C00D
0061C00C    71 58           jno     short 0061C066
0061C00E    58              pop     eax
0061C00F    0F84 03000000   je      0061C018
0061C015    66:13EF         adc     bp, di
0061C018    76 03           jbe     short 0061C01D
0061C01A    77 01           ja      short 0061C01D
0061C01C  - 7E C1           jle     short 0061BFDF
0061C01E    DD22            frstor  [edx]
0061C020    7C 03           jl      short 0061C025
0061C022    7D 01           jge     short 0061C025
0061C024    9A BA21D624 CBB>call    far BACB:24D621BA
```

将断点下至 `Process32First` 函数（获得进程列表会调用）

```assembly
bp Process32First+2
```

`F9` 运行，断在了 `7c85a26e` 处：（应该在 `Process32FirstA` 中）

```assembly
7C85A26E >  55              push    ebp
7C85A26F    8BEC            mov     ebp, esp
7C85A271    81EC 30020000   sub     esp, 230
7C85A277    A1 08D1887C     mov     eax, [7C88D108]
7C85A27C    56              push    esi
7C85A27D    8B75 0C         mov     esi, [ebp+C]
7C85A280    85F6            test    esi, esi
7C85A282    8945 FC         mov     [ebp-4], eax
7C85A285    8B45 08         mov     eax, [ebp+8]
7C85A288    0F84 9D000000   je      7C85A32B
7C85A28E    813E 28010000   cmp     dword ptr [esi], 128
7C85A294    0F82 91000000   jb      7C85A32B
7C85A29A    53              push    ebx
7C85A29B    8D8D D0FDFFFF   lea     ecx, [ebp-230]
7C85A2A1    51              push    ecx
7C85A2A2    50              push    eax
7C85A2A3    C785 D0FDFFFF 2>mov     dword ptr [ebp-230], 22C
7C85A2AD    E8 FDFeFFFF     call    Process32FirstW
```

`Shift+F9` 回到用户代码：

```assembly
00620B3F > /E9 AD000000      jmp     00620BF1            ;直接跳转到下面
00620B44   |60               pushad
00620B45   |8DBD E1444000    lea     edi, [ebp+4044E1]
00620B4B   |B9 26000000      mov     ecx, 26
00620B50   |E8 A9EDFFFF      call    0061F8FE
00620B55   |AB               stos    dword ptr es:[edi]
00620B56  ^|E2 F8           loopd   short 00620B50
00620B58   |61               popad
00620B59   |60               pushad
00620B5A   |E8 F2EFFFFF      call    0061FB51
          .....
00620BF1   \0BC0             or      eax, eax            ;跳转到此，单步往下走
00620BF3    0F84 27020000    je      00620E20            ;此处未跳转
00620BF9    8B95 45424000    mov     edx, [ebp+404245]    ;[ebp+404245]的值从 0 开始，依次为各进程 PID
00620BFF    3B95 35424000    cmp     edx, [ebp+404235]    ;[ebp+404235]中为父进程 OllyDbg 的 PID，此句进行比对
00620C05    0F84 DC000000    je      00620CE7            ;在比对前将[ebp+404235]的值改为 Explorer.exe 的 PID
00620C0B    B8 3D424000      mov     eax, 0040423D       ;就可以骗过 DIGI.EXE
00620C10    03C5             add     eax, ebp
00620C12    50               push    eax
00620C13    FFB5 39424000    push    dword ptr [ebp+404239]
00620C19    50               push    eax
00620C1A    8B85 05454000    mov     eax, [ebp+404505]
00620C20    8038 CC          cmp     byte ptr [eax], 0CC
00620C23    74 10            je      short 00620C35
```

不多说，改之。顺利通过检测。：）

清除刚才的断点。（重要，除非你愿意一次又一次重打第一关 `BOSS`）

## 四、（继续）万里长征第二步

通过了第一关，开始进行寻找 `OEP` 的伟大战役。

（注：`OEP`=`Original Entry Point`，就是加壳前程序开始的地方，也是我们 `dump` 的起点）

常识告诉我们，`OEP` 应该在 `.text` 段。（当然也有不是的时候，不过大多数的时候还是是的）

`Alt+M` 打开内存图，`F2` 设一个访问断点在 `.text` 段上。（没有用二次断点，因为幸运的是，此时代码段已经解压完毕）

`F9` 往下跑，程序停在 `0040fb74` 处，在 `.text` 段。（简直就是 `VC` 程序的标准入口，这里应该就是 `EOP` 没错了）

```assembly
0040FB74 > .  68 A82A4100    push    00412AA8
0040FB79   .  68 D4FC4000    push    0040FCD4                    ;  jmp 到 msvcrt._except_handler3; SE 处理程序安装
0040FB7E   .  64:A1 0000000>mov     eax, fs:[0]
0040FB84   .  50             push    eax
0040FB85   .  64:8925 00000>mov     fs:[0], esp
0040FB8C   .  83EC 68       sub     esp, 68
0040FB8F   .  53             push    ebx
0040FB90   .  56             push    esi
0040FB91   .  57             push    edi
0040FB92   .  8965 E8       mov     [ebp-18], esp
0040FB95   .  33DB          xor     ebx, ebx
0040FB97   .  895D FC       mov     [ebp-4], ebx
0040FB9A   .  6A 02          push    2
0040FB9C   .  FF15 64144100 call    [411464]                     ;  msvcrt.__set_app_type
0040FBA2   .  59             pop     ecx
0040FBA3   .  830D 5C8E6100>or      dword ptr [618E5C], FFFFFFFF
0040FBAA   .  830D 608E6100>or      dword ptr [618E60], FFFFFFFF
0040FBB1   .  FF15 68144100 call    [411468]                     ;  msvcrt.__p__fmode
0040FBB7   .  8B0D A06F4100 mov     ecx, [416FA0]
0040FBBD   .  8908          mov     [eax], ecx
0040FBBF   .  FF15 6C144100 call    [41146C]                     ;  msvcrt.__p__commode
0040FBC5   .  8B0D 9C6F4100 mov     ecx, [416F9C]
```

至此，`EOP` 已经找到。

由于可能多次重复，用 `OllyMachine` 插件完成整个上述过程：

```
;LOAD.TXT
;//////////////////////////////////////////////////////
invoke bp, 0x7c85a26e;Process32First+2
run
runtousercode
invoke bp, 0x7c85a26e;Process32First+2
run
invoke bc, 0x7c85a26e
runtousercode
invoke writememlong, 0x61f235, 1960, 2
invoke bp, 0x0040fb74
run
;//////////////////////////////////////////////////////
```

## 五、（出击）衣带渐宽终不悔

局势很明朗，`dump` 之。这里用了 `OllyDbg` 的插件 `OllyDump`。

注意 `dump` 的时候不要选择 `Rebuild Import`（术业有专攻，这个留给 `ImportRec` 来处理），保存为 `DIGI_DUMP.EXE`。

打开 `ImportRec`（修复 `IAT` 之王），在它的进程表中找到正在调试的 `DIGI`。

在 `OEP` 中填入刚刚找到的 `0000FB74`（偏移量），点击`"IAT AutoSearch"`，再点击`"Get Imports"`，得到 `Imported Functions` 列表。

表中有两块为解决的项：

- `? FThunk:00011048 NbFunc:2D (decimal:45) valid:NO`  这一项中有 45 个函数不可用
- `? FThunk:00011504 NbFunc:16 (decimal:22) valid:NO`  这一项中有 22 个函数不可用

注意表中 `00011048` 为偏移地址，所以实际地址应加上基地址 `00400000`，为 `00411048`。

在 `OllyDbg` 的数据窗口中来到 `00411048`（按地址方式显示）：
