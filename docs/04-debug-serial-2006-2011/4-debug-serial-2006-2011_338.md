
# xx 职业介绍软件注册算法简单分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-295.htm`](https://www.pediy.com/kssd/pediy09/pediy09-295.htm)

> 【文章标题】: 久灵职业介绍管理注册算法简单分析

> 【文章作者】: `ziyihou`

> 【详细过程】

> 前 2 天在论坛上看到有人求助这个软件,顺便下来看一下，首先用 `FI` 查壳，发现是 `UPX v0.94-1.92`，比较弱的壳，直接用 `OD` 载入手脱，到达入口点后用 `OD` 自带的 `OllyDump` 保存，无需修复，脱壳后直接运行。用 `OD` 载入脱壳后的文件`unpack.exe`，运行，进入后在软件的主菜单点“帮助--注册--输入以有注册码”，来到注册窗口，回到 `OD`，用 `OD` 的超级字串搜索，发现如下信息：

```
006C5070    B9 E0546C00      mov     ecx, 006C54E0                 ; 注册
006C5075    BA E8546C00      mov     edx, 006C54E8                 ; 注册码有误
```

> 在 `OD` 中找到 `006C5070` 位置，向上看，发现在 `006C506C` 处可以跳过这个错误，因此在判断的地方也就是 `006C5069` 处按 `F2`，回到程序注册窗口，输入用户名 `ziyihou`，随便输入注册码，点注册，程序断在这里：

```
006C5064    E8 83FAD3FF      call    00404AEC
006C5069    83F8 18          cmp     eax, 18                        <==比较注册码的位数是否是 18H，也就是 24 位
006C506C    74 1D            je      short 006C508B                 <==不是 24 位则出错
006C506E    6A 10            push    10
006C5070    B9 E0546C00      mov     ecx, 006C54E0                 ; 注册
006C5075    BA E8546C00      mov     edx, 006C54E8                 ; 注册码有误。
006C507A    A1 F0FC6D00      mov     eax, dword ptr [6DFCF0]
006C507F    8B00             mov     eax, dword ptr [eax]
006C5081    E8 B241DEFF      call    004A9238
006C5086    E9 AF030000      jmp     006C543A
006C508B    C645 FB 01       mov     byte ptr [ebp-5], 1
006C508F    8D55 F4          lea     edx, dword ptr [ebp-C]
006C5092    8B45 FC          mov     eax, dword ptr [ebp-4]
006C5095    8B80 9C030000    mov     eax, dword ptr [eax+39C]
006C509B    E8 08F9EBFF      call    005849A8
006C50A0    807D FB 00       cmp     byte ptr [ebp-5], 0
006C50A4    74 77            je      short 006C511D
006C50A6    C645 FB 00       mov     byte ptr [ebp-5], 0
006C50AA    8D45 D8          lea     eax, dword ptr [ebp-28]
006C50AD    50               push    eax
006C50AE    B9 01000000      mov     ecx, 1                         <==取试验码中的 1 位
006C50B3    BA 07000000      mov     edx, 7                         <==从试验码的第 7 位开始取
006C50B8    8B45 F4          mov     eax, dword ptr [ebp-C]        <==EAX 为试验码
006C50BB    E8 8CFCD3FF      call    00404D4C
006C50C0    8B45 D8          mov     eax, dword ptr [ebp-28]       <==EAX 为试验码的第 7 位
006C50C3    BA 00556C00      mov     edx, 006C5500                 <==EDX 为 2D(字符 -)
006C50C8    E8 6BFBD3FF      call    00404C38
006C50CD    75 4E            jnz     short 006C511D                 <==不等则出错
006C50CF    8D45 D4          lea     eax, dword ptr [ebp-2C]
006C50D2    50               push    eax
006C50D3    B9 01000000      mov     ecx, 1                         <==取试验码中的 1 位
006C50D8    BA 0D000000      mov     edx, 0D                        <==从试验码的第 0DH 位开始取,也就是第 13 位
006C50DD    8B45 F4          mov     eax, dword ptr [ebp-C]
006C50E0    E8 67FCD3FF      call    00404D4C
006C50E5    8B45 D4          mov     eax, dword ptr [ebp-2C]       <==EAX 为试验码的第 13 位
006C50E8    BA 00556C00      mov     edx, 006C5500                 <==EDX 为 2D(字符 -)
006C50ED    E8 46FBD3FF      call    00404C38
006C50F2    75 29            jnz     short 006C511D                 <==不等则出错
006C50F4    8D45 D0          lea     eax, dword ptr [ebp-30]
006C50F7    50               push    eax
006C50F8    B9 01000000      mov     ecx, 1                         <==取试验码中的 1 位
006C50FD    BA 13000000      mov     edx, 13                       <==从试验码的第 13H 位开始取,也就是第 19 位
006C5102    8B45 F4          mov     eax, dword ptr [ebp-C]
006C5105    E8 42FCD3FF      call    00404D4C
006C510A    8B45 D0          mov     eax, dword ptr [ebp-30]       <==EAX 为试验码的第 19 位
006C510D    BA 00556C00      mov     edx, 006C5500                 <==EDX 为 2D(字符 -)
006C5112    E8 21FBD3FF      call    00404C38
006C5117    75 04            jnz     short 006C511D                 <==不等则出错
006C5119    C645 FB 01       mov     byte ptr [ebp-5], 1
006C511D    807D FB 00       cmp     byte ptr [ebp-5], 0
```

