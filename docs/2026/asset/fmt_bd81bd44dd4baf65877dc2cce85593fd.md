
# 阿达连连看 3.80再脱再爆

- **【破文标题】** 阿达连连看 3.80再脱再爆
- **【破文作者】** 二哥weiyi75[Dfcg]
- **【作者邮箱】** weiyi75@sohu.com
- **【作者主页】** Dfcg官方大本营+龙族联盟论坛
- **【使用工具】** `peid`, `UnkillOD`
- **【破解平台】** Win2000/XP
- **【软件名称】** 阿达连连看 3.80
- **【下载地址】** `http://www.chinadfcg.com/viewthread.php?tid=11729`
- **【软件简介】**
    漂亮的办公小姐在电脑前埋头苦干的时候，十之八九，她在玩阿达连连看！ 这是一款很耐玩的桌面游戏，时下办公一族的新宠。

    一、游戏中包含了数套图案关卡，可以测试玩家眼明手快及逻辑判断能力。

    二、包含关卡有可爱的动物、神奇宝贝、星座传说、麻将、甜点饼干、电脑系统、街头霸王、机器猫。

    三、绿色软件，不在系统中留下任何垃圾。

    四、华丽的画面、动人的音效，令人欲罢不能。

- **【软件大小】** 4.66M
- **【破解目的】** 为我的破解之路铺一块小石头。
- **【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 破解内容

上次二哥的阿达连连看 3.58先脱后爆没有脱衣爆破，随着这个软件流程的熟悉，先脱后爆更容易。

首先 `Peid` 查壳，`Nothing found \*!` 再看 EP 区段 `.perplex`，原来是 Acprotect1.X 版加的壳。

因为是 VB 程序，Stolen Code 很简单，没有必要苦苦跟踪，VB 的 IAT 加密目前大部分加密软件都是有心无力的。

`OD` 异常设置不忽略内存异常，全自动隐藏 `OD` 插件帮你隐藏住 `OD`，载入程序。

### 初跟踪 Stolen Code

```
00451000 a>  60                              pushad //外壳入口,F9运行。
00451001     66:81C0 EE8F                    add ax,8FEE
00451006     66:13EF                         adc bp,di
00451009     FC                              cld
0045100A     85C8                            test eax,ecx
0045100C     72 03                           jb short adalinks.00451011
0045100E     73 01                           jnb short adalinks.00451011
00451010   - 72 D3                           jb short adalinks.00450FE5
00451012     DDB9 59104500                   fstsw word ptr ds:[ecx+451059]
00451018     68 E860726B                     push 6B7260E8
0045101D     66:81DF BA66                    sbb di,66BA
00451022     58                              pop eax
00451023     BA 1A2B66B2                     mov edx,B2662B1A
00451028     0F81 04000000                   jno adalinks.00451032
0045102E     66:BD 002E                      mov bp,2E00
00451032     81EA 092B66B2                   sub edx,B2662B09
00451038     8B19                            mov ebx,dword ptr ds:[ecx]
..............................................................................
0045FAAD     CD 01                           int 1   //最后一次异常。
0045FAAF     40                              inc eax
0045FAB0     40                              inc eax
0045FAB1     0BC0                            or eax,eax
0045FAB3     75 05                           jnz short adalinks.0045FABA
0045FAB5     90                              nop
0045FAB6     90                              nop
0045FAB7     90                              nop
0045FAB8     90                              nop
0045FAB9     61                              popad
0045FABA     33C0                            xor eax,eax
0045FABC     64:8F00                         pop dword ptr fs:[eax]
0045FABF     58                              pop eax
0045FAC0     60                              pushad
0045FAC1     E8 00000000                     call adalinks.0045FAC6
0045FAC6     5E                              pop esi
0045FAC7     83EE 06                         sub esi,6
0045FACA     B9 57000000                     mov ecx,57
```

`Alt+M` 打开内存镜像

```
内存镜像，项目 21
 地址=00401000 //对准这里F2断点,Shift+F9飞向光明之巅
 大小=00048000 (294912.)
 Owner=adalinks 00400000
 区段=.text
 包含=code
 类型=Imag 01001002
 访问=R
 初始访问=RWE
```

```
004023F6   - FF25 40114000                   jmp dword ptr ds:[401140]                   ; MSVBVM60.EVENT\_SINK\_Release
004023FC   - FF25 E4114000                   jmp dword ptr ds:[4011E4]  ; MSVBVM60.ThunRTMain
```

Stolen Code2 Call `004023FC` 执行效果

