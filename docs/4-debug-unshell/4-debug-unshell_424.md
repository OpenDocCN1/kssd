
# Asterisk Password Recovery XP

> 原文：[`www.pediy.com/kssd/pediy06/pediy6223.htm`](https://www.pediy.com/kssd/pediy06/pediy6223.htm)

## ASProtect V1.2 脱壳——Asterisk Password Recovery XP

*   下载页面：`http://www.fadsoft.com/`
*   软件大小：`368 KB`
*   软件语言：`英文`
*   软件类别：`共享版 / 密码恢复`
*   应用平台：`Win9x/NT/2000/XP`
*   加入时间：`2002-07-11`
*   下载次数：`18214`
*   推荐等级：`***`

**【软件简介】**：星号密码克星用来读取隐藏在星号 `*****` 后面的密码，同时可以读取 `IE` 中的星号密码。

**【软件限制】**：`30` 天试用、功能限制

**【作者声明】**：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

**【破解工具】**：`Ollydbg1.09`、`FI`、`LordPE`、`ImportREC`、`RegMon`

---

### 【过 程】

手里拿到的这个版本是以前某个朋友上传的，没有写版本号。天空上的是 `V 1.89` 版，主程序直接用 `ASProtect v1.2` 加壳，比我手里的版本少了一些特殊的东西呀，或许下面所说的 `UPX + RarSfx` 并不是作者做的。 ^O^

`apr189_portable.exe` 用 `FI` 看是“`UPX + RarSfx`” 晕，没见过这种混合型的东东。 :-<

后来按 `UPX` 的方法脱壳后发现…… :-) 原来 `apr189_portable.exe` 是用 `WinRAR` 高级自解压 方式打包成的 `SFX` 自解压文件！然后再用 `UPX` 压缩了这个 `SFX` 文件！原来 `FI` 显示 `UPX + RarSfx` 是这个原因呀。

双击 `apr189_portable.exe`，程序会在 `C:WINDOWSTEMP` 下解压成 `RarSFX0` 文件夹，程序退出后会自动删除这个文件夹。早知道 `UPX` 也就没必要脱了。好了，在调试时把 `RarSFX0` 复制出来，里面有 `5` 个文件：`AsteriskPR.exe`、`apr.ani`、`gotit.wav`、`help.htm`、`mousehook.dll`。真正的主程序是 `AsteriskPR.exe`！用 `FI` 看是 `ASProtect V1.2` 壳。呵呵，真是用了不少心思呀！:-) 这个东东可以用 `CasprGui` 自动脱壳，比较完美啦，比我手脱的要小不少。

呵呵，`PEID 0.8` 也能找到这个东东的 `OEP`，:-) 试试用 `Ollydbg` 手动脱壳吧，虽然这个版本的 `ASProtect` 已被大侠们脱过 `N` 次了，但毕竟自己做一次才能更好的学习呀。 :-D

---

### 一、脱壳

```asm
00401000       68 01C04000        push ASTERISK.0040C001
                              ==== >进入 OD 后断在这！

00401005       C3                retn
```

`F9` 运行，程序会在异常处中断。

```asm
00DB009D     3100              xor dword ptr ds:[eax],eax
                          ==== >第一个异常
00DB009F     EB 01             jmp short 00DB00A2
```

`Shift+F9` 通过异常，`20` 次程序运行。好了，`Try Again`，按 `19` 次 `Shift+F9`，停下来。

```asm
00DC24E1     60                pushad
00DC24E2     A3 29B12F03       mov dword ptr ds:[32FB129],eax
00DC24E7     C490 60E80600     les edx,fword ptr ds:[eax+6E860]
00DC24ED     0000              add byte ptr ds:[eax],al
00DC24EF     8B6424 08         mov esp,dword ptr ss:[esp+8]
                          ==== >堆栈区的第二条地址        下断点！
00DC24F3     EB 0C             jmp short 00DC2501
00DC24F5     33F6              xor esi,esi
00DC24F7     64:FF36           push dword ptr fs:[esi]
00DC24FA     64:8926           mov dword ptr fs:[esi],esp
00DC24FD     FF06              inc dword ptr ds:[esi]
                          ==== >第 19 次异常在这儿！        :-)
                          ==== >看看堆栈区的第二条地址是：00DC24EF  F2 设断
00DC24FF   ^ EB E8             jmp short 00DC24E9
00DC2501     05 ACA78DB2       add eax,B28DA7AC
00DC2506     E8 09000000       call 00DC2514
                          ==== >变形 JMP！F7 走进
00DC250B     23C5              and eax,ebp
00DC250D     48                dec eax
00DC250E     E9 05000000       jmp 00DC2518
                          ==== >最后从这跳下去！
00DC2513     90                nop
00DC2514     C1E0 F7           shl eax,0F7
00DC2517     C3                retn
                          ==== >返回 00DC250B
```

在 `00DC24EF` 处设断后，`Shift+F9` 运行，程序会中断在 `00DC24EF`

`F7` 走，会从 `00DC2517` 处返回 `00DC250B`，再跳到 `00DC2518` 处！