> 由上面的分析可知注册码的形式为:`xxxxxx-xxxxx-xxxxx-xxxxx`

```
006C5121    74 46            je      short 006C5169
006C5123    C645 FB 00       mov     byte ptr [ebp-5], 0
006C5127    8D45 CC          lea     eax, dword ptr [ebp-34]
006C512A    50               push    eax
006C512B    B9 01000000      mov     ecx, 1                         <==取试验码中的 1 位
006C5130    BA 01000000      mov     edx, 1                         <==从试验码的第 1 位开始取
006C5135    8B45 F4          mov     eax, dword ptr [ebp-C]
006C5138    E8 0FFCD3FF      call    00404D4C
006C513D    8B45 CC          mov     eax, dword ptr [ebp-34]
006C5140    8A00             mov     al, byte ptr [eax]            <==al 为试验码的第 1 位的数值
006C5142    3C 41            cmp     al, 41                       <==al 中的值小于 41(即字母 A)则出错
006C5144    72 23            jb      short 006C5169
006C5146    8D45 C8          lea     eax, dword ptr [ebp-38]
006C5149    50               push    eax
006C514A    B9 01000000      mov     ecx, 1                         <==取试验码中的 1 位
006C514F    BA 01000000      mov     edx, 1                         <==从试验码的第 1 位开始取
006C5154    8B45 F4          mov     eax, dword ptr [ebp-C]
006C5157    E8 F0FBD3FF      call    00404D4C
006C515C    8B45 C8          mov     eax, dword ptr [ebp-38]
006C515F    8A00             mov     al, byte ptr [eax]            <==al 为试验码的第 1 位的数值
006C5161    3C 61            cmp     al, 61                       <==al 中的值大于等于 61(即字母 a)则出错
006C5163    73 04            jnb     short 006C5169
006C5165    C645 FB 01       mov     byte ptr [ebp-5], 1
006C5169    807D FB 00       cmp     byte ptr [ebp-5], 0
006C516D    0F84 96000000    je      006C5209
006C5173    C645 FB 00       mov     byte ptr [ebp-5], 0
006C5177    33C0             xor     eax, eax
006C5179    55               push    ebp
006C517A    68 FF516C00      push    006C51FF
006C517F    64:FF30          push    dword ptr fs:[eax]
006C5182    64:8920          mov     dword ptr fs:[eax], esp
006C5185    8D45 C4          lea     eax, dword ptr [ebp-3C]
006C5188    50               push    eax
006C5189    8D55 C0          lea     edx, dword ptr [ebp-40]
006C518C    8B45 FC          mov     eax, dword ptr [ebp-4]
006C518F    8B80 9C030000    mov     eax, dword ptr [eax+39C]
006C5195    E8 0EF8EBFF      call    005849A8
006C519A    8B45 C0          mov     eax, dword ptr [ebp-40]
006C519D    B9 04000000      mov     ecx, 4                         <==取试验码中的 4 位
006C51A2    BA 03000000      mov     edx, 3                         <==从试验码的第 3 位开始取
006C51A7    E8 A0FBD3FF      call    00404D4C
006C51AC    8B45 C4          mov     eax, dword ptr [ebp-3C]
006C51AF    E8 744BD4FF      call    00409D28
006C51B4    3D E8030000      cmp     eax, 3E8                       <==EAX 中为实验码 3 到 6 位的 16 进制值
006C51B9    7C 3A            jl      short 006C51F5                 <==如果小于 3E8(即十进制 1000)则出错
006C51BB    8D45 BC          lea     eax, dword ptr [ebp-44]
006C51BE    50               push    eax
006C51BF    8D55 B8          lea     edx, dword ptr [ebp-48]
006C51C2    8B45 FC          mov     eax, dword ptr [ebp-4]
006C51C5    8B80 9C030000    mov     eax, dword ptr [eax+39C]
```



