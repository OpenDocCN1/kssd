
# WebSynchronizer V1.0 build 20

> 原文：[`www.pediy.com/kssd/pediy06/pediy6986.htm`](https://www.pediy.com/kssd/pediy06/pediy6986.htm)

`ASProtect` 1.23 Beta 21 脱壳+破解——WebSynchronizer V1.0 build 20

下载页面：`http://www.skycn.com/soft/12108.html`

软件大小：`979 KB`

软件语言：`英文`

软件类别：`国外软件 / 共享版 / 数据备份`

应用平台：`Win9x/NT/2000/XP`

加入时间：`2003-6-12`

下载次数：`859`

推荐等级：`*****`

【软件简介】：档案同步化工具 - `WebSynchronizer`，由知名续传软件 `ReGet` 之软件出版公司所推出，是网站同步化、档案镜像、档案备份的绝佳工具，可以执行下列主要工作：
1. 本机资料夹及远程资料夹的同步化；
2. 两台远程计算机中的资料夹同步化；
3. 两个本机资料之同步化。

【保护方式】：`ASProtect` 壳+`30` 天试用

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg` 1.09、`PEiD`、`LordPE`、`ImportREC` V1.4.2+

## 一、脱壳过程：

`WebSynchronizer` V1.0 build 20 是 N 久以前下载的，现在或许已经升级了。

设置 `Ollydbg` 忽略除了“内存访问异常”之外的所有其它异常选项。老规矩：用 `IsDebug` 1.4 插件去掉 `Ollydbg` 的调试器标志。载入后弹出“是压缩代码——要继续进行分析吗？”，点“否”。

```
00401000     68 01B06300          push WebSynch.0063B001 //进入 OD 后停在这！
00401005     E8 01000000          call WebSynch.0040100B
```

`F9` 运行，程序会中断在异常处，`Shift+F9` 通过异常

```
00B905CC     3100                 xor dword ptr ds:[eax],eax //第 27 次异常
00B905CE     64:8F05 00000000     pop dword ptr fs:[0]
00B905D5     58                   pop eax
00B905D6     833D DC49B900 00     cmp dword ptr ds:[B949DC],0
00B905DD     74 14                je short 00B905F3
00B905DF     6A 0C                push 0C
00B905E1     B9 DC49B900          mov ecx,0B949DC
00B905E6     8D45 F8              lea eax,dword ptr ss:[ebp-8]
00B905E9     BA 04000000          mov edx,4
00B905EE     E8 09C3FFFF          call 00B8C8FC
00B905F3     FF75 FC              push dword ptr ss:[ebp-4]
00B905F6     FF75 F8              push dword ptr ss:[ebp-8]
00B905F9     8B45 F4              mov eax,dword ptr ss:[ebp-C]
00B905FC     8338 00              cmp dword ptr ds:[eax],0
00B905FF     74 02                je short 00B90603
00B90601     FF30                 push dword ptr ds:[eax]
00B90603     FF75 F0              push dword ptr ss:[ebp-10]
00B90606     FF75 EC              push dword ptr ss:[ebp-14]
00B90609     C3                   retn //直接在这下断！ Shift+F9 断在这！返回到 00BA33F0
```

```
00BA33F0     E9 E0070000          jmp 00BA3BD5
```

如果 `F7` 跟踪下去会兜个圈子，在 `00BA3F70` 处返回 `00BA33F5` ★ ★

```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
00BA3BD5     66:81D0 E8C4         adc ax,0C4E8
00BA3BDA     80D3 47              adc bl,47
00BA3BDD     BE 780A3C24          mov esi,243C0A78
00BA3BE2     81C6 5009DD80        add esi,80DD0950
00BA3BE8     C1CE 6D              ror esi,6D
00BA3BEB     81C6 262783F6        add esi,F6832726
00BA3BF1     8BEE                 mov ebp,esi
00BA3BF3     81ED B9D3B2B1        sub ebp,B1B2D3B9
00BA3BF9     66:C1CF 23           ror di,23
00BA3BFD     0AC5                 or al,ch
  …… …… 省 略  …… ……
00BA3EDE     BF DA758539          mov edi,398575DA
00BA3EE3     81C7 F7CA338F        add edi,8F33CAF7
00BA3EE9     F7D7                 not edi
00BA3EEB     F7D7                 not edi
00BA3EED     8BEF                 mov ebp,edi
00BA3EEF     81F5 E140B9C8        xor ebp,C8B940E1
00BA3EF5     8BFD                 mov edi,ebp
00BA3EF7     03F9                 add edi,ecx
00BA3EF9     81EF D4FCFFFF        sub edi,-32C
00BA3EFF     FFE7                 jmp edi
00BA3F70     FFE5                 jmp ebp      //跳回 00BA33F5
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
```

```
00BA33F0     E9 E0070000          jmp 00BA3BD5 //所以在此处可以直接 F4 到下 1 行
00BA33F5     E1 85                loopde short 00BA337C
00BA33F5     66:8BC3              mov ax,bx //F4 下来后的变化！ 省了不少步  ★ ★
00BA33F8     E8 00000000          call 00BA33FD
00BA34DD     0F85 17000000        jnz 00BA34FA
00BA34E3     B0 7F                mov al,7F
00BA34E5     E9 23000000          jmp 00BA350D //此处下断，F9，断在这！ ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg) 跳出循环！
00BA34FA     BE 47854936          mov esi,36498547
00BA34FF     E9 9EFFFFFF          jmp 00BA34A2 //循环
00BA3681     0F85 18000000        jnz 00BA369F
00BA3687     66:B9 057E           mov cx,7E05
00BA368B     E9 25000000          jmp 00BA36B5 //此处下断，F9，断在这！ ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg) 跳出循环！
00BA369F     66:81D8 D644         sbb ax,44D6
00BA36A4     E9 7BFFFFFF          jmp 00BA3624 //循环
00BA374A     0F85 15000000        jnz 00BA3765
00BA3750     0FB7FA               movzx edi,dx
00BA3753     E9 22000000          jmp 00BA377A //此处下断，F9，断在这！ ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg) 跳出循环！
00BA3765     B9 0177552D          mov ecx,2D557701
00BA376A     E9 7CFFFFFF          jmp 00BA36EB //循环
00BA39D3     83EA 04              sub edx,4
00BA39D6     81FA 24FEFFFF        cmp edx,-1DC
00BA39DC     0F85 7EFFFFFF        jnz 00BA3960 //F4 下去
00BA39E2     E9 10000000          jmp 00BA39F7
  …… …… 省 略  …… ……
00BA3BC4     5B                   pop ebx
00BA3BC5     58                   pop eax
00BA3BC6     05 B0CBF57E          add eax,7EF5CBB0 //EAX=0019819A
00BA3BCB     5C                   pop esp
00BA3BCC     03C3                 add eax,ebx
                               //EAX=0019819A + 00400000=0059819A  这就是 OEP 值  ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)
00BA3BCE     894424 1C            mov dword ptr ss:[esp+1C],eax
00BA3BD2     61                   popad
00BA3BD3     FFE0                 jmp eax   //飞向光明之巅！![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)  跳至 0059819A
```

```
0059819A     55                   push ebp //在这儿用 LordPE 纠正 ImageSize 后完全 DUMP 这个进程
0059819B     8BEC                 mov ebp,esp
0059819D     6A FF                push -1
0059819F     68 C8AC5B00          push WebSynch.005BACC8
005981A4     68 20A75900          push WebSynch.0059A720
005981A9     64:A1 00000000       mov eax,dword ptr fs:[0]
005981AF     50                   push eax
005981B0     64:8925 00000000     mov dword ptr fs:[0],esp
005981B7     83EC 58              sub esp,58
005981BA     53                   push ebx
005981BB     56                   push esi
005981BC     57                   push edi
005981BD     8965 E8              mov dword ptr ss:[ebp-18],esp
005981C0     FF15 18725B00        call dword ptr ds:[5B7218]
```

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `0019819A`，点 `IT AutoSearch`，点“`Get Import`”，用“`追踪层次 1`”和“`ASProtect 1.2x`”插件全部修复无效函数。`FixDump`，正常运行！删除 `.adata` 区段，用 `LordPE` 重建 `PE`，`2.28M`->`774K`

## 二、完美爆破

经过层层跟踪，终于去掉了启动时的 `NAG`! ![icon\_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg) 其实可以下断 `BP DialogBoxParamA`

```
77D350B3     55                   push ebp //断在这里！看堆栈
77D350B4     8BEC                 mov ebp,esp
77D350B6     53                   push ebx
77D350B7     56                   push esi
77D350B8     8B75 08              mov esi,dword ptr ss:[ebp+8]
```

```
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
001292A4   00470A3A  /CALL 到 DialogBoxParamA 来自 dumped\_.00470A34 //就是这里出来 NAG 啦！
001292A8   00400000  |hInst = 00400000
001292AC   00000129  |pTemplate = 129
001292B0   00000000  |hOwner = NULL
001292B4   0040F620  |DlgProc = dumped\_.0040F620
001292B8   00000000  lParam = NULL
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
```

```
004709EF     53                 push ebx
004709F0     56                 push esi
004709F1     33DB               xor ebx,ebx
004709F3     57                 push edi
004709F4     895C24 14          mov dword ptr ss:[esp+14],ebx
004709F8     E8 437B0B00        call dumped\_.00528540
004709FD     85C0               test eax,eax
004709FF     75 6E              jnz short dumped\_.00470A6F
```



00470A01 FF15 C4755B00 call dword ptr ds:[<&user32.GetActiveWindow>]

00470A07 8BF0 mov esi,eax

00470A09 A1 D4C85F00 mov eax,dword ptr ds:[5FC8D4]

00470A0E 68 78C85F00 push dumped_.005FC878

00470A13 68 80C85F00 push dumped_.005FC880

00470A18 50 push eax

00470A19 E8 720D0000 call dumped_.00471790

00470A1E 8B0D D4C85F00 mov ecx,dword ptr ds:[5FC8D4]

00470A24 53 push ebx

00470A25 68 20F64000 push dumped_.0040F620

00470A2A 56 push esi

00470A2B 8B41 08 mov eax,dword ptr ds:[ecx+8]

00470A2E 68 29010000 push 129

00470A33 50 push eax

00470A34 FF15 C8755B00 call dword ptr ds:[<&user32.DialogBoxParamA>]//NAG，可以从上面跳过

00470A3A 3D 170C0000 cmp eax,0C17

00470A3F 75 0E jnz short dumped_.00470A4F

00528955 E8 16D7EDFF call dumped_.00406070

0052895A 8BC6 mov eax,esi//改为：mov al,1 就 OK 啦 ![icon_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)

0052895C 5F pop edi

0052895D 5E pop esi

0052895E 5D pop ebp

0052895F 5B pop ebx

00528960 8B4C24 3C mov ecx,dword ptr ss:[esp+3C]

00528964 64:890D 00000000 mov dword ptr fs:[0],ecx

0052896B 83C4 48 add esp,48

0052896E C3 retn

0052896F 8B4C24 40 mov ecx,dword ptr ss:[esp+40]

00528973 B8 01000000 mov eax,1//已注册标志 ![icon_smile.gif](img/59da1937da424a7e6d4210c5b355f670.jpg)

00528978 5B pop ebx

00528979 64:890D 00000000 mov dword ptr fs:[0],ecx

00528980 83C4 48 add esp,48

00528983 C3 retn

————————————————————————

1、完美爆破：注册版本，当然没有 NAG 啦

`0052895A 8BC6 mov eax,esi//改为：mov al,1`

2、手动填入注册信息：

`[HKEY_LOCAL_MACHINESOFTWAREReGet SoftwareRegistration]`

`"WebSynchronizerName"="fly [FCG]" //你的名字`

`"WebSynchronizerCode"="13572468" //随意`

`"WebSynchronizerActCode"="90123456" //随意`

—————————————————————————————————

```
          ,     _/
        /| _.-~/            _     ,        青春都一饷
       ( /~   /            ~-. _ |
       `\\  _/                ~ )          忍把浮名
   _-~~~-.)  )__/;;,.          _  //'
  /'_,   --~    ~~~-  ,;;____(  .-~~~-.        换了破解轻狂
 `~ _( ,_.-- (      ,;'' /    ~--   /._`
  /~~//'   / `~         ) /--.._, _)  `~
  "  `~"   "      `"      /~'`    \`\\~~
                          "      "   "~'  ""
        Cracked By 巢水工作坊——fly [OCN][FCG][NUKE]
                  2003-12-22  16:16
```

`4dd466d0e1e0cc123afbe504da2462ca`