```asm
00DC2518     2BC9              sub ecx,ecx
00DC251A     64:8F01           pop dword ptr fs:[ecx] ; 00DC2870
00DC251D     59                pop ecx
00DC251E     E8 00000000       call 00DC2523
00DC2523     98                cwde
00DC2524     8B2C24            mov ebp,dword ptr ss:[esp]
00DC2527     58                pop eax
00DC2528     81ED AB28BD00     sub ebp,0BD28AB
00DC252E     1BC6              sbb eax,esi
00DC2530     B9 F1AD8DB2       mov ecx,B28DADF1
00DC2535     81F1 A78530B2     xor ecx,B23085A7
00DC253B     33C6              xor eax,esi
00DC253D     03CD              add ecx,ebp
00DC253F     B8 15B08DB2       mov eax,B28DB015
00DC2544     8BF8              mov edi,eax
00DC2546     81EF 0FB08DB2     sub edi,B28DB00F
00DC254C     23C2              and eax,edx
00DC254E     BB ADA38DB2       mov ebx,B28DA3AD
00DC2553     C1C0 BA           rol eax,0BA
00DC2556     3119              xor dword ptr ds:[ecx],ebx
00DC2558     83EF 01           sub edi,1
00DC255B     8BC3              mov eax,ebx
00DC255D     05 D1A38DB2       add eax,B28DA3D1
00DC2562     93                xchg eax,ebx
00DC2563     F8                clc
00DC2564     41                inc ecx
00DC2565     41                inc ecx
00DC2566     41                inc ecx
00DC2567     41                inc ecx
00DC2568     B8 00000000       mov eax,0
00DC256D     48                dec eax
00DC256E     03C7              add eax,edi
00DC2570   ^ 79 E4             jns short 00DC2556
                          ==== >F4 下去
00DC2572     40                inc eax
00DC2573     61                popad
00DC2574     0BC5              or eax,ebp
00DC2576     48                dec eax
00DC2577     C3                retn
                          ==== >返回到 00DC24CE
```

```asm
00DC24CE     5B                pop ebx         ; ASTERISK.00400000
00DC24CF     58                pop eax
00DC24D0     05 AA2CE3A4       add eax,A4E32CAA
00DC24D5     5C                pop esp
00DC24D6     0BC9              or ecx,ecx
00DC24D8   ^ 74 E3             je short 00DC24BD
00DC24DA     8901              mov dword ptr ds:[ecx],eax
00DC24DC     03C3              add eax,ebx
                          ==== >EAX=000045A6 + 00400000=004045A6   这就是 OEP 值  :-)
00DC24DE     894424 1C         mov dword ptr ss:[esp+1C],eax
00DC24E2     61                popad
00DC24E3     FFE0              jmp eax
                          ==== >跳向 OEP
```

---

```asm
004045A6       55              db 55
                          ==== >在这儿用 LordPE 完全 DUMP 这个进程
004045A7       8B              db 8B
004045A8       EC              db EC
004045A9       6A              db 6A
```

---

重新运行程序，运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000045A6`，点 `IT AutoSearch`，点“`Get Import`”，有一个函数手动修复之后无效，用 `ASProtect 1.22` 的插件追踪也不行： :-<

```text
FThunk: 00005328  NbFunc: 00000001
0  00005328  ?  0000  00401000
```

索性就这样 `FixDump`，却正常运行！:-) `73K ->104K`

---

### 二、关于破解

程序的 `30` 天试用是用 `ASProtect` 做的限制，脱完壳后就取消试用期了。

或者把注册表中的：

```text
[HKEY_CLASSES_ROOTCLSID{9F0BB62D-07DF-6011-4495-DE6C8C1F2D37}]
@="8A31RyvX"
```

删除。也可以把 `8A31RyvX` 改成：`JvZwZi8fBQjZeL/K`

调试的时候在内存中发现这段代码： :-) 不清楚这是哪个作者留的 :-P

```text
☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

00DC28A0  50 6C 65 61 73 65 20 64 6F 6E 74 20 63 72 61 63  Please dont crac
00DC28B0  6B 20 69 74 20 74 68 61 6E 6B 20 79 6F 75 20 69  k it thank you i
00DC28C0  66 20 79 6F 75 20 6E 65 65 64 20 61 20 72 65 67  f you need a reg
00DC28D0  69 73 74 65 72 20 63 6F 70 79 20 70 6C 65 61 73  ister copy pleas
00DC28E0  65 20 6C 65 74 20 6D 65 20 6B 6E 6F 77 00 78 52  e let me know

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆
```

这个东东保护的不错，或许是不完全版。看了一会儿没能找到好的突破口，算了吧，该休息了。:-<

`BTW`：某些程序反 `RegMon`、`FileMon`，这个东东就 `KILL` 掉 `RegMon`，我们可以对程序动手术，但这样碰到一个就要改一次，也不好找到关键的地方；所以我们可以把自己的 `RegMon`、`FileMon` 修改一下，让程序认不出来，呵呵。 :-)

---

```text
     ,     _/ 
    /| _.-~/         _     ,        青春都一饷
   ( /~   /          ~-.| 
   `\\  _/             ~ )          忍把浮名 
  _-~~~-.)  )_;/;;,.         _  //'    
 /'\_,   --~    ~~~-  ,;;\_\_(  (.-~~~-.        换了破解轻狂 
\`~ _( ,_.-- (     ,;'' /    ~--   /.\_\`
```



/~~//' /`~           ) /--..\_, )\_  \`~

  "`~"  "        \`"        /~'\`    \`\\~~

                         "       "   "~'  ""

                    Cracked By 巢水工作坊——fly \[OCN\]\[FCG\]

                         2003-09-16  22:40

`4dd466d0e1e0cc123afbe504da2462ca`