```assembly
006C51CB    E8 D8F7EBFF     call    005849A8  
006C51D0    8B45 B8         mov     eax, dword ptr [ebp-48]  
006C51D3    B9 04000000     mov     ecx, 4                        <==取试验码中的 4 位  
006C51D8    BA 03000000     mov     edx, 3                        <==从试验码的第 3 位开始取  
006C51DD    E8 6AFBD3FF     call    00404D4C  
006C51E2    8B45 BC         mov     eax, dword ptr [ebp-44]  
006C51E5    E8 3E4BD4FF     call    00409D28  
006C51EA    3D 0F270000     cmp     eax, 270F                     <==EAX 中为实验码 3 到 6 位的 16 进制值  
006C51EF    7F 04           jg      short 006C51F5                <==如果大于 270F(即十进制 9999)则出错  
006C51F1    C645 FB 01      mov     byte ptr [ebp-5], 1  
006C51F5    33C0            xor     eax, eax  
006C51F7    5A              pop     edx  
006C51F8    59              pop     ecx  
006C51F9    59              pop     ecx  
006C51FA    64:8910         mov     dword ptr fs:[eax], edx  
006C51FD    EB 0A           jmp     short 006C5209  
006C51FF   ^ E9 F4EBD3FF     jmp     00403DF8  
006C5204    E8 1BF0D3FF     call    00404224  
006C5209    807D FB 00      cmp     byte ptr [ebp-5], 0  
006C520D    74 5E           je      short 006C526D  
006C520F    C645 FB 00      mov     byte ptr [ebp-5], 0  
006C5213    33C0            xor     eax, eax  
006C5215    55              push    ebp  
006C5216    68 63526C00     push    006C5263  
006C521B    64:FF30         push    dword ptr fs:[eax]  
006C521E    64:8920         mov     dword ptr fs:[eax], esp  
006C5221    8D45 B4         lea     eax, dword ptr [ebp-4C]  
006C5224    50              push    eax  
006C5225    8D55 B0         lea     edx, dword ptr [ebp-50]  
006C5228    8B45 FC         mov     eax, dword ptr [ebp-4]  
006C522B    8B80 9C030000   mov     eax, dword ptr [eax+39C]  
006C5231    E8 72F7EBFF     call    005849A8  
006C5236    8B45 B0         mov     eax, dword ptr [ebp-50]  
006C5239    B9 01000000     mov     ecx, 1                        <==取试验码中的 1 位   
006C523E    BA 0A000000     mov     edx, 0A                       <==从试验码的第 10 位开始取  
006C5243    E8 04FBD3FF     call    00404D4C  
006C5248    8B45 B4         mov     eax, dword ptr [ebp-4C]  
006C524B    E8 D84AD4FF     call    00409D28  
006C5250    83F8 04         cmp     eax, 4                        <==EAX 为试验码的第 10 位  
006C5253    7D 04           jge     short 006C5259                <==如果大于等于 4 则出错  
006C5255    C645 FB 01      mov     byte ptr [ebp-5], 1  
006C5259    33C0            xor     eax, eax  
006C525B    5A              pop     edx  
006C525C    59              pop     ecx  
006C525D    59              pop     ecx  
006C525E    64:8910         mov     dword ptr fs:[eax], edx  
006C5261    EB 0A           jmp     short 006C526D  
006C5263   ^ E9 90EBD3FF     jmp     00403DF8  
006C5268    E8 B7EFD3FF     call    00404224  
006C526D    807D FB 00      cmp     byte ptr [ebp-5], 0  
006C5271    74 5E           je      short 006C52D1  
006C5273    C645 FB 00      mov     byte ptr [ebp-5], 0  
006C5277    33C0            xor     eax, eax  
006C5279    55              push    ebp  
006C527A    68 C7526C00     push    006C52C7  
006C527F    64:FF30         push    dword ptr fs:[eax]  
006C5282    64:8920         mov     dword ptr fs:[eax], esp  
006C5285    8D45 AC         lea     eax, dword ptr [ebp-54]  
006C5288    50              push    eax  
006C5289    8D55 A8         lea     edx, dword ptr [ebp-58]  
006C528C    8B45 FC         mov     eax, dword ptr [ebp-4]  
006C528F    8B80 9C030000   mov     eax, dword ptr [eax+39C]  
006C5295    E8 0EF7EBFF     call    005849A8  
006C529A    8B45 A8         mov     eax, dword ptr [ebp-58]  
006C529D    B9 01000000     mov     ecx, 1                        <==取试验码中的 1 位  
006C52A2    BA 0B000000     mov     edx, 0B                       <==从试验码的第 11 位开始取  
006C52A7    E8 A0FAD3FF     call    00404D4C  
006C52AC    8B45 AC         mov     eax, dword ptr [ebp-54]  
006C52AF    E8 744AD4FF     call    00409D28  
006C52B4    83F8 05         cmp     eax, 5                        <==EAX 为试验码的第 10 位  
006C52B7    7C 04           jl      short 006C52BD                <==如果小于 5 则出错  
006C52B9    C645 FB 01      mov     byte ptr [ebp-5], 1  
006C52BD    33C0            xor     eax, eax  
006C52BF    5A              pop     edx  
006C52C0    59              pop     ecx  
006C52C1    59              pop     ecx  
006C52C2    64:8910         mov     dword ptr fs:[eax], edx  
006C52C5    EB 0A           jmp     short 006C52D1  
006C52C7   ^ E9 2CEBD3FF     jmp     00403DF8  
006C52CC    E8 53EFD3FF     call    00404224  
006C52D1    807D FB 00      cmp     byte ptr [ebp-5], 0  
006C52D5    0F84 8A000000   je      006C5365  
006C52DB    C645 FB 00      mov     byte ptr [ebp-5], 0  
006C52DF    33C0            xor     eax, eax  
006C52E1    55              push    ebp  
006C52E2    68 5B536C00     push    006C535B  
006C52E7    64:FF30         push    dword ptr fs:[eax]  
006C52EA    64:8920         mov     dword ptr fs:[eax], esp  
006C52ED    8D45 A4         lea     eax, dword ptr [ebp-5C]  
006C52F0    50              push    eax  
006C52F1    8D55 A0         lea     edx, dword ptr [ebp-60]  
006C52F4    8B45 FC         mov     eax, dword ptr [ebp-4]  
006C52F7    8B80 9C030000   mov     eax, dword ptr [eax+39C]  
006C52FD    E8 A6F6EBFF     call    005849A8  
006C5302    8B45 A0         mov     eax, dword ptr [ebp-60]  
006C5305    B9 01000000     mov     ecx, 1                        <==取试验码中的 1 位  
006C530A    BA 0C000000     mov     edx, 0C                       <==从试验码的第 12 位开始取  
006C530F    E8 38FAD3FF     call    00404D4C  
006C5314    8B45 A4         mov     eax, dword ptr [ebp-5C]  
006C5317    8A00            mov     al, byte ptr [eax]  
006C5319    3C 41           cmp     al, 41                        <==al 为试验码的第 12 位的数值   
006C531B    72 34           jb      short 006C5351                <==如果小于 41(即字母 A)则出错  
006C531D    8D45 9C         lea     eax, dword ptr [ebp-64]  
006C5320    50              push    eax  
006C5321    8D55 98         lea     edx, dword ptr [ebp-68]  
006C5324    8B45 FC         mov     eax, dword ptr [ebp-4]  
006C5327    8B80 9C030000   mov     eax, dword ptr [eax+39C]  
006C532D    E8 76F6EBFF     call    005849A8  
006C5332    8B45 98         mov     eax, dword ptr [ebp-68]  
006C5335    B9 01000000     mov     ecx, 1                        <==取试验码中的 1 位  
006C533A    BA 0C000000     mov     edx, 0C                       <==从试验码的第 12 位开始取  
006C533F    E8 08FAD3FF     call    00404D4C  
006C5344    8B45 9C         mov     eax, dword ptr [ebp-64]  
006C5347    8A00            mov     al, byte ptr [eax]                
006C5349    3C 5A           cmp     al, 5A                        <==al 为试验码的第 12 位的数值  
006C534B    73 04           jnb     short 006C5351                <==如果大于 5A(即字母 Z)则出错  
006C534D    C645 FB 01      mov     byte ptr [ebp-5], 1  
006C5351    33C0            xor     eax, eax  
006C5353    5A              pop     edx  
006C5354    59              pop     ecx  
006C5355    59              pop     ecx  
006C5356    64:8910         mov     dword ptr fs:[eax], edx  
006C5359    EB 0A           jmp     short 006C5365  
```



