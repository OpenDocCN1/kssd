
# 小游戏-VM 测试

> 原文：[`www.pediy.com/kssd/pediy10/82948.html`](https://www.pediy.com/kssd/pediy10/82948.html)

发个 vm，喜欢门电路，喜欢卡诺图，状态机的可以玩下。高手飘过。

![](img/8ba846cc0bb1af8f99c968b5531d60b1.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `RM090228_by_ShellWolf.rar` |

先发一组

`sessiondiy`

`123456780CEE7918`

【文章标题】: `VMCraCk` 的双重爆破&&`VM` 简要分析

【文章作者】: `eASYVMBOOm`

【作者主页】: `http://blog.sina.com.cn/77muyulong`

【软件名称】: `RM090228_by_ShellWolf.exe`

【下载地址】: `http://bbs.pediy.com/showthread.php?t=82948`

【加壳方式】: 无

【保护方式】: `NoProtect`

【使用工具】: `OD` `winHEX`

【软件介绍】: `ShellWolf` 老大制作的一个 `CM`

【作者声明】: 并没能按照老大的要求弄出算法 仅作学习笔记...

------------------------------------------------------------------------

【详细过程】

最近工作比较忙 好久都顾不上写点东西了 今天偶得 `ShellWolf` 老大制作的一个 `VMCraCk` 爱不释手 奈何水平一般 仅做了部分分析 于是成文 仅为一学习笔记耳...

载入程序 搜索字符参考 找到

文本字符串参考位于 `RM090228:.text`，项目 11

地址=`0040188C`

反汇编=`push RM090228.00419680`

文本字符=`ASCII "Success!"`

跟过去看到了许多字符 这就是验证的核心了

代码

```
00401760    .  55                push ebp                            ; msgbox
00401761    .  8BEC              mov ebp,esp
00401763    .  6A FF             push -1
00401765    .  68 A0964100       push RM090228.004196A0
0040176A    .  68 A0424000       push RM090228.004042A0              ; SE handler installation
0040176F    .  64:A1 00000000    mov eax,dword ptr fs:[0]
00401775    .  50                push eax
00401776    .  64:8925 00000000  mov dword ptr fs:[0],esp
0040177D    .  81EC 24030000     sub esp,324
00401783    .  A1 F8FC4100       mov eax,dword ptr ds:[41FCF8]
00401788    .  8945 E4          mov dword ptr ss:[ebp-1C],eax
0040178B    .  53                push ebx
0040178C    .  56                push esi
0040178D    .  57                push edi
0040178E    .  8965 E8          mov dword ptr ss:[ebp-18],esp
00401791    .  8BF1             mov esi,ecx
00401793    .  89B5 D0FCFFFF    mov dword ptr ss:[ebp-330],esi
```

该函数用来读取注册名和验证吗

```
00401799    .  6A 0F             push 0F                               ; /Arg3 = 0000000F
0040179B    .  8D45 D4          lea eax,dword ptr ss:[ebp-2C]          ; |
0040179E    .  50               push eax                               ; |Arg2
0040179F    .  68 EA030000      push 3EA                               ; |Arg1 = 000003EA
004017A4    .  E8 22250100      call <RM090228.GetText>                ; \\RM090228.00413CCB
00413CCB <>/$  55               push ebp                               ; GetText
00413CCC   |.  8BEC            mov ebp,esp
00413CCE   |.  8379 48 00      cmp dword ptr ds:[ecx+48],0
00413CD2   |.  75 16           jnz short RM090228.00413CEA
00413CD4   |.  FF75 10         push dword ptr ss:[ebp+10]              ; /Count
00413CD7   |.  FF75 0C         push dword ptr ss:[ebp+C]              ; |Buffer
00413CDA   |.  FF75 08         push dword ptr ss:[ebp+8]              ; |ControlID
00413CDD   |.  FF71 1C         push dword ptr ds:[ecx+1C]             ; |hWnd
00413CE0   |.  FF15 A0924100    call dword ptr ds:[<&USER32.GetDlgI>; \\GetDlgItemTextA
00413CE6   |.  5D             pop ebp
00413CE7   |.  C2 0C00         retn 0C
```

作用同上

```
004017A9    .  8AD8             mov bl,al
004017AB    .  889D CCFCFFFF    mov byte ptr ss:[ebp-334],bl
004017B1    .  6A 40             push 40                                 ; /Arg3 = 00000040
004017B3    .  8D8D D4FDFFFF     lea ecx,dword ptr ss:[ebp-22C]          ; |
004017B9    .  51               push ecx                                 ; |Arg2
004017BA    .  68 EB030000      push 3EB                                 ; |Arg1 = 000003EB
004017BF    .  8BCE             mov ecx,esi                              ; |
004017C1    .  E8 05250100      call <RM090228.GetText>                ; \\RM090228.00413CCB
00413CCB <>/$  55               push ebp                               ; GetText
00413CCC   |.  8BEC            mov ebp,esp
00413CCE   |.  8379 48 00      cmp dword ptr ds:[ecx+48],0
00413CD2   |.  75 16           jnz short RM090228.00413CEA
00413CD4   |.  FF75 10         push dword ptr ss:[ebp+10]              ; /Count
00413CD7   |.  FF75 0C         push dword ptr ss:[ebp+C]              ; |Buffer
00413CDA   |.  FF75 08         push dword ptr ss:[ebp+8]              ; |ControlID
00413CDD   |.  FF71 1C         push dword ptr ds:[ecx+1C]             ; |hWnd
00413CE0   |.  FF15 A0924100    call dword ptr ds:[<&USER32.GetDlgI>; \\GetDlgItemTextA
00413CE6   |.  5D             pop ebp
00413CE7   |.  C2 0C00         retn 0C
```

以下为判断验证码合法性的代码 长度必须要为 `0X10`...

```
004017C6    .  80FB 04          cmp bl,4                                ; bl<0x4 跳
004017C9    .  0F82 14010000    jb RM090228.004018E3
004017CF    .  80FB 0F          cmp bl,0F                               ; bl>0xf 跳
004017D2    .  0F87 0B010000    ja RM090228.004018E3
004017D8    .  3C 10            cmp al,10                               ; 长度为 0x10
004017DA    .  0F85 03010000    jnz RM090228.004018E3
```

先说主程序爆破 看代码

```
0040187A    .  E8 91FDFFFF      call <RM090228.VMFile>
0040187F    .  83C4 10          add esp,10
00401882    .  6A 00            push 0
00401884    .  84C0             test al,al
00401886       74 0B           je short RM090228.00401893
00401888    .  8D4D D4         lea ecx,dword ptr ss:[ebp-2C]
0040188B    .  51               push ecx
0040188C    .  68 80964100      push RM090228.00419680                 ; ASCII "Success!"
00401891    .  EB 18            jmp short RM090228.004018AB
00401893    >  8D55 D4          lea edx,dword ptr ss:[ebp-2C]
00401896    .  52               push edx
00401897    .  EB 0D            jmp short RM090228.004018A6
00401899    .  B8 01000000      mov eax,1
0040189E    .  C3               retn
0040189F    .  8B65 E8          mov esp,dword ptr ss:[ebp-18]
004018A2    .  6A 00            push 0
004018A4    .  6A 00            push 0
004018A6    >  68 78964100      push RM090228.00419678                 ; ASCII "Fail!"
004018AB    >  8B8D D0FCFFFF    mov ecx,dword ptr ss:[ebp-330]
004018B1    .  E8 78FD0000      call RM090228.0041162E
004018B6    .  C745 FC FFFFFFFF mov dword ptr ss:[ebp-4],-1
004018BD    .  68 6C964100      push RM090228.0041966C                 ; ASCII "Register"
004018C2    .  6A 01            push 1
004018C4    .  8BB5 D0FCFFFF    mov esi,dword ptr ss:[ebp-330]
004018CA    .  8BCE             mov ecx,esi
004018CC    .  E8 22240100      call RM090228.00413CF3
004018D1    .  6A 01            push 1
004018D3    .  6A 01            push 1
004018D5    .  8BCE             mov ecx,esi
004018D7    .  E8 CB230100      call RM090228.00413CA7
004018DC    .  8BC8             mov ecx,eax
004018DE    .  E8 D8240100      call RM090228.00413DBB
004018E3    >  8B4D F0          mov ecx,dword ptr ss:[ebp-10]
004018E6    .  64:890D 00000000 mov dword ptr fs:[0],ecx
004018ED    .  8B4D E4          mov ecx,dword ptr ss:[ebp-1C]
004018F0    .  E8 60290000      call RM090228.00404255
004018F5    .  5F               pop edi
004018F6    .  5E               pop esi
004018F7    .  5B               pop ebx
004018F8    .  8BE5             mov esp,ebp
004018FA    .  5D               pop ebp
004018FB    .  C3               retn
```

主程序爆破十分简单 只要把

```
00401886       74 0B           je short RM090228.00401893
```

`nop` 掉就可以了 输入任何 `0x10` 位的验证码 都可以通过...

接着深入 在判断之前看一下那个 `call` 跟进

该函数用来 `load` `vmdata.dat` 走啊走 就到了

```
0040170C   |.  52               push edx
```



# 0040170D Analysis

`push RM090228.00419660 ; ASCII "vmdata.dat"`

`call RM090228.00401100`

`call <RM090228.VMStart>`

这个 `call` 跟进就到了 `VMengine`

接下来是完整的 `VMengine` 后面的注释是我的简要分析 由于时间因素 只分析了其中一个 `proc`

`VM` 的初始化 这个引擎没有初始化虚拟机环境 直接就是读取字节码执行了...

## VM Initialization and Bytecode Execution

`00401210 <> $ 55 push ebp ; VMstart`

`00401211 . 8BEC mov ebp,esp`

`00401213 . 83EC 2C sub esp,2C ; enter 分配局部变量`

`00401216 . A1 F8FC4100 mov eax,dword ptr ds:[41FCF8]`

`0040121B . 8945 F8 mov dword ptr ss:[ebp-8],eax`

`0040121E . 56 push esi`

`0040121F . A1 580C4200 mov eax,dword ptr ds:[420C58]`

`00401224 . 8B0D 480C4200 mov ecx,dword ptr ds:[420C48]`

`0040122A . 0308 add ecx,dword ptr ds:[eax]`

`0040122C . 890D 440C4200 mov dword ptr ds:[420C44],ecx`

`00401232 > 8B15 440C4200 mov edx,dword ptr ds:[420C44] ; 内存指针所指变量置寄存器 edx`

`00401238 . 0FB602 movzx eax,byte ptr ds:[edx] ; 拓展复制 edx 指向的字节码`

`0040123B . 3D FF000000 cmp eax,0FF ; 比较字节码是否==0xFF`

`00401240 . 0F84 77030000 je RM090228.004015BD ; 是则跳=>VM_RETN`

`00401246 . 8B0D 440C4200 mov ecx,dword ptr ds:[420C44] ; 内存指针所指变量置寄存器 ecx`

`0040124C . 8A11 mov dl,byte ptr ds:[ecx] ; 字节码置 dl`

`0040124E . 8855 E7 mov byte ptr ss:[ebp-19],dl ; 字节码置 VM_CODE`

`00401251 . 0FB645 E7 movzx eax,byte ptr ss:[ebp-19] ; VM_CODE 拓展置 eax`

`00401255 . 8945 D4 mov dword ptr ss:[ebp-2C],eax ; VM_CODE 置局部变量 VM_CODE2??`

`00401258 . 837D D4 05 cmp dword ptr ss:[ebp-2C],5 ; VM_CODE>0x5??`

`0040125C . 0F87 37030000 ja RM090228.00401599 ; Y==>jmp`

`00401262 . 8B4D D4 mov ecx,dword ptr ss:[ebp-2C] ; VM_CODE 置 ecx`

`00401265 . FF248D CA154000 jmp dword ptr ds:[ecx*4+4015CA] ; VMJMPTOPROC 基址 4015ca 共 6 个`

读取字节码 参与运算 计算出 `procJMPaddress`

可以看出 `procJMPbase` 是 `4015CA`

内存里看一下

`004015CA 0040126C RM090228.0040126C`
`004015CE 00401310 RM090228.00401310`
`004015D2 00401351 RM090228.00401351`
`004015D6 004013F8 RM090228.004013F8`
`004015DA 0040146E RM090228.0040146E`
`004015DE 004014E4 RM090228.004014E4`

总共 6 个过程 加上在这个之外的 `0XFF` `VM_RETN` 一共是 7 个

第一次读取的字节码是 `0X01` 对应的过程为 `RM090228.00401310`

跳过去...

## Proc2: Constant to Memory

`00401310 > \\A1 440C4200 mov eax,dword ptr ds:[420C44] ; proc2 取 VMEIP`

`00401315 . 66:8B48 02 mov cx,word ptr ds:[eax+2] ; WORD[VMEIP+2]=>cx`

`00401319 . 66:894D EC mov word ptr ss:[ebp-14],cx ; 保存 cx 该 WORD`

`0040131D . 8B15 580C4200 mov edx,dword ptr ds:[420C58] ; \\`

`00401323 . 8B02 mov eax,dword ptr ds:[edx] ; |读取 DWORD`

`00401325 . 83C0 08 add eax,8 ; |==>DWODR[[00420c58]]+8`

`00401328 . 8B0D 580C4200 mov ecx,dword ptr ds:[420C58] ; |存回 DWORD`

`0040132E . 8901 mov dword ptr ds:[ecx],eax ; /`

`00401330 . 8B15 440C4200 mov edx,dword ptr ds:[420C44] ; 取 VMEIP`

`00401336 . 8B42 04 mov eax,dword ptr ds:[edx+4] ; DWORD[VMEIP+4]=>eax`

`00401339 . 8945 D8 mov dword ptr ss:[ebp-28],eax ; 保存 eax 该 DWORD`

`0040133C . 0FB74D EC movzx ecx,word ptr ss:[ebp-14] ; 拓展取 WORD 到 ecx`

`00401340 . 8B15 5C0C4200 mov edx,dword ptr ds:[420C5C] ; XXXXXXXX`

`00401346 . 8B45 D8 mov eax,dword ptr ss:[ebp-28] ; 取 DWORD 到 eax`

`00401349 . 89048A mov dword ptr ds:[edx+ecx*4],eax ; mov [[420c5c]+WORD*4],DWORD`

`0040134C . E9 53020000 jmp RM090228.004015A4`

`004015A4 > \\8B0D 580C4200 mov ecx,dword ptr ds:[420C58] ; \\`

`004015AA . 8B15 480C4200 mov edx,dword ptr ds:[420C48] ; |`

`004015B0 . 0311 add edx,dword ptr ds:[ecx] ; |`

`004015B2 . 8915 440C4200 mov dword ptr ds:[420C44],edx ; /修改 VMEIP`

`004015B8 .^ E9 75FCFFFF jmp RM090228.00401232`

以上就是一个完整的 `VM` 过程 在每条指令后面都有我家的详细注释

以上这条指令就相当于写常数到内存地址

原字节码为

`00F30040 01 00 61 03 DE 84 D7 08 .a?.`

还原后为

`mov [[420c5c]+361*4],08D784DE`

也就是

`01 00 YY YY XX XX XX XX`

```
;******************************
mov [[420c5c]+YYYY*4],XXXXXXXX
```

这样一步步分析 方可得到类似于源码的算法 由于本人过菜 加之时间有限 所以无法完成算法分析 故追随某牛 另辟蹊径...

## Validation Check

还是看验证上面的判断

`00401884 . 84C0 test al,al`

`00401886 74 0B je short RM090228.00401893`

此判断验证 `al` 是否为 `0` 如果为 `0` 则跳转 也就会出错 于是我们只要让 `VM` 返回个 `al!=0` 就可以了 如果我们把所有 `VM_code` 都搞清楚的话 这个应该不太难 这里偷了个懒 因为发现只要让 `VMengine` 直接返回 就可以通过验证 而返回的字节码我们已经知道是 `0XFF` 了 所以 就可以修改虚拟机的字节码 达到爆破的效果...

但是用 `WINHEX` 修改后却失败了 调试发现 如果只变这一个字节的话不能读出字节码 这个虚拟机的字解码是把 `vmdata.dat` 文件里数据抽出来并且经过本身数据的参与计算以后放到了内存里 修改这个字节会影响到后面运算的进行 所以会出现这样的错误 看来还是比较麻烦的...

## Checksum Calculation Code

```
00401FFC |> /3306 xor eax,dword ptr ds:[esi]
00401FFE |. |83C6 04 add esi,4
00402001 |. |BB FF000000 mov ebx,0FF
00402006 |. |21C3 and ebx,eax
00402008 |. |C1E8 08 shr eax,8
0040200B |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
0040200E |. |BB FF000000 mov ebx,0FF
00402013 |. |21C3 and ebx,eax
00402015 |. |C1E8 08 shr eax,8
00402018 |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
0040201B |. |BB FF000000 mov ebx,0FF
00402020 |. |21C3 and ebx,eax
00402022 |. |C1E8 08 shr eax,8
00402025 |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
00402028 |. |BB FF000000 mov ebx,0FF
0040202D |. |21C3 and ebx,eax
0040202F |. |C1E8 08 shr eax,8
00402032 |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
00402035 |. |3306 xor eax,dword ptr ds:[esi]
00402037 |. |83C6 04 add esi,4
0040203A |. |BB FF000000 mov ebx,0FF
0040203F |. |21C3 and ebx,eax
00402041 |. |C1E8 08 shr eax,8
00402044 |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
00402047 |. |BB FF000000 mov ebx,0FF
0040204C |. |21C3 and ebx,eax
0040204E |. |C1E8 08 shr eax,8
00402051 |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
00402054 |. |BB FF000000 mov ebx,0FF
00402059 |. |21C3 and ebx,eax
0040205B |. |C1E8 08 shr eax,8
0040205E |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
00402061 |. |BB FF000000 mov ebx,0FF
00402066 |. |21C3 and ebx,eax
00402068 |. |C1E8 08 shr eax,8
0040206B |. |33049F xor eax,dword ptr ds:[edi+ebx*4]
0040206E |. |49 dec ecx
0040206F |.^\\0F85 87FFFFFF \\jnz RM090228.00401FFC
```



应该说现在已经可以内存补丁了

至于是怎么做到 `DATAFile` 爆破的 等有时间在研究 或者青海风老师指点下.....

附上 `DATAFile` 爆破修改的字节

`Offsets`:`16` `进制`

```
 破     原

C:    A0    EA
D:    55    50
E:    3C    48
F:    EC    C5
3B:    FF    01
```

--------------------------------------------------------------------------------

【经验总结】

1. 找准 `VM` 关键点最重要 `VM` 中亦可爆破
2. 注重 `VM` 外防爆破...

--------------------------------------------------------------------------------

【版权声明】: 本文首发于 `PEDIY`(`hIMcrACk`) `一蓑烟雨`(`i++`) 无版权 欢迎转载 欢迎大侠指点~

2009 年 03 月 01 日 2:40:54

本代码由 `xTiNt` 自动着色 [`kbadboy.yeah.net`](http://kbadboy.yeah.net)