```
00402402     0000                            add byte ptr ds:[eax],al
00402404     41                              inc ecx
00402405     D22D F009800C                   shr byte ptr ds:[C8009F0],cl
0040240B     BD 6C6E0000                     mov ebp,6E6C
00402410     48                              dec eax
00402411     0000                            add byte ptr ds:[eax],al
00402413     0030                            add byte ptr ds:[eax],dh
00402415     0000                            add byte ptr ds:[eax],al
00402417     0040 00                         add byte ptr ds:[eax],al
0040241A     0000                            add byte ptr ds:[eax],al
```

堆栈友好提示

```
0012FFBC    0046A1F8  返回到 adalinks.0046A1F8 来自 adalinks.004023FC
0012FFC0    004028C8  adalinks.004028C8 //Stolen Code1 Push 4028C8 执行效果
0012FFC4    77E614C7  返回到 kernel32.77E614C7
0012FFC8    0012CEA8
0012FFCC    004AA38C
0012FFD0    7FFDF000
```

根据五种语言入口熟悉和堆栈提示修复入口如下

```
004023F0   - FF25 F4104000                   jmp dword ptr ds:[<&msvbvm60.EVENT\_SINK\_Add>; msvbvm60.EVENT\_SINK\_AddRef
004023F6   - FF25 40114000                   jmp dword ptr ds:[<&msvbvm60.EVENT\_SINK\_Rel>; msvbvm60.EVENT\_SINK\_Release
004023FC   - FF25 E4114000                   jmp dword ptr ds:[<&msvbvm60.ThunRTMain>]   ; msvbvm60.ThunRTMain
00402402     0000                            add byte ptr ds:[eax],al //不能在这里处理，否则无法运行
00402404 U>  68 C8284000                     push Unpack\_.004028C8 //修复如下,用OD插件直接修正入口为2404，重建输入表方式1即可运行。
00402409     E8 EEFFFFFF                     call <jmp.&msvbvm60.ThunRTMain>
```

由于 Acprotect 更新 N 代，简单的 `Esp` 定律已经无法找到 Stolen Code。

Fly 大侠的话

> 在壳把所有的代码解压之后、处理 Stolen Code 之前，把进程 `Dump` 出来，补上那段壳代码，模仿构造当时的堆栈和寄存器值环境，这样就由壳自己来解决 Stolen Code 的问题啦。推而广之，这个 Stolen Code 简便解决方案也适用于某些其他壳。

由于 Acprotect 更新 N 代，这个版本已经很强悍了，入口校验我们后面分析。

借 VB 的软柿子了解如何能带发修行。

用 `push 004028C8` 入手

`ESP` 定律辅助

重起 `OD`

```
00451000 a>  60                              pushad //F8
00451001     66:81C0 EE8F                    add ax,8FEE  //hr esp ,F9运行
00451006     66:13EF                         adc bp,di
00451009     FC                              cld
0045100A     85C8                            test eax,ecx
0045100C     72 03                           jb short adalinks.00451011
0045100E     73 01                           jnb short adalinks.00451011
```

还是分对中断

```
004626FF     61                              popad  '1
00462700     893D 8F1E4500                   mov dword ptr ds:[451E8F],edi
00462750     60                              pushad '2
00462751     E8 0ABDFFFF                     call adalinks.0045E460
0046276E     61                              popad
0046276F     8B1C24                          mov ebx,dword ptr ss:[esp]
004627BF     60                              pushad '3
004627C0     E8 9BBCFFFF                     call adalinks.0045E460
004627D0     61                              popad
004627D1     8F05 F31E4500                   pop dword ptr ds:[451EF3]
00462821     60                              pushad '4
00462822     E8 39BCFFFF                     call adalinks.0045E460
0046282E     61                              popad
0046282F     BA EF1E4500                     mov edx,adalinks.00451EEF
0046287F     60                              pushad '5
00462880     E8 15DEFFFF                     call adalinks.0046069A
00462885     61                              popad
00462886     8B3C24                          mov edi,dword ptr ss:[esp]
004628D6     60                              pushad '6
004628D7     E8 5BDBFFFF                     call adalinks.00460437
004628DC     61                              popad
004628DD     8B35 A71E4500                   mov esi,dword ptr ds:[451EA7]
0046292D     60                              pushad '7
0046292E     E8 86FDFFFF                     call adalinks.004626B9
00462933     61                              popad
00462934     891D 971E4500                   mov dword ptr ds:[451E97],ebx '到这里滚动条往下,不注意下面的Stolen Code变形位置根本发现不了，这里代发修行，重建输入表方式1即可运行，代码资源已经解压。
0046293A     FF35 971E4500                   push dword ptr ds:[451E97]
00462940     57                              push edi
00462941     BF E31E4500                     mov edi,adalinks.00451EE3
00462946     8BDF                            mov ebx,edi
00462948     5F                              pop edi
```