```
006C535B ^ E9 98EAD3FF jmp 00403DF8
006C5360 E8 BFEED3FF call 00404224
006C5365 807D FB 00 cmp byte ptr [ebp-5], 0
006C5369 75 2A jnz short 006C5395 <==以上条件满足则保存注册码,然后进行重起验证
006C536B 8D45 F4 lea eax, dword ptr [ebp-C]
006C536E BA 0C556C00 mov edx, 006C550C
006C5373 E8 4CF5D3FF call 004048C4
006C5378 6A 10 push 10
006C537A B9 E0546C00 mov ecx, 006C54E0 ; 注册
006C537F BA E8546C00 mov edx, 006C54E8 ; 注册码有误。
006C5384 A1 F0FC6D00 mov eax, dword ptr [6DFCF0]
006C5389 8B00 mov eax, dword ptr [eax]
006C538B E8 A83EDEFF call 004A9238
006C5390 E9 A5000000 jmp 006C543A
006C5395 807D FB 01 cmp byte ptr [ebp-5], 1
006C5399 0F85 9B000000 jnz 006C543A
```

经过以上分析,得知正确的注册码形式是这样的: 第一位为大写字母,第 3 到 6 位为数字,并且在 1000 到 9999 之间,第 7 13 19 位为`-`,

第 10 位为小于 4 的数字,第 11 位为大于 5 的数字,第 12 位为大写字母,因此我们如下构造试验码:

`F12845-6739K-9876G-43119`

然后点注册,系统提示注册信息保存成功,请重新启动软件.

明显是重起验证，搜索一下注册表，软件果然将注册信息写在注册表里，在注册表创建了

`\HKEY\_LOCAL\_SOFTWARE\xx 软件\职业介绍管理单机版`项,在里面有 3 个植,分别是 `Name`、`Pass`、`Ver`

其中为用户名 `ziyihou`，为我们输入的试验码 `F12845-6739K-9876G-43119`，看到这些，心里有点底了，重新用 `OD` 载入

脱壳后的文件 `unpack.exe`，下断点 `bpx RegQueryValueExA`，然后 `F9` 运行，结果郁闷了，程序被中断 N 次，但是没有一次

是读取我们输入的试验码的。经过几次失败后，冷静的分析了一下，想起来在软件的安装目录下还有一个名为 `xx_JOBPro.exe`

文件，会不会在这个文件里进行验证呢？想到这里，看了一下系统进程，这个文件果然不知道什么时候运行了！

用 `OD` 载入 `xx_JOBPro.exe`，脱壳，然后超级字串搜索，果然又有发现：

```
00570D98 |. BA 48125700 mov edx, 00571248 ; －－－未注册
```

然后上下看了看程序，发现其代码和上面分析的主文件基本上如出一辙，于是在这段程序开始的地方 `00570D64` 处 `F2`，运行，

程序断在这里：

```
00570D64 /. 55 push ebp <==ECX 为我们输入的试验码，EDX 为一个字符串
00570D65 |. 8BEC mov ebp, esp
00570D67 |. B9 15000000 mov ecx, 15
00570D6C |> 6A 00 /push 0
00570D6E |. 6A 00 |push 0
00570D70 |. 49 |dec ecx
00570D71 |.^ 75 F9 \jnz short 00570D6C
00570D73 |. 51 push ecx
00570D74 |. 53 push ebx
00570D75 |. 56 push esi
00570D76 |. 8BD8 mov ebx, eax
00570D78 |. 33C0 xor eax, eax
00570D7A |. 55 push ebp
00570D7B |. 68 2D125700 push 0057122D
00570D80 |. 64:FF30 push dword ptr fs:[eax]
00570D83 |. 64:8920 mov dword ptr fs:[eax], esp
00570D86 |. 8D55 EC lea edx, dword ptr [ebp-14]
00570D89 |. A1 14CD5800 mov eax, dword ptr [58CD14]
00570D8E |. 8B00 mov eax, dword ptr [eax]
00570D90 |. E8 7F6AF0FF call 00477814
00570D95 |. 8D45 EC lea eax, dword ptr [ebp-14]
00570D98 |. BA 48125700 mov edx, 00571248 ; －－－未注册
00570D9D |. E8 3A39E9FF call 004046DC
```

继续跟踪，发现程序果然在做和上面的分析一样的重复工作，由于我们已经按基本规则构造了试验码，因此这部分分析

略过，只取有用的部分：

```
00570486 . B9 01000000 mov ecx, 1 <==取试验码中的 1 位
0057048B . BA 02000000 mov edx, 2 <==从试验码的第 2 位开始取
00570490 . 8B45 E8 mov eax, dword ptr [ebp-18]
00570493 . E8 9C44E9FF call 00404934
00570498 . 8B55 CC mov edx, dword ptr [ebp-34] <==EDX 为试验码的第 2 位
0057049B . 58 pop eax <==EAX 为计算出的注册码的第 2 位
0057049C . E8 7F43E9FF call 00404820
005704A1 75 04 jnz short 005704A7 <==不等则为试用版(我的为 3)
005704A3 . C645 F6 01 mov byte ptr [ebp-A], 1
005704A7 > 807D F6 00 cmp byte ptr [ebp-A], 0
```

`..........................................................`

```
00570590 . B9 02000000 mov ecx, 2 <==取试验码中的 2 位
00570595 . BA 08000000 mov edx, 8 <==从试验码的第 8 位开始取
0057059A . 8B45 E8 mov eax, dword ptr [ebp-18]
0057059D . E8 9243E9FF call 00404934
005705A2 . 8B55 B8 mov edx, dword ptr [ebp-48] <==EDX 为试验码的第 8 9 位
005705A5 . 58 pop eax <==EAX 为计算出的注册码的第 8 9 位
005705A6 . E8 7542E9FF call 00404820
005705AB /75 04 jnz short 005705B1 <==不等为试用版(我的为 35)
```

```
005707FF . B9 04000000 mov ecx, 4 <==取试验码中的 4 位
00570804 . BA 15000000 mov edx, 15 <==从试验码的第 21 位开始取
00570809 . 8B45 E8 mov eax, dword ptr [ebp-18]
0057080C . E8 2341E9FF call 00404934
00570811 . 8B45 80 mov eax, dword ptr [ebp-80] <==EAX 中为实验码 21 到 24 位的 16 进制值(我的是 23D9)
00570814 . E8 478FE9FF call 00409760
00570819 . 2BF0 sub esi, eax <==ESI 中为计算出的注册码 21 到 24 位的 16 进制值
0057081B . 75 04 jnz short 00570821 <==不等为试用版(我的是 23D9,换成 10 进制为 9177)
```

继续跟踪发现程序只比较以上几位试验码,按照如上计算结果修改实验码为:

`F32845-3539K-9876G-49177`

PS：在 `00570D64 /. 55 push ebp` 处，

```
EAX 00E8B8B4
ECX 00EA36C8 ASCII "F32845-3539K-9876G-49177"
EDX 00EA35B8 ASCII "35db9da6b3"
EBX 0000000C
ESP 0012F9AC
EBP 0012FB60
ESI 0058F1BC Cds_JOBP.0058F1BC
EDI 00E8B8B4
EIP 005702C0 Cds_JOBP.005702C0
```

`EDX` 为一个字符串，经分析这个字符串是由用户名和机器码计算

生成的，用来计算注册码的，跟踪了一下，发现机器码是在 `xx_JOBPro.exe` 如下位置生成的：

```
00571A2A |. E8 ED1CE9FF call 0040371C <==这个 call 就是生成机器码的
00571A2F |. 8D85 DCFEFFFF lea eax, dword ptr [ebp-124]
00571A35 |. 50 push eax
00571A36 |. 8D95 C8FEFFFF lea edx, dword ptr [ebp-138]
```

由于本人水平所限，具体怎么用生成的机器码和用户名计算生成一个字符串来计算注册码，没有跟出来，所以也就无法写出

注册机来，希望那位高人能把用机器码和用户名生成字符串的过程写篇教程出来，让我等菜鸟学习一下！

--------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

`2007` 年 `10` 月 `05` 日 `9:00:07`

`4dd466d0e1e0cc123afbe504da2462ca`
